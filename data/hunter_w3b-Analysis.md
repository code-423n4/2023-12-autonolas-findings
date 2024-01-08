# Analysis - Olas Contest

![Olas-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FQuJYSHjijiw.0&w=256&q=75)

## Description overview of The Olas Contest

Olas is a platform that allows for the creation of off-chain services to support decentralized applications . These off-chain services handle things like data feeds, APIs, computation, automation, etc. that don't need to be included on-chain.Some examples of off-chain services could be: price oracles that fetch token prices from exchanges, natural language AI assistants, weather data APIs, computer vision APIs, web scraped data feeds, etc.

**In summary, Olas is**:

- A unified network for off-chain services like automation, oracles, co-owned AI.

- It offers tools/infrastructure to build off-chain services and incentivize their creation/operation in a decentralized way.

- The key parts are governance, registries, and tokenomics:

  - Governance allows the community to propose/vote on changes using the veOLAS governance token. This includes cross-chain governance.

  - Registries allow developers to register/manage their off-chain code (represented on-chain as NFTs).

  - Tokenomics incentivizes creation of useful code and capital growth via bonding mechanism.

So in essence, Olas aims to provide the plumbing/incentives for a decentralized network of off-chain services powered by its native OLAS token and governed by its community. The core value proposition is enabling scalable dapps without bloating blockchains by leveraging these off-chain "helper" services.

**The key contracts of Olas protocol for this Audit are**:

- **GovernorOLAS.sol**: This is the main governance contract that allows anyone meeting a threshold to propose governance actions for veOLAS holders to vote on. It implements an OpenZeppelin-based governance system with features like proposal creation, voting, execution and compatibility with previous versions. This contract is important to audit how governance decisions are made.

- **OLAS.sol**: This ERC-20 token contract includes additional `owner/minter` management features and inflation-controlled minting. Auditing this contract ensures the tokenomics are implemented as intended and `ownership/minting` is restricted properly.

- **veOLAS.sol**: This contract manages the virtual voting tokens representing voting power based on `amount/time` of OLAS locked. Auditing this contract is important to validate the voting power calculation and that votes can only be cast by `veOLAS` holders.

- **Tokenomics.sol**: This contract manages OLAS token inflation and reward mechanisms. A thorough audit of this contract is needed to ensure rewards are distributed correctly over epochs as intended by the protocol.

- **Treasury.sol**: This contract manages the OLAS treasury by handling funds, bond `deposits/withdrawals` and balances. Auditing this contract is critical to validate treasury funds are stored and managed securely per the tokenomics design.

Auditing the key contracts of the Olas Protocol is essential to ensure the security of the protocol and they form the backbone of the Olas protocol, outlining its governance structure, lending terms, token management. Focusing on them first will provide a solid foundation for understanding the protocol's operation and how it manages user assets and stability. Here's why it's important to audit these specific contracts:

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

**Governance contracts**

- **GovernorOLAS.sol**: This is the `main governance` contract. It allows anyone meeting a `threshold` to propose governance actions, and for `veOLAS` holders to vote on proposals.The contract is an implementation of a governance system using the OpenZeppelin framework, integrating features such as proposal creation, voting, execution, timelock control, and compatibility with previous versions, designed for the Autonolas protocol.

- **Timelock.sol**: The contract is a `time-controlled` execution system, utilizing the OpenZeppelin TimelockController, for managing delays and access control over `proposed` and `executed` transactions.

- **OLAS.sol**: This contract is an ERC-20 token with additional features such as `owner` and `minter` management, inflation-controlled minting, and allowance manipulation.

- **veOLAS.sol**: This manages the virtual veOLAS tokens that represent voting power. Holders lock OLAS tokens to receive veOLAS with voting weight based on amount/time locked.The veOLAS contract is an implementation of a `time-weighted` voting system using a modified Curve Finance Vyper implementation, allowing users to lock `OLAS` tokens for a specified period with time-dependent voting power and checkpointed state.

- **wveOLAS.sol**: The contract Serving as a wrapper for view functions of the veOLAS contract, implementing a token with features related to voting power, supply points, slope changes, user points, and other functionalities, while restricting certain operations such as `transfers` and `delegation` to ensure non-transferability and non-delegatability of the underlying veOLAS token.

- **GuardCM.sol**: This contract is designed for a Gnosis Safe community multisig (CM). It serves as a `security guard` to validate transactions executed by a multisig wallet, checking for authorized combinations of target addresses, function selectors, and chain IDs. Additionally, it includes functionality for pausing and unpausing the guard, changing the governor address, and managing authorized target-selector-chainID combinations. The contract also handles bridged transactions across different chains and supports multiple operation selectors like schedule, scheduleBatch, requireToPassMessage, processMessageFromForeign, and sendMessageToChild.

- **FxGovernorTunnel.sol**: The contract is implementing the IFxMessageProcessor interface, serving as a child tunnel `bridge` for processing messages from the `root network`, featuring functions for receiving native network tokens, changing the Root Governor address, and processing messages with specified validations.

- **HomeMediator.sol**: The contract is serving as a mediator for a bridge between the foreign (L1) and home (L2) networks, implementing functions for receiving native network tokens, changing the Foreign Governor address, and processing messages with specified validations, particularly interacting with the AMB Contract Proxy (Home) on the home network.

- **FxERC20ChildTunnel.sol**: Manage bridged ERC-20 tokens between Layer2 and Layer1 Ethereum networks, facilitating token deposits, withdrawals.

- **FxERC20RootTunnel.sol**: Manage bridged ERC-20 tokens between Layer1 and Layer2 Ethereum networks, facilitating token withdrawals, deposits.

**Registries contracts**

- **GenericRegistry.sol**: This contract is utilizing ERC-721 standard for managing unique units with ownership and manager roles, supporting operations such as `changing ownership`, `setting a base URI`, and providing functions for unit existence and token URI generation.

- **UnitRegistry.sol**: Designed for registering and managing generalized units or entities with distinct Unit Types (e.g., Component or Agent), supporting creation, updates, and queries of these units with associated dependencies and IPFS CID hashes.

- **ComponentRegistry.sol**: Designed for registering and managing components with specific functionalities, including checking dependencies and obtaining subcomponents based on a linearized map.

- **AgentRegistry.sol**: Designed for registering and managing agents with specific functionalities, including checking dependencies against a `Component Registry` and obtaining `subcomponents` based on the unit type.

- **GenericManager.sol**: The contract is representing a generic manager template with functionalities for changing ownership, pausing, and unpausing the contract, designed to be extended for specific `registry management purposes`.

- **RegistriesManager.sol**: This contract is serving as a `periphery manager` for creating and updating components and agents, utilizing two distinct registry contracts for components and agents while inheriting generic management functionalities.

- **GnosisSafeMultisig.sol**: The contract is a Solidity smart contract that facilitates the creation of Gnosis Safe multisignature wallets, allowing users to deploy a Gnosis Safe with customized parameters such as owners, threshold, and optional payment details, utilizing a Gnosis Safe proxy factory.

**Tokenomics contracts**

- **Depository.sol**: This contract is implementing an `OLAS` Bond Depository, allowing users to `create` and `redeem` bonds with specified vesting periods and OLAS token rewards, while `managing` various bond products with unique characteristics and limitations.

- **Dispenser.sol**: Designed for `distributing` incentives to the owner of `components`/agents, with functions to claim rewards and top-ups based on specified unit types and IDs, utilizing a `tokenomics` contract and `treasury` contract for incentive calculations and fund transfers.

- **DonatorBlacklist.sol**: The contract allows the `owner` to `manage` blacklisting statuses for donator addresses, with functions to change the owner, set `blacklisting` statuses for multiple accounts, and check the blacklisting status of a specific account.

- **GenericBondCalculator.sol**: GenericBondCalculator facilitates the calculation of `OLAS token` payouts based on a bonding mechanism utilizing `UniswapV2` LP tokens, with safeguards against overflow and zero addresses, and includes functions to determine the current `OLAS/totalSupply` ratio for LP tokens.

- **Tokenomics.sol**: This contract is a tokenomics logic designed to manage the `inflation` and `reward mechanisms` for OLAS tokens, considering both ETH and OLAS tokens. The contract defines structures for `unit points`, `epoch points`, and `tokenomics points`, each containing various parameters related to rewards, top-ups, and tokenomics calculations. It sets limits on various variables, such as total supply, time, and coefficients, and specifies the maximum duration. The contract also includes functions for initializing and updating tokenomics parameters, changing the owner, managing various contract addresses, and adjusting incentive fractions, reserving and refunding amounts for bond programs, finalizing epoch incentives for specific components or agent IDs, tracking service donations, and performing epoch-based checkpoints to update tokenomics parameters and distribute rewards.

- **Treasury.sol**: The contract manages the OLAS Treasury. It handles various functionalities, including receiving ETH and OLAS tokens, depositing LP tokens for OLAS, withdrawing funds to specified addresses, updating treasury balances, and managing token-related operations such as enabling/disabling tokens for bonding with OLAS. The contract also includes mechanisms for handling service donations in ETH, rebalancing treasury funds for a specific epoch, and draining slashed funds from the service registry.

### Roles

Roles in the provided smart contract ("Treasury") are generally associated with different addresses or entities that have specific permissions or responsibilities. Here are the roles and their associated functionalities in the contract:

1. **Owner:**

   - Address: `owner`
   - Role: The owner has special privileges and can perform critical actions such as changing the owner address, updating managing contract addresses (tokenomics, depository, dispenser), changing the minimum accepted ETH amount, enabling/disabling tokens, and pausing/unpausing the contract.

2. **Depository Manager:**

   - Address: `depository`
   - Role: The depository manager has permission to call the `depositTokenForOLAS` function, allowing them to deposit LP tokens for OLAS. This role is restricted to the `depository` address.

3. **Dispenser Manager:**

   - Address: `dispenser`
   - Role: The dispenser manager has permission to call the `withdrawToAccount` function, allowing them to withdraw ETH and OLAS amounts to a specified account. This role is restricted to the `dispenser` address.

4. **Tokenomics Manager:**

   - Address: `tokenomics`
   - Role: The tokenomics manager has permission to call functions related to rebalancing treasury funds (`rebalanceTreasury`) and draining slashed funds from the service registry (`drainServiceSlashedFunds`). This role is restricted to the `tokenomics` address.

5. **Service Registry:**

   - Address: The service registry is managed by the `IServiceRegistry` interface, and certain functions in the contract interact with it to drain slashed funds.

6. **Service Donors:**
   - Address: Any account that calls the `depositServiceDonationsETH` function.
   - Role: Entities that donate ETH to the services specified in the `serviceIds` array. This action helps fund services in the ecosystem.

These roles are defined by the contract's access control mechanisms, and certain functions can only be executed by specific addresses to ensure proper authorization and security.

## Approach Taken-in Evaluating The Olas Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Olas Protocol.

    I start with the following contracts, which play crucial roles in the Olas protocol:

    **Main Contracts I Looked At**

I start with the following contracts, which play crucial roles in the Olas protocol:

                GovernorOLAS.sol
                OLAS.sol
                veOLAS.sol
                wveOLAS.sol
                GuardCM.sol
                GenericRegistry.sol
                UnitRegistry.sol
                Depository.sol
                Dispenser.sol
                Tokenomics.sol
                Treasury.sol

I started my analysis by examining the foundational governance contracts, such as `GovernorOLAS.sol` which define the core mechanisms for proposing and executing governance actions in the Olas protocol. Subsequently, I delved into the token-related contracts, including `OLAS.sol`, `veOLAS.sol`, and others, to understand the intricacies of the tokenomics and voting systems. Additionally, I scrutinized the registry contracts, such as `GenericRegistry.sol` and its specialized counterparts, which play a crucial role in managing units, components, and agents within the protocol. The multisig and bridge-related contracts, such as `GuardCM.sol`, `FxGovernorTunnel.sol`, `HomeMediator.sol`, `FxERC20ChildTunnel.sol`, and `FxERC20RootTunnel.sol`, were also thoroughly examined to ensure the secure flow of information and assets between different networks. Lastly, I scrutinized the tokenomics contracts, including `Depository.sol`, `Dispenser.sol`, `DonatorBlacklist.sol`, `GenericBondCalculator.sol`, `Tokenomics.sol`, and `Treasury.sol`, which collectively govern the inflation, reward mechanisms, and treasury management for OLAS tokens. This comprehensive analysis provides a holistic understanding of the Olas protocol and its underlying functionalities.

24. **Documentation Review**:

    Then went to Review [This Docs](https://www.autonolas.network/documents/whitepaper/Whitepaper%20v1.0.pdf) for a more detailed and technical explanation of Olas protocol.

25. **Compiling code and running provided tests**:

26. **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Codebase Quality

Overall, I consider the quality of the Olas protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Olas Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                                       |
| **Code Comments**                        | During the audit of the Olas contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                                 |
| **Documentation**                        | The documentation and Whitepaper of the Olas project is quite comprehensive and detailed, providing a solid overview of how Olas protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is governance: 98.72%, tokenomics: 100%, registries: 100%, lockbox-solana: methods 100% this is Good.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last.                                                                                                                                                                                                                                                                                                                  |
| **Error**                                | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| **Imports**                              | In Olas protocol the contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files. 2. openzeppelin contracts updated to a newer version.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Olas protocol. These risks encompass concentration risk in GovernorOLAS, OLAS risk and more, third-party dependency risk, and centralization risks arising.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. The `veOLAS.sol` contract implements time-weighted votes with linear decay over time. Systemic risks may arise if the chosen decay function does not align with the protocol's long-term goals or if users exploit potential vulnerabilities in the decay calculation.

2. The `wveOLAS.sol` contract uses user points stored in an array (getUserPoint). It's crucial to ensure that array indices are within bounds to prevent out-of-bounds access vulnerabilities.

3. The function `wveOLAS.sol::lockedEnd` provides the lock end time for a given account. Any miscalculation or manipulation of the lock end time could have systemic implications for the token's functionality.

4. The HomeMediator handles external calls in a loop through the `processMessageFromForeign` function. While it uses the low-level call (`target.call{value: value}(payload)`), which mitigates reentrancy risks to some extent, it is essential to ensure that external contracts called within this loop are also secure against reentrancy, DOS attacks and out-of-gas vulnerabilities.

5. The `mint` and `burn` functions in BridgedERC20 contract perform important token-related actions, but there are no corresponding events emitted to notify external systems or users about these operations.

6. The `bondCounter` is used to assign unique IDs to bonds in Depository contract. If the contract creates a very large number of bonds, there is a risk of the uint32 bondCounter overflowing. It's advisable to implement mechanisms to handle a potential overflow, such as resetting or upgrading the contract.

7. The `getCurrentPriceLP` function retrieves reserves in GenericBondCalculator contract without checking if the Uniswap pair exists or if it has sufficient liquidity. This might lead to unexpected behavior if used with invalid or empty pairs.

8. The inflation calculations assume certain rates and limitations. Any unexpected changes in these rates, especially related to OLAS tokens, could have significant consequences on the tokenomics.

9. The `Treasury.sol` contract makes assumptions about the total supply of ETH and OLAS tokens, including estimates of the time it would take to reach certain supply limits. These assumptions may become inaccurate over time, especially in dynamic markets.

### Centralization Risks:

1. The `GovernorOLAS.sol::_executor` function returns a single address. If this executor has excessive control or is a single entity, it introduces centralization risks during proposal execution.

2. The `OLAS.sol::inflationControl` function checks if the requested mint amount is within inflation boundaries based on the supply cap. The inflexibility of this mechanism might not allow for dynamic adjustments, leading to potential centralization of inflation control.

3. The `veOLAS.sol` contract allows scheduled changes in slope values, potentially controlled by the contract deployer. Misuse or centralized control over slope changes may impact the fairness and decentralization of the protocol.

4. If the `IVEOLAS` contract has governance features, the centralization of governance control could be a risk. This may lead to decision-making power being concentrated in the hands of a few entities, potentially undermining decentralization principles.

5. The `GuardCM.sol` relies on a specific proposal (governorCheckProposalId) being in a defeated state to allow certain actions. Depending on how this proposal is created and voted on, it could introduce centralization risks.

6. The `FxGovernorTunnel::changeRootGovernor` function allows changing the rootGovernor address. The authority to change this address is centralized in the contract and depends on the condition (`msg.sender != address(this)`), allowing only the contract itself to initiate the change.

7. The contract allows the owner to change various tokenomics parameters. Depending on how these changes are executed, it might introduce risks related to central control over the economic aspects of the system.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Olas protocol.**

## Conclusion

In general, The Olas protocol presents a well-designed architecture. The Olas protocol exhibits strengths comprehensive testing, and detailed documentation, we believe the team has done a good job regarding the code. However, there are areas for improvement, including governance features, profit distribution optimization, and code documentation. Systemic risks include initial governance role assignment and dynamic proposal generation, while centralization risks involve certain contracts and functions. Recommendations include enhancing governance features, optimizing profit distribution, improving documentation, addressing systemic risks, and mitigating centralization risks. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
35 hours