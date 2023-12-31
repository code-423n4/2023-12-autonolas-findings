## Summary
| Head | Details |
| :--- | :--- |
| Protocol Overview | Sources utilized to get context. |
| Methods | Codebase evaluation methodology. |
| Codebase | Codebase quality and recommendations. |
| Risks | Concerns associated with Autonolas' architecture, business-logic and centralization. |
| Time spent | The amount of time spent on the research. |
| Acknowledgments |  |
## Protocol Overview
### Quick briefing
Autonolas Protocol facilitates autonomous on-chain services through the use of smart contracts to orchestrating providers and receivers. These contracts are central to coordinating software contributions from developers within the blockchain ecosystem.

1. **Foundational Concepts**: Autonolas is developed with a focus on autonomy. It aims to provide solutions for blockchain-based autonomous services, addressing the current needs and functionalities required in this domain.
    
2. **Technical Architecture**: The protocol employs a Multi-Agent Systems Architecture to ensure secure and efficient consensus mechanisms. This architecture supports crypto-native agent services and is integral to the on-chain operations of the protocol. Key components include Agent Components, Canonical Agents, Agent Instances, and Services.
    
3. **Tokenomics**: The OLAS token plays a crucial role in the Autonolas ecosystem. It is used to balance economic factors and encourage participation. The protocol's economic model includes bonding and discount factors, vital for its long-term economic health.
    
4. **Use Cases**: Autonolas has applications in several areas, particularly in DAO operations and protocol infrastructure. It supports functions such as treasury management, yield optimization, and trading strategy execution, underlining its utility in decentralized operations.
    
5. **Governance**: The governance system of Autonolas is designed for transparency and community involvement. It features a governance module, specific contracts like veOLAS and GovernorOLAS, and mechanisms including timelock (with incentives for longer-term locks) and a community multisig. These elements collectively enable a decentralized governance process.
### Sources
The protocol team has provided extensive high quality documentation both at the audit repo and at the docs webpage. Alongisde some content found on Youtube, this is enough contextualization to begin the security research on the codebase. 
I have opted not to build diagrams as the team already provided high-quality overviews:
![Pasted image 20231231103146.png](https://raw.githubusercontent.com/code-423n4/2023-12-autonolas/main/governance/docs/On-chain_architecture_v5.png)

The Whitepaper is an 80-page document that covers the audit scope and much more about Autonolas.

These are the sources utilized for this security research outside of what is contained at the repository:

| Source | Link | Brief description |
| :--- | :--- | :--- |
| Protocol | [Autonolas Protocol - Autonolas Developer Documentation](https://docs.autonolas.network/protocol/) | Protocol overview |
| Whitepaper | [Whitepaper v1.0 (autonolas.network)](https://www.autonolas.network/documents/whitepaper/Whitepaper%20v1.0.pdf) | Whitepaper - very complete. Not read on its entirety as many of the concepts it explains are not in-scope. |
| thefett - Autonolas Deep Dive | [protocol deep dives w/ thefett - autonolas (youtube.com)](https://www.youtube.com/watch?v=oznC6JXb7EQ) | A youtube video of David Minarsch deep diving on Autonolas and its features |
| Autonolas and IPFS | [Decentralized Off-chain Backends: How Autonolas utilizes IPFS across its stack - David Minarsch (youtube.com)](https://www.youtube.com/watch?v=bzSjPTF-6G4) | A youtube video of David Minarsch explaining how IPFS is utilized - notice how this isn't directly in the scope of the audit but helps to get context in the features. |

## Methods
The security research was done on three steps: context-gathering, codebase-evaluation and report-generation.

**Context-gathering:** involved reading all documentation available and reviewing all sources provided at the protocol overview. The main intention during this phase is to develop a mental-model of flows and reach clarity on the protocol's concepts. This helps with the research as threat-modeling can be adjusted to the protocol and its target audience operational risks.

**Codebase-evaluation:** consisted of 3 steps. Automated diagramming, line-by-line review and semantical analysis. 
1. Automated diagramming: with the utilization of [Solidity Visual Developer by tintinweb]([Solidity Visual Developer - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor)) I have overviewed the contracts before diving into the code. This helps pointing the areas of most expected complexity and they are left to be checked at last. At the end of this stage, I have a list of the functions to analyze in order. Example diagram of the Depository.sol contract:
	![Diagram](https://i.postimg.cc/Y0S9B5PL/digraph-G.png)
	GuardCM.sol contract:
	![GuardCM](https://i.postimg.cc/rw6Vn6gw/GUardCM.png)
2. Line-by-line review: consisted of analyzing the functions at the order determined at the previous stage. This thorough approach from simple to complex functions enables the understanding of complexity behemoths, such as the [checkpoint function at the Tokenomics contract]([2023-12-autonolas/tokenomics/contracts/Tokenomics.sol at main · code-423n4/2023-12-autonolas (github.com)](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L880C5-L1075C1)).
3. Semantical analysis: consisted of first mirroring methods that did opposite things (deposit/withdraw, join/leave, sell/buy) to find asymmetries; and secondly a mid-level overview of state-variable-assignments and external calls to see if they made sense based on the documentation-specified features.

**Report-generation:** involved gathering the observations made on the previous phases and pre-sorting them into the following categories: "ask-the-sponsor", "easy-proof-of-concept", "hard-proof-of-concept", "false-positive".
After the pre-sort, questions are asked and the reports are written.
Finally the analysis report is written and the research is wrapped up.
## Codebase

##### Tokenomics
| Aspect | Strengths | Weaknesses | Grade |
| :--- | :--- | :--- | ---- |
| Documentation | Clear natspec comments that help to quickly understand methods. The tokenomics repo also contains a PDF that thoroughly describes the business-logic and the contracts from a high-level and from a technical standpoint |  | A |
| Code-clarity | Most of the variables utilized are self-descriptive. <br><br>Functions properly state their use-cases at their names.  The developers utilize a lot of comments to describe the steps taken. <br><br>The definition of a public global constants at the TokenomicsConstants abstract contract is a very interesting design choice as it enables all inheriting contract to utilize those without much developer overhead. | The excessive amount of acronyms utilized at Tokenomics:checkpoint increases the effort required to understand it.<br><br>The usage of tp.unitPoints[0] and tp.unitPoints[1] to differentiate components from agents at the Tokenomics contract makes the developer experience not very intuitive and should be a point of concern during further developments. | B |
| Complexity | The codebase manages to avoid complexity at almost its entirety.<br><br>The features are implemented by composition of small code snippets.<br><br>Inline-assembly is utilized in very small snippets - which represents a good balance of optimization and safety. | Even though the Tokenomics:checkpoint function has to take many parameters and states into account, it could use some refactoring to reduce the risks of missing state transitions. Consider checking out the extra comments.<br><br> | B |

**extra comments**
The checkpoint function could be broken down into several smaller functions that can help the developers see more clearly whether a state is being properly transitioned or not.

I've taken this function and commented how it could be refactored into separate smaller functions that could be called in the following sequence:
1. getImplementationAddress
2. checkDonationBlock
3. calculateEpochTime
4. calculateIncentives
5. calculateInflation
6. adjustEffectiveBond
7. updateNextEpochTokenomics
8. adjustMaxBond
9. calculateAndUpdateIDF
10. calculateOwnerTopUps
11. rebalanceTreasuryAndSettleEpoch

By separating the state transitions into 11 smaller blocks, the developers are less likely to commit errors.

##### Governance
| Aspect | Strengths | Weaknesses | Grade |
| :--- | :--- | :--- | ---- |
| Documentation | Similarly to the tokenomics repo, the Autonolas team has provided top-notch information to enable other developers get context. The governance repo contains documents that not only explain the contracts but also the bonding mechanism, the bridging mechanics and previous found vulnerabilities. That combined with well-written natspec and developer comments makes up for solid documentation. |  | A |
| Code-clarity | Variables and functions have descriptive naming.<br>Natspec comments express context with clarity.<br>Developer comments provide a step-by-step approach to functions.<br>Public variables in CAPSLOCK to avoid magic values.<br><br>The acronyms utilized at veOLAS:checkpoint are intuitive |  | A |
| Complexity | The contracts inherit battle-tested libraries from open-zeppelin and implement most* Gnosis' Guard interfaces correctly.<br><br>Most functions are straightforward and don't implement more features than their naming suggests.  | veOLAS:checkpoint is a very big method and it is easy to get lost. Could use some refactoring to help developers not lose track of state transitions.  | B |

* supportsInterface implementation was missing on GuardCM contract - makes Safe:setGuard revert.
##### Registries
| Aspect | Strengths | Weaknesses | Grade |
| :--- | :--- | :--- | ---- |
| Documentation | Similarly to the repos previously described, the documentation was amazing. |  | A |
| Code-clarity | Variables and functions are descriptive.<br><br>There's no magic number usage.<br><br>Some functions have certain complexity but are accompained by thorough developer comments.  |  | A |
| Complexity | The inheritance patterns are clearly defined and easy to disguinsh.<br>UnitRegistry inherits GenericRegistry. AgentRegistry and ComponentRegistry inherit UnitRegistry.<br>RegistriesManager inherits GenericManager.<br><br>The repo also has contracts that are Gnosis Safe-based multisigs and follow the defined interfaces. | The usage of inline assembly instead of abi decoding can lead to oversights. | A- |
The registries repo was truly pleasurable to look for bugs as the folder structuring, the design patterns and the logic was well set up.

I have opted not to include the lockbox analysis as it is not a domain (solang) I have good understanding of.
## Risks

### A-01 - Centralization
There are several functions that are owner-determined and could be used to induce end-user losses. This is an expected side-effect of managed-protocols.
The product's interactions are permissionless provided the protocol's core functionalities are set up.
#### Recommended Mitigation Steps
The protocol has plans to implement Community-Owned multisigs and the contracts deployer is a Timelock governed by the DAO. 

### A-02 The blacklist mechanics are easily circumvented

#### Description
The DonatorBlacklist contract inhibits addresses blacklisted by the protocol from donating ETH. This is easily circumvented by transferring the funds to a second address then donating. 

There is no clear definition on the parties that should be blacklisted, so it is difficult to suggest mitigation alternatives. 
Consider defining the blacklist persona further while keeping in mind some of the following elements:
OFAC sanctioned addresses
Mixer services-related addresses
Public-exploit addresses
Business-determined addresses
### A-03 - Inline Assembly Usage for Data Decoding  
#### Description
The usage of nline assembly bypasses many of Solidity's safety mechanisms. This can lead to issues in data handling, increased difficulty in debugging, and potential exploitation by malicious actors if any vulnerabilities are present. Moreover, the reduced readability and increased complexity make future maintenance and updates more challenging and error-prone.

The following example is taken from the HomeMediator contract:
```solidity
// solhint-disable-next-line no-inline-assembly
assembly {
    // First 20 bytes is the address (160 bits)
    i := add(i, 20)
    
    target := mload(add(data, i))
    // Offset the data by 12 bytes of value (96 bits)
    i := add(i, 12)
    value := mload(add(data, i))
    // Offset the data by 4 bytes of payload length (32 bits)
    i := add(i, 4)
    payloadLength := mload(add(data, i))
}
```
#### Recommended Mitigation Steps
Consider utilizing abi.decode for complex data-handling operations.
### A-04 - RegistriesManager:create can be front-run and enable contribution theft

#### Impact
The RegistriesManager create function enables any developer to create/register a new component or agent to an unit owner of the caller's choice. The issue with that is the mempool is public and malicious parties could opt to front-run the transaction and steal credit for the component/agent contribution by calling the same transaction but with it's own address as the unit owner.

#### Proof of Concept
As the dependencies data are stored off-chain, the ability of permissionlessly calling [create]([2023-12-autonolas/registries/contracts/RegistriesManager.sol at main · code-423n4/2023-12-autonolas (github.com)](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L27C5-L43C6)) with any parameters should suffice as proof of concept. 
```solidity
function create(
        IRegistry.UnitType unitType,
        address unitOwner,
        bytes32 unitHash,
        uint32[] memory dependencies
    ) external returns (uint256 unitId)

    {

        // Check if the creation is paused
        if (paused) {
            revert Paused();
        }

        if (unitType == IRegistry.UnitType.Component) {
            unitId = IRegistry(componentRegistry).create(unitOwner, unitHash, dependencies);
        } else {
            unitId = IRegistry(agentRegistry).create(unitOwner, unitHash, dependencies);
        }
    }
```
#### Recommended Mitigation Steps
The issue is contribution theft and cannot be mitigated by in-contract mechanics. 
Utilize a relay such as flashbots for component/agent creation.
Extensively warn developers to properly place licensing and credits at their dependencies and components before pushing a transaction to create the component/agent.
Consider the possibility of disputes arising in the future.

### A-05 - GuardCM's behaviour under the paused state induces mistakes

#### Impact
The checkTransaction function at the GuardCM contract checks if it has authorized arguments and is important to protect the Gnosis Safe community multisig.
The function works by reverting in case any unwanted parameter is passed.
The issue, however, lies at the first if condition (that guards the whole checks ran at the function):
```solidity
if (paused == 1) {
```
This means if the contract GuardCM contract is paused (defined by paused == 2), checkTransaction does not check anything and will allow the conditions that were expected to cause reversion not to revert.

This is confusing and can induce mistakes. The "pause" keyword may lead one to believe the whole multisig is paused and the Guard would revert every single call.

#### Proof of Concept
Place the following code snippet at the GuardCM.js test, after the "Pause the guard" test:
```javascript
it("Pause the guard and it doesn't revert if to argument is the multisig's address", async function () {
            // Try to pause the guard not by the timelock or the multisig
            await expect(
                guard.pause()
            ).to.be.revertedWithCustomError(guard, "ManagerOnly");

            // Pause the guard by the timelock
            const payload = guard.interface.encodeFunctionData("pause", []);
            await timelock.execute(guard.address, payload);
            expect(await guard.paused()).to.equal(2);
            
            // put random txData to checkTransaction
            // put the multisig address as the to argument.
            let txData = await timelock.interface.encodeFunctionData("schedule", [l1BridgeMediators[0], 0, gnosisPayload,
                Bytes32Zero, Bytes32Zero, 0]);
            await guard.checkTransaction(multisig.address, 0, txData, 0, 0, 0, 0, AddressZero, AddressZero, "0x", AddressZero);
            
        });
```

Run the test with:
```shell
npx hardhat test test/GuardCM.js --grep "Pause the guard and it doesn't revert if to argument is the multisig's address"
```

Notice how the GuardCM contract doesn't revert.
#### Recommended Mitigation Steps
Make sure to revert if the paused variable is equal 2 during checkTransaction calls.
If the intention is to pause the Safe's calls from being guarded, consider setting it's guard to the zero address during pause periods then setting it back to Guard implementations when unpausing.

## Time spent
55 hours

## Acknowledgements
Thanks for taking the time to read this report.
Thanks to the Autonolas team for the opportunity and specifically to Kupermind for patiently answering all my questions.

### Time spent:
55 hours