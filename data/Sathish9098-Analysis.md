# Olas(Autonolas) Analysis

Olas functions as an integrated platform designed to facilitate a range of off-chain services, encompassing automation processes, oracle systems, and collaboratively owned artificial intelligence capabilities. It provides a comprehensive suite for the development and deployment of these services, underpinned by a protocol that promotes the incentivization of both the creation and ongoing decentralized operation of these services. This infrastructure is engineered to support a co-owned, decentralized model, emphasizing collaborative development and management within its ecosystem.

# Protocol Risk model

## Systemic risks

## Governance

### Cross-Chain Communication Security risk

#### Bridge Contract Dependability (FxGovernorTunnel and HomeMediator):

- These contracts are responsible for relaying governance decisions between the Ethereum mainnet and other networks like Polygon and Gnosis Chain.
- They process encoded data sent across the bridge, executing transactions based on this data.
- Any vulnerability in these contracts (e.g., incorrect message decoding, improper validation of message sender) could lead to unauthorized or incorrect execution of governance actions.

#### Underlying Bridge Technology Risks (FxPortal, AMB)

- ``FxPortal``: Used for communication between Ethereum and Polygon. Relies on the security and efficiency of Polygon's PoS bridge.
- ``Arbitrary Message Bridge (AMB)`` : Facilitates communication between Ethereum and Gnosis Chain.
- These bridges operate by relaying messages and require stringent security to prevent interception or alteration of messages. A breach could result in lost, delayed, or manipulated governance instructions.

#### Bridge Reliability and Downtime
- Any downtime or performance issues in the bridge technologies can delay the transmission of governance decisions, impacting the timeliness of their implementation.
- Prolonged bridge downtimes could render cross-chain governance inoperative, forcing reliance on single-chain governance mechanisms, which may not be feasible for certain decisions.

#### Cross-Chain Reorg Risks

Blockchain reorganizations (reorgs) can affect the finality of cross-chain messages. A reorg on the source chain after a message has been sent but before it's finalized on the destination chain could result in discrepancies or double executions.

### Governance Proposal and Execution Delays risk

#### Implications of Timelock Delays:

- ``Delayed Response in Dynamic Situations``: In a rapidly changing environment, such as in response to security threats, market movements, or urgent protocol upgrades, the delay can hinder the protocol's ability to implement timely decisions.
- ``Reduced Flexibility``: The set delay period (which might be several days or even longer) means that once a proposal is approved, the protocol must wait for this period to elapse before any action can be taken, even if the situation demands an immediate response.
- ``Potential for Market Manipulation``: In cases where governance decisions are market-sensitive (e.g., tokenomic changes), the known delay period can be exploited by market participants for profit, potentially leading to price manipulation or front-running.
- ``Security vs. Agility Trade-off`` : The protocol must balance the need for security (preventing hasty or malicious actions) with the need for agility (responding swiftly to critical issues).

### Risk of Low Voter Participation

#### Consequences of Low Voter Turnout:

- ``Unrepresentative Decisions``: With low participation, the decisions made could disproportionately reflect the will of a small group of voters, which might not align with the broader community's interest.
- ``Easier Manipulation``: Low turnout reduces the number of votes required to pass proposals, making it easier for a minority or even malicious actors to influence governance decisions.
- ``Reduced Protocol Legitimacy``: Consistently low participation can undermine the legitimacy of the governance process, as decisions are made by a small segment of the holder base.

#### Technical and Sociological Factors:

- ``Complexity and Engagement``: The technical complexity of proposals or a disconnect between stakeholders' understanding and the matters at hand can lead to lower engagement.
- ``Token Distribution``: If veOLAS tokens are concentrated in the hands of a few holders, it can demotivate broader community participation, leading to apathy among smaller holders.
- ``Incentive Structures``: Lack of adequate incentives for voting can result in voter apathy.

## Registries

### Dependency and Interoperability Risks

- ``Risk``: The registries depend on each other (e.g., AgentRegistry depending on ComponentRegistry). Issues in one registry could cascade to others, especially considering the dependency checks in contracts like AgentRegistry.
- ``Impact``: A failure or vulnerability in one registry could compromise the integrity of another, affecting the overall reliability of the system.

### Data Integrity and Validation Risks:

- ``Risk``: The registries handle critical data, such as IPFS hashes and dependencies. Inadequate validation of this data could lead to incorrect or malicious data being registered.
- ``Impact``: Incorrect data could lead to faulty operations or could be used to exploit other parts of the system, such as the deployment of compromised agents or components.

## Tokenomics

### Liquidity Pool Dominance

- ``Risk``: Large liquidity providers might become overly influential, controlling a significant portion of the OLAS tokens.
- ``Impact``: This could lead to market manipulation or centralization of token ownership, affecting the token's price and stability.

### Economic Sustainability

- ``Risk``: The long-term viability of the tokenomics model is crucial. Unsustainable incentive models could lead to resource drainage.
- ``Impact``: If the incentives become unsustainable, it could lead to a decrease in developer participation and liquidity provision, harming the protocol's growth.

### Token Concentration and Governance

- ``Risk``: Token concentration in the hands of a few might lead to governance issues, where major holders wield disproportionate influence.
- ``Impact`` : This could skew decision-making processes and lead to decisions that favor a minority of stakeholders.

### Gaming the Incentive System:

- ``Risk``: If the incentive mechanisms are not well-designed, there's a risk of developers gaming the system to earn rewards without contributing meaningfully.
- ``Impact``: Such exploitation could drain resources and detract from the protocol's goal of fostering genuinely useful code.


## Technical risks

## Governance

### Smart Contract Bugs and Flaws
Even with rigorous testing and audits, there's always a risk of undiscovered bugs in smart contracts, which could lead to malfunctions or exploits. This includes risks in both the governance contracts (like GovernorOLAS) and associated contracts (like FxGovernorTunnel, HomeMediator, BridgedERC20).

### Upgradeability and Contract Interoperability
If any part of the governance system is upgradeable, there's a risk associated with the upgrade process itself, including potential mismatches or incompatibilities between different contract versions.

### Denial of Service (DoS) Attacks
Certain governance mechanisms could be vulnerable to DoS attacks, where an attacker deliberately floods the system with transactions or proposals to halt or slow down the governance process.

### Time Manipulation Risk
Functions dependent on block timestamps might be vulnerable to minor manipulations by miners, affecting processes that rely on specific timing conditions.

### Transaction Ordering Manipulation 
Malicious actors could attempt to exploit the public nature of pending transactions in the mempool to influence governance decisions, such as by front-running vote-related transactions.

## Registries

### Integration with External Systems (like IPFS)

- Dependence on external systems for critical functionality (e.g., storing metadata on IPFS) introduces risks related to the availability and integrity of these systems.
- Downtime or data loss in external systems could adversely affect the registries' operation.

## Tokenomics

### Scalability and Performance Issues

- High transaction volumes, especially in liquidity pools and staking mechanisms, might strain the blockchain network, leading to high gas fees and slow transaction times.
- Scalability issues could hinder user participation and affect the overall 

### Liquidity Pool Risks:
- Liquidity pools, particularly those involving bonding mechanisms, are exposed to risks such as impermanent loss or pool imbalances.
- These risks could deter liquidity providers or lead to losses, impacting the protocol’s liquidity.

### Regulatory Compliance Risks:
- Evolving regulatory standards in different jurisdictions can impact tokenomics, especially regarding DeFi practices and cross-chain transactions.
- Non-compliance could lead to legal challenges or necessitate significant changes in the tokenomics model.

## Integration risks
Integration risks in the context of the Autonolas protocol, particularly considering its multi-chain focus and interaction with various blockchain networks and external systems, involve a range of technical challenges.

## Governance

### Smart Contract Compatibility Risks

- ``Risk Details``: When integrating with contracts on different blockchains or systems (like Ethereum, Polygon, and Gnosis Chain), there's a risk of compatibility issues due to differing contract standards, gas models, or blockchain functionalities.
- ``Technical Specifics``: Differences in EVM compatibility, state size limits, or gas pricing mechanisms across chains can affect the behavior of integrated contracts.

### Cross-Chain Communication Failures:

- ``Risk Details``: Reliance on cross-chain bridges (like FxPortal for Polygon and AMB for Gnosis Chain) for governance actions introduces risks of communication failures or discrepancies in data transmission.
- ``Technical Specifics``: Bridge mechanisms may face downtime, data throughput limitations, or inconsistencies in data validation and verification processes.

### Network Congestion and Transaction Delays:

- ``Risk Details``: High network congestion on integrated blockchains can lead to delayed transaction confirmations, affecting time-sensitive operations.
- ``Technical Specifics``: Network overload, especially on chains with limited throughput, can result in increased transaction fees and delayed block confirmations.

### User Interface and Interaction Issues:

- ``Risk Details``: Integrating various blockchain functionalities into a user-friendly interface poses challenges, where mismatches or errors can lead to user mistakes or misinterpretations.
- ``Technical Specifics`` : Complex interactions required for cross-chain functionalities might not be intuitively presented in user interfaces, increasing the risk of errors.

## Registries

### Inter-Registry Dependencies

- The AgentRegistry and ComponentRegistry rely on each other for validating dependencies. If one registry fails or has incorrect data, it could impact the integrity of the other.
- Malfunctions or vulnerabilities in one registry could cascade, affecting the functionality of dependent registries.

### Consistency Across Registries:

- Ensuring consistent data across multiple registries is challenging. Discrepancies can arise due to synchronization issues or during updates.
- Inconsistencies can lead to operational failures or incorrect behavior in contracts relying on registry data.

## Tokenomics

### External Data and Oracle Integration Risks
- ``Risk Description``: Dependence on external oracles for market data or other inputs could introduce inaccuracies.
- ``Potential Impacts``:
Malfunctioning or manipulated oracles could skew incentive mechanisms, affecting staking rewards, liquidity incentives, and other crucial operations.

## Admin abuse risks

```
function changeGovernor(address newGovernor) external {
function changeGovernorCheckProposalId(uint256 proposalId) external {
function changeOwner(address newOwner) external {
function mint(address account, uint256 amount) external {
function burn(uint256 amount) external {
function pause() external virtual {
function unpause() external virtual {
function changeManager(address newManager) external virtual {
function setBaseURI(string memory bURI) external virtual {
function changeManagers(address _tokenomics, address _treasury) external {
function changeBondCalculator(address _bondCalculator) external {
function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {
function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {
function changeTokenomicsImplementation(address implementation) external {
function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {
function changeDonatorBlacklist(address _donatorBlacklist) external {
 
```
### Owner Access (Functions like changeOwner, changeGovernor, mint, burn, etc.)

#### Impact 

- ``Governance Manipulation``: Unauthorized changes to governance roles or rules could lead to detrimental protocol changes or malicious governance actions.
- ``Token Supply Disruption``: The ability to mint and burn tokens could be exploited to manipulate the token's supply, potentially destabilizing its value and the protocol's economy.
- ``Unauthorized Contract Upgrades``: Changing contract implementations or altering critical settings could introduce vulnerabilities or alter the protocol’s intended behavior.
- ``Operational Disruption``: Functions like pausing contracts can halt critical operations, directly impacting users and the protocol's functionality.

```solidity
function reserveAmountForBondProgram(uint256 amount) external returns (bool success) {
function refundFromBondProgram(uint256 amount) external {

```
### Depository Access (Functions like reserveAmountForBondProgram, refundFromBondProgram)

#### Impact 
- ``Financial Losses``: Unauthorized access to reserve or refund functions could lead to misappropriation of funds, affecting the protocol's financial stability.
- ``Bond Program Manipulation``: The integrity of bond programs could be compromised, affecting investor confidence and the protocol's reputation.

```solidity
function trackServiceDonations(
        address donator,
        uint256[] memory serviceIds,
        uint256[] memory amounts,
        uint256 donationETH
    ) external {
    
```
### Treasury Access (Function like trackServiceDonations)

#### Impact 
- ``Misallocation of Funds``: If the tracking or allocation of donations is manipulated, it could lead to improper use of funds, affecting the protocol's financial integrity.
- ``Data Integrity Issues``: Inaccurate tracking of donations could disrupt the protocol’s financial reporting and decision-making processes.

```solidity
function accountOwnerIncentives(address account, uint256[] memory unitTypes, uint256[] memory unitIds) external
```
#### Impact 
- ``Incentive Manipulation``: Unauthorized changes to incentive distributions could unfairly advantage certain users, disrupting the protocol's reward mechanisms and user trust.
- ``Resource Drainage``: Malicious distribution of incentives could drain resources, impacting the protocol's sustainability.

## Architecture assessment of business logic 

### Governance Architecture

The governance architecture involves various contracts and mechanisms working together to ensure decentralized decision-making and management. Based on the information provided earlier about contracts like GovernorOLAS, FxGovernorTunnel, HomeMediator, and others, let's construct an outline of the likely governance architecture.

![GovernanceArchi](https://gist.github.com/assets/58845085/16edddc6-027f-4aeb-bd14-c2ed9faa1efd)

### Registries Architecture

Architecture assessment of the business logic based on the architecture of the Autonolas protocol's registries system (comprising UnitRegistry, ComponentRegistry, AgentRegistry, and RegistriesManager), we'll consider key architectural aspects and their alignment with business objectives

![registrues](https://gist.github.com/assets/58845085/1c7f5378-aaaa-4d32-beda-d9876517cdd1)


### GuardCM Contract

Crucial component designed to oversee and possibly regulate actions taken by a community-owned multisig wallet (CM). This contract likely functions as a safeguard within the governance framework, ensuring that actions taken by the CM align with the broader intentions and rules of the decentralized autonomous organization (DAO).

### Mind Map
![GuardCM Mindmap](https://gist.github.com/assets/58845085/e46b66bd-1386-4a81-940f-1ff4233015fa)

### Intractions Flow

![GuardCM](https://gist.github.com/assets/58845085/ab3ed165-fea2-421c-8d7b-98b86da840fc)


## Software Engineering Considerations

### Security Best Practices

- Follow established security best practices, such as checks-effects-interactions pattern, to prevent reentrancy attacks, and use guard checks against overflows/underflows.
- Regular security audits and incorporating static analysis tools in the development pipeline are essential.

### Testing and Quality Assurance

- Comprehensive testing, including unit tests, integration tests, and stress tests, should be conducted. This ensures that the contracts behave as expected under various scenarios.
- Consideration should be given to test-driven development (TDD) practices to ensure robustness from the outset.

### Modularity and Reusability

The contracts demonstrate modularity (e.g., inheriting from OpenZeppelin contracts). This approach should be maintained or enhanced to ensure code reusability and easier updates.

### Scalability Considerations:
Design contracts with scalability in mind. This includes considering layer 2 solutions or sidechains for more efficient processing, especially for cross-chain functionalities.

### Community Feedback and Governance:
Incorporate mechanisms for community feedback and governance decisions into the software development lifecycle, reflecting the decentralized ethos of the protocol.

### Security First Approach:
Given the critical role of registries in managing on-chain representations of components and agents, a security-first approach is paramount. This includes following best practices like check-effects-interactions pattern, secure handling of external calls, and gas optimizations.

### Disaster Recovery and Emergency Protocols:
Implement mechanisms for data recovery and emergency handling in case of critical failures. This could include pausing functionality in the event of a detected anomaly or breach.


## Testing suite Analysis

High levels of coverage across all metrics, indicating thorough testing.

### Governance

-------------------------|----------|----------|----------|----------|----------------|
File                     |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-------------------------|----------|----------|----------|----------|----------------|
 contracts\              |      100 |    93.84 |      100 |    98.81 |                |
  GovernorOLAS.sol       |      100 |      100 |      100 |      100 |                |
  OLAS.sol               |      100 |      100 |      100 |      100 |                |
  Timelock.sol           |      100 |      100 |      100 |      100 |                |
  veOLAS.sol             |      100 |    92.37 |      100 |     98.4 |249,253,283,286 |
  wveOLAS.sol            |      100 |      100 |      100 |      100 |                |
 contracts\bridges\      |       96 |      100 |       95 |    97.62 |                |
  BridgedERC20.sol       |      100 |      100 |      100 |      100 |                |
  FxERC20ChildTunnel.sol |      100 |      100 |      100 |      100 |                |
  FxERC20RootTunnel.sol  |    78.57 |      100 |       80 |       85 |       74,77,79 |
  FxGovernorTunnel.sol   |      100 |      100 |      100 |      100 |                |
  HomeMediator.sol       |      100 |      100 |      100 |      100 |                |
 contracts\bridges\test\ |      100 |      100 |      100 |      100 |                |
  FxRootMock.sol         |      100 |      100 |      100 |      100 |                |
 contracts\interfaces\   |      100 |      100 |      100 |      100 |                |
  IERC20.sol             |      100 |      100 |      100 |      100 |                |
  IErrors.sol            |      100 |      100 |      100 |      100 |                |
 contracts\multisigs\    |      100 |      100 |      100 |      100 |                |
  GuardCM.sol            |      100 |      100 |      100 |      100 |                |
 contracts\test\         |       75 |      100 |       80 |       75 |                |
  BrokenERC20.sol        |       75 |      100 |       80 |       75 |             31 |
-------------------------|----------|----------|----------|----------|----------------|
All files                |    98.79 |    97.19 |    98.36 |     98.7 |                |
-------------------------|----------|----------|----------|----------|----------------|

#### Notable Insights
``veOLAS.sol``: Although most metrics are at 100%, there's a minor gap in branch coverage (92.37%) and line coverage (98.4%). Uncovered lines include 249, 253, 283, 286, suggesting potential edge cases or conditional branches not tested.

#### Recommendations
``veOLAS.sol``: Increase test cases to cover the branches and lines mentioned above, focusing on the conditional logic that might not be fully exercised in the current test suite.

### Registries 

------------------------------------|----------|----------|----------|----------|----------------|
File                                |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------------------------------|----------|----------|----------|----------|----------------|
 contracts\                         |    90.54 |    78.13 |    86.21 |    87.34 |                |
  AgentRegistry.sol                 |      100 |     87.5 |       75 |    86.67 |          62,72 |
  ComponentRegistry.sol             |      100 |      100 |      100 |      100 |                |
  GenericManager.sol                |    57.14 |       50 |    66.67 |    57.14 |... 27,28,31,32 |
  GenericRegistry.sol               |    69.23 |    35.71 |    71.43 |       60 |... 6,98,99,100 |
  RegistriesManager.sol             |      100 |      100 |      100 |      100 |                |
  UnitRegistry.sol                  |      100 |      100 |      100 |      100 |                |
 contracts\interfaces\              |      100 |      100 |      100 |      100 |                |
  IErrorsRegistries.sol             |      100 |      100 |      100 |      100 |                |
  IRegistry.sol                     |      100 |      100 |      100 |      100 |                |
 contracts\multisigs\               |    80.77 |    81.82 |      100 |    83.72 |                |
  GnosisSafeMultisig.sol            |      100 |    83.33 |      100 |      100 |                |
  GnosisSafeSameAddressMultisig.sol |    72.22 |    81.25 |      100 |       75 |... 118,119,120 |
------------------------------------|----------|----------|----------|----------|----------------|
All files                           |       88 |    79.07 |    88.24 |    86.57 |                |
------------------------------------|----------|----------|----------|----------|----------------|

#### Notable Insights
``GenericManager.sol`` and GenericRegistry.sol show significantly lower coverage across all metrics. Specifically, GenericRegistry.sol has only 69.23% statement coverage and 35.71% branch coverage.
Uncovered lines in ``GenericRegistry.sol`` include 96, 98, 99, 100, and in ``GenericManager.sol``, they include 25, 26, 27, 28, 31, 32.

#### Recommendations
``GenericManager.sol`` & ``GenericRegistry.sol``: Develop additional test cases to improve coverage, particularly targeting the conditional logic and functions that are currently under-tested.
``AgentRegistry.sol``: Address uncovered lines (62, 72) to enhance branch coverage.

### Tokenomics

-----------------------------|----------|----------|----------|----------|----------------|
File                         |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-----------------------------|----------|----------|----------|----------|----------------|
 contracts\                  |      100 |    95.22 |      100 |    98.56 |                |
  Depository.sol             |      100 |       80 |      100 |    93.53 |... 312,385,440 |
  Dispenser.sol              |      100 |      100 |      100 |      100 |                |
  DonatorBlacklist.sol       |      100 |      100 |      100 |      100 |                |
  GenericBondCalculator.sol  |      100 |    71.43 |      100 |       90 |          32,60 |
  Tokenomics.sol             |      100 |      100 |      100 |      100 |                |
  TokenomicsConstants.sol    |      100 |      100 |      100 |      100 |                |
  TokenomicsProxy.sol        |      100 |      100 |      100 |      100 |                |
  Treasury.sol               |      100 |      100 |      100 |      100 |                |
 contracts\interfaces\       |      100 |      100 |      100 |      100 |                |
  IDonatorBlacklist.sol      |      100 |      100 |      100 |      100 |                |
  IErrorsTokenomics.sol      |      100 |      100 |      100 |      100 |                |
  IGenericBondCalculator.sol |      100 |      100 |      100 |      100 |                |
  IOLAS.sol                  |      100 |      100 |      100 |      100 |                |
  IServiceRegistry.sol       |      100 |      100 |      100 |      100 |                |
  IToken.sol                 |      100 |      100 |      100 |      100 |                |
  ITokenomics.sol            |      100 |      100 |      100 |      100 |                |
  ITreasury.sol              |      100 |      100 |      100 |      100 |                |
  IUniswapV2Pair.sol         |      100 |      100 |      100 |      100 |                |
  IVotingEscrow.sol          |      100 |      100 |      100 |      100 |                |
-----------------------------|----------|----------|----------|----------|----------------|
All files                    |      100 |    95.22 |      100 |    98.56 |                |
-----------------------------|----------|----------|----------|----------|----------------|

#### Notable Insights
``Depository.sol`` shows lower branch coverage (80%) compared to other metrics. Uncovered lines are 312, 385, 440.
``GenericBondCalculator.sol`` has a lower branch coverage of 71.43% and line coverage of 90%, with uncovered lines 32 and 60.

#### Recommendations
``Depository.sol`` & ``GenericBondCalculator.sol`` : Enhance test cases focusing on branches and lines not currently covered. Pay particular attention to complex conditional statements and scenarios that may not be adequately represented in the current test suite.

### General Recommendations for Testiong 

- ``Consistent Test Standards``: Ensure uniformity in test coverage standards across all contracts to maintain code quality and reliability.
- ``Branch and Edge Case Testing``: Prioritize testing for branches and edge cases, especially in contracts with lower branch coverage.
- ``Continuous Integration of Tests``: Implement continuous integration processes to automatically run tests and report coverage, aiding in identifying coverage gaps promptly.

## Code Weak Points and Single point of failures

### Centralized Control in BridgedERC20

- ``Affected Code``: mint and burn functions.
- ``Weakness``: Both functions are only callable by the owner. This centralization poses a risk, as the security of the entire token supply hinges on a single account.

```solidity

function mint(address account, uint256 amount) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    _mint(account, amount);
}

function burn(uint256 amount) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    _burn(msg.sender, amount);
}

```

### Data Validation in FxGovernorTunnel and HomeMediator

- ``Affected Code``: processMessageFromRoot and processMessageFromForeign.
- ``Weakness``: The processing functions unpack and execute transaction data from cross-chain messages. Inadequate validation of this data can lead to execution of harmful transactions.
Code Snippet (FxGovernorTunnel):

```solidity

function processMessageFromRoot(uint256 stateId, address rootMessageSender, bytes memory data) external override {
    // ... validation checks ...

    for (uint256 i = 0; i < dataLength;) {
        // ... data unpacking and transaction execution ...
    }
}

```

### Governance Actions Timelock in GovernorOLAS

- ``Affected Code``: propose and _execute functions.
- ``Weakness``: The timelock delays the execution of governance decisions, which can be a limitation in urgent situations.

```solidity

function _execute(
    uint256 proposalId,
    address[] memory targets,
    uint256[] memory values,
    bytes[] memory calldatas,
    bytes32 descriptionHash
) internal override(Governor, GovernorTimelockControl)
{
    super._execute(proposalId, targets, values, calldatas, descriptionHash);
}

```

### Potential for Front-Running

- ``Affected Code``: Public functions involved in governance decisions and token operations in all contracts.
- ``Weakness``: The visibility of transactions in the mempool before being confirmed can lead to front-running, especially in governance and token-related transactions.










### Time spent:
15 hours