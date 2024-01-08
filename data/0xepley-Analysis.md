# 🛠️ Analysis - Olas
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I would follow when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://code4rena.com/audits/2023-12-olas

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-12-autonolas?tab=readme-ov-file#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Ethereum Credit Guild](https://credit-guild.gitbook.io/introduction/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| Slither report of the project for some basic analysis|
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-12-autonolas?tab=readme-ov-file#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-12-autonolas?tab=readme-ov-file#files-in-scope)||
|7|Infographic|[Figma](https://www.figma.com/)|Tried to make Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-12-autonolas?tab=readme-ov-file#files-in-scope)|Code where I should focus more|

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics

-  **Language:** language in which of the source is written
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **Comment Lines:** lines containing single or block comments
-  **Blank Lines:** Blank Lines in the source file
-  **Total Lines:** Total number of lines in the file 

## Analysis of sloc of Governance contract

[![govv.png](https://i.postimg.cc/RCXZCRhL/govv.png)](https://postimg.cc/MXfSd1yn)

## Analysis of sloc of registry contract

[![reg.png](https://i.postimg.cc/VvjT64BB/reg.png)](https://postimg.cc/30x9fXqd)

## Analysis of sloc of tokenomics contract

[![tokk.png](https://i.postimg.cc/yYLjTwd3/tokk.png)](https://postimg.cc/xcbLfpL9)

## AST Node Statics of Governance Contracts
[![g.png](https://i.postimg.cc/q7fc70qn/g.png)](https://postimg.cc/VJR07xjL)


## AST Node Statics of Registries Contracts
[![reg.png](https://i.postimg.cc/SR0j6N07/reg.png)](https://postimg.cc/N94gYtj5)


## AST Node Statics of Tokenomics Contracts
[![tok.png](https://i.postimg.cc/GtQr4cbp/tok.png)](https://postimg.cc/gLn15bX9)



## Sequence Diagram of Governance Contract
[![2-1.png](https://i.postimg.cc/g0qMFrsr/2-1.png)](https://postimg.cc/qtqc2J9d)

## Contract Integration Graph
[![dfdf.png](https://i.postimg.cc/7ZTBYKHC/dfdf.png)](https://postimg.cc/zy8wxSR5)





# Contracts Description Table


|  Contract        | Type            | Bases              | Visibility | Mutability | Modifiers |
|:----------------:|:---------------:|:------------------:|:----------:|:----------:|:---------:|
| **Depository**   | Implementation  | IErrorsTokenomics  |            |            |           |
| └                | <Constructor>   | Public ❗️          | 🛑         | NO❗️       |           |
| └                | changeOwner     | External ❗️        | 🛑         | NO❗️       |           |
| └                | changeManagers  | External ❗️        | 🛑         | NO❗️       |           |
| └                | changeBondCalculator | External ❗️  | 🛑         | NO❗️       |           |
| └                | create          | External ❗️        | 🛑         | NO❗️       |           |
| └                | close           | External ❗️        | 🛑         | NO❗️       |           |
| └                | deposit         | External ❗️        | 🛑         | NO❗️       |           |
| └                | redeem          | External ❗️        | 🛑         | NO❗️       |           |
| └                | getProducts     | External ❗️        |            | NO❗️       |           |
| └                | isActiveProduct | External ❗️        |            | NO❗️       |           |
| └                | getBonds        | External ❗️        |            | NO❗️       |           |
| └                | getBondStatus   | External ❗️        |            | NO❗️       |           |
| └                | getCurrentPriceLP | External ❗️      |            | NO❗️       |           |
| **AgentRegistry**| Implementation  | UnitRegistry       |            |            |           |
| └                | <Constructor>   | Public ❗️          | 🛑         | UnitRegistry ERC721 |    |
| └                | _checkDependencies | Internal 🔒     | 🛑         |            |           |
| └                | _getSubComponents | Internal 🔒      |            |            |           |
| └                | calculateSubComponents | External ❗️ |            | NO❗️       |           |
| **UnitRegistry** | Implementation  | GenericRegistry    |            |            |           |
| └                | <Constructor>   | Public ❗️          | 🛑         | NO❗️       |           |
| └                | _checkDependencies | Internal 🔒     | 🛑         |            |           |
| └                | create          | External ❗️        | 🛑         | NO❗️       |           |
| └                | updateHash      | External ❗️        | 🛑         | NO❗️       |           |
| └                | getUnit         | External ❗️        |            | NO❗️       |           |
| └                | getDependencies | External ❗️        |            | NO❗️       |           |
| └                | getUpdatedHashes | External ❗️      |            | NO❗️       |           |
| └                | getLocalSubComponents | External ❗️  |            | NO❗️       |           |
| └                | _getSubComponents | Internal 🔒      |            |            |           |
| └                | _calculateSubComponents | Internal 🔒 |            |            |           |
| └                | _getUnitHash    | Internal 🔒        |            |            |           |
| **GenericRegistry** | Implementation | IErrorsRegistries, ERC721 |   |            |           |
| └                | changeOwner     | External ❗️        | 🛑         | NO❗️       |           |
| └                | changeManager   | External ❗️        | 🛑         | NO❗️       |           |
| └                | exists          | External ❗️        |            | NO❗️       |           |
| └                | setBaseURI      | External ❗️        | 🛑         | NO❗️       |           |
| └                | tokenByIndex    | External ❗️        |            | NO❗️       |           |
| └                | _toHex16        | Internal 🔒        |            |            |           |
| └                | _getUnitHash    | Internal 🔒        |            |            |           |
| └                | tokenURI        | Public ❗️          |            | NO❗️       |           |
| **GovernorOLAS**  | Implementation  | Governor, GovernorSettings, GovernorCompatibilityBravo, GovernorVotes, GovernorVotesQuorumFraction, GovernorTimelockControl ||||
| └                | <Constructor>   | Public ❗️          | 🛑         | Governor GovernorSettings GovernorVotes GovernorVotesQuorumFraction GovernorTimelockControl | |
| └                | state           | Public ❗️          |            | NO❗️       |           |
| └                | propose         | Public ❗️          | 🛑         | NO❗️       |           |
| └                | proposalThreshold | Public ❗️       |            | NO❗️       |           |
| └                | _execute        | Internal 🔒        | 🛑         |            |           |
| └                | _cancel         | Internal 🔒        | 🛑         |            |           |
| └                | _executor       | Internal 🔒        |            |            |           |
| └                | supportsInterface | Public ❗️       |            | NO❗️       |           |
| **Dispenser**    | Implementation  | IErrorsTokenomics  |            |            |           |
| └                | <Constructor>   | Public ❗️          | 🛑         | NO❗️       |           |
| └                | changeOwner     | External ❗️        | 🛑         | NO❗️       |           |
| └                | changeManagers  | External ❗️        | 🛑         | NO❗️       |           |
| └                | claimOwnerIncentives | External ❗️  | 🛑         | NO❗️       |           |
| **Tokenomics**   | Implementation  | TokenomicsConstants, IErrorsTokenomics | | |            |
| └                | <Constructor>   | Public ❗️          | 🛑         | TokenomicsConstants |    |
| └                | initializeTokenomics | External ❗️  | 🛑         | NO❗️       |           |
| └                | tokenomicsImplementation | External ❗️ | | NO❗️       |
| └                | changeTokenomicsImplementation | External ❗️ | 🛑 | NO❗️ |           |
| └                | changeOwner     | External ❗️        | 🛑         | NO❗️       |           |
| └                | changeManagers  | External ❗️        | 🛑         | NO❗️       |           |
| └                | changeRegistries | External ❗️      | 🛑         | NO❗️       |           |
| └                | changeDonatorBlacklist | External ❗️ | 🛑 | NO❗️ |           |
| └                | changeTokenomicsParameters | External ❗️ | 🛑 | NO❗️ |           |
| └                | changeIncentiveFractions | External ❗️ | 🛑 | NO❗️ |           |
| └                | reserveAmountForBondProgram | External ❗️ | 🛑 | NO❗️ |           |
| └                | refundFromBondProgram | External ❗️ | 🛑 | NO❗️ |           |
| └                | _finalizeIncentivesForUnitId | Internal 🔒 | 🛑 | | |
| └                | _trackServiceDonations | Internal 🔒 | 🛑 | | |
| └                | trackServiceDonations | External ❗️ | 🛑 | NO❗️ |           |
| └                | _calculateIDF   | Internal 🔒        |            |            |           |
| └                | checkpoint      | External ❗️        | 🛑         | NO❗️       |           |
| └                | accountOwnerIncentives | External ❗️ | 🛑 | NO❗️ |           |
| └                | getOwnerIncentives | External ❗️   |            | NO❗️       |           |
| └                | getInflationPerEpoch | External ❗️  |            | NO❗️       |           |
| └                | getUnitPoint    | External ❗️        |            | NO❗️       |           |
| └                | getIDF          | External ❗️        |            | NO❗️       |           |
| └                | getLastIDF      | External ❗️        |            | NO❗️       |           |



|  Symbol  |  Meaning  |
|:--------:|-----------|
|    🛑    | Function can modify state |
|    💵    | Function is payable |




## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. In particular, tests have been written successfully.

-   2) Overall line coverage percentage provided by your tests : 99.56

### What could they have done better?

-  1) In order to understand the test scenarios and develop more effective test scenarios, the following bob, alice and other roles are can be defined one by one, in this way role definitions increase the quality and readability in tests

```solidity

 // Sample labels
vm.label(bob, 'bob');
vm.label(alice, 'alice');
vm.label(DEPLOYER, 'deployer');
vm.label(USD_OWNER, 'usd owner');
vm.label(POOL_PROXY, 'lending pool');
```


-  3) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


[![test-cases.jpg](https://i.postimg.cc/1zgD5wCt/test-cases.jpg)](https://postimg.cc/v1s40gdF)

Ref:https://xin-xia.github.io/publication/icse194.pdf

[![sdf.png](https://i.postimg.cc/pXZH2Xdh/sdf.png)](https://postimg.cc/nCC52xBp)




# Test-case analysis of Governance Contracts

### General Information
- **Solc Version:** 0.8.23
- **Optimizer Enabled:** true
- **Runs:** 1,000,000
- **Block Limit:** 30,000,000 gas

### Methods Overview

|   | Contract                | Method                    | Avg        | # Calls    |
|---|-------------------------|---------------------------|------------|------------|
| ☑ | BridgedERC20            | approve                   | 46,103     | 3          |
| ☑ | BridgedERC20            | changeOwner               | 28,073     | 2          |
| ☑ | BridgedERC20            | mint                      | 70,325     | 4          |
| ☑ | BrokenERC20             | approve                   | 46,079     | 1          |
| ☑ | BrokenERC20             | mint                      | 68,219     | 1          |
| ☑ | ChildMockERC20          | approve                   | 46,103     | 6          |
| ☑ | ChildMockERC20          | mint                      | 70,406     | 11         |
| ☑ | FxERC20ChildTunnel      | deposit                   | 59,904     | 3          |
| ☑ | FxERC20ChildTunnel      | depositTo                 | 60,404     | 2          |
| ☑ | FxERC20ChildTunnel      | setFxRootTunnel           | 44,047     | 12         |
| ☑ | FxERC20RootTunnel       | setFxChildTunnel          | 44,048     | 13         |
| ☑ | FxERC20RootTunnel       | withdraw                  | 86,993     | 1          |
| ☑ | FxERC20RootTunnel       | withdrawTo                | 87,358     | 1          |
| ☑ | FxGovernorTunnel        | processMessageFromRoot    | 77,584     | 7          |
| ☑ | FxRootMock              | setRootTunnel             | 42,400     | 12         |
| ☑ | GnosisSafe              | execTransaction           | 137,193    | 15         |
| ☑ | GnosisSafeProxyFactory  | createProxyWithNonce      | 421,116    | 32         |
| ☑ | GovernorOLAS            | cancel                    | 116,825    | 2          |
| ☑ | GovernorOLAS            | castVote                  | 106,684    | 6          |
| ☑ | GovernorOLAS            | execute                   | 160,945    | 2          |
| ☑ | GovernorOLAS            | propose                   | 277,306    | 11         |
| ☑ | GovernorOLAS            | queue                     | 129,456    | 6          |
| ☑ | GuardCM                 | checkTransaction          | 365,047    | 2          |
| ☑ | MockAMBMediator         | changeForeignGovernor      | 21,676     | 1          |
| ☑ | MockAMBMediator         | changeHomeMediator         | 26,759     | 12         |
| ☑ | MockAMBMediator         | processMessageFromForeign  | 85,308     | 7          |
| ☑ | MockTimelockCM          | execute                    | 57,465     | 14         |
| ☑ | OLAS                    | approve                    | 46,198     | 62         |
| ☑ | OLAS                    | burn                       | 33,620     | 2          |
| ☑ | OLAS                    | changeMinter               | 30,096     | 21         |
| ☑ | OLAS                    | changeOwner                | 28,114     | 3          |
| ☑ | OLAS                    | decreaseAllowance          | 26,936     | 2          |
| ☑ | OLAS                    | increaseAllowance          | 29,201     | 1          |
| ☑ | OLAS                    | mint                       | 65,721     | 84         |
| ☑ | OLAS                    | permit                     | 74,414     | 1          |
| ☑ | OLAS                    | transfer                   | 50,076     | 32         |
| ☑ | OLAS                    | transferFrom               | 48,792     | 5          |
| ☑ | Timelock                | cancel                     | 25,739     | 2          |
| ☑ | Timelock                | grantRole                  | 51,350     | 22         |
| ☑ | Timelock                | renounceRole               | 24,912     | 2          |
| ☑ | Timelock                | schedule                   | 54,757     | 2          |
| ☑ | veOLAS                  | checkpoint                 | 234,664    | 3          |
| ☑ | veOLAS                  | createLock                 | 256,967    | 34         |
| ☑ | veOLAS                  | createLockFor              | 267,318    | 2          |
| ☑ | veOLAS                  | depositFor                 | 163,155    | 20         |
| ☑ | veOLAS                  | increaseAmount             | 171,808    | 13         |





# Test-case analysis of Tokenomics Contracts

### General Information
- **Solc Version:** 0.8.20
- **Optimizer Enabled:** true
- **Runs:** 4000
- **Block Limit:** 30,000,000 gas




| Contract            | Method                            | Avg         | # calls     | ✓             |
|---------------------|-----------------------------------|-------------|-------------|---------------|
| DepositAttacker     | flashAttackDepositImmuneClone     | 452716      | 1           | ✓             |
| DepositAttacker     | flashAttackDepositImmuneOriginal  | 423062      | 1           | ✓             |
| Depository          | changeBondCalculator              | 26932       | 4           | ✓             |
| Depository          | changeManagers                    | 30440       | 4           | ✓             |
| Depository          | changeOwner                       | 29799       | 3           | ✓             |
| Depository          | close                             | 39061       | 15          | ✓             |
| Depository          | create                            | 94673       | 81          | ✓             |
| Depository          | deposit                           | 194571      | 22          | ✓             |
| Depository          | redeem                            | 54187       | 12          | ✓             |
| Dispenser           | changeManagers                    | 31969       | 26          | ✓             |
| Dispenser           | changeOwner                       | 28069       | 1           | ✓             |
| Dispenser           | claimOwnerIncentives              | 128570      | 8           | ✓             |
| DonatorBlacklist    | changeOwner                       | 28069       | 1           | ✓             |
| DonatorBlacklist    | setDonatorsStatuses               | 49810       | 2           | ✓             |
| ERC20Token          | approve                           | 42944       | 194         | ✓             |
| ERC20Token          | changeMinter                      | 28993       | 134         | ✓             |
| ERC20Token          | mint                              | 63871       | 333         | ✓             |
| MockRegistry        | changeUnitOwner                   | 29221       | 34          | ✓             |
| MockTokenomics      | setServiceRegistry                | 43910       | 2           | ✓             |
| MockVE              | createLock                        | 46035       | 10          | ✓             |
| MockVE              | setWeightedBalance                | 23744       | 9           | ✓             |
| ReentrancyAttacker  | setAttackMode                     | 21721       | 1           | ✓             |
| ReentrancyAttacker  | setTransfer                       | 26606       | 2           | ✓             |
| Tokenomics          | accountOwnerIncentives            | 92790       | 1           | ✓             |
| Tokenomics          | changeDonatorBlacklist            | 33046       | 5           | ✓             |
| Tokenomics          | changeIncentiveFractions          | 98615       | 78          | ✓             |
| Tokenomics          | changeManagers                    | 36978       | 71          | ✓             |
| Tokenomics          | changeRegistries                  | 38847       | 2           | ✓             |
| Tokenomics          | changeTokenomicsImplementation    | 32923       | 1           | ✓             |
| Tokenomics          | changeTokenomicsParameters        | 48956       | 14          | ✓             |
| Tokenomics          | checkpoint                        | 152864      | 64          | ✓             |
| Tokenomics          | initializeTokenomics              | 344397      | 64          | ✓             |
| Tokenomics          | trackServiceDonations             | 154731      | 3           | ✓             |
| Treasury            | changeManagers                    | 30889       | 117         | ✓             |
| Treasury            | changeMinAcceptedETH              | 29852       | 1           | ✓             |
| Treasury            | changeOwner                       | 28156       | 1           | ✓             |
| Treasury            | depositServiceDonationsETH        | 211895      | 37          | ✓             |
| Treasury            | depositTokenForOLAS               | 138133      | 5           | ✓             |
| Treasury            | disableToken                      | 26666       | 3           | ✓             |
| Treasury            | drainServiceSlashedFunds          | 48931       | 1           | ✓             |
| Treasury            | enableToken                       | 47077       | 101         | ✓             |
| Treasury            | pause                             | 29180       | 1           | ✓             |
| Treasury            | rebalanceTreasury                 | 32468       | 4           | ✓             |
| Treasury            | unpause                           | 29230       | 1           | ✓             |
| Treasury            | withdraw                          | 39486       | 2           | ✓             |
| UniswapV2ERC20      | approve                           | 46208       | 51          | ✓             |
| UniswapV2ERC20      | transfer                          | 48812       | 31          | ✓             |
| UniswapV2Factory    | createPair                        | 3258226     | 43          | ✓             |
| UniswapV2Pair       | approve                           | 46265       | 80          | ✓             |
| UniswapV2Pair       | transfer                          | 49592       | 46          | ✓             |
| UniswapV2Router02   | addLiquidity                      | 237216      | 42          | ✓             |
| UniswapV2Router02   | removeLiquidity                   | 160521      | 4           | ✓             |
| UniswapV2Router02   | swapExactTokensForTokens          | 104160      | 4           | ✓             |
| ZuniswapV2Factory   | createPair                        | 1919723     | 28          | ✓             |
| ZuniswapV2Router    | addLiquidity                      | 248874      | 27          | ✓             |



# Test-case analysis of Tokenomics Contracts

### General Information
- **Solc Version:** 0.8.21
- **Optimizer Enabled:** true
- **Runs:** 750
- **Block Limit:** 30,000,000 gas


| Contract                       | Method                | Avg        | # calls     | ✓             |
|--------------------------------|-----------------------|------------|-------------|---------------|
| AgentRegistry                  | changeManager         | 47328      | 14          | ✓             |
| AgentRegistry                  | create                | 251826     | 77          | ✓             |
| AgentRegistry                  | setBaseURI            | 31830      | 1           | ✓             |
| AgentRegistry                  | updateHash            | 67389      | 3           | ✓             |
| ComponentRegistry              | changeManager         | 46156      | 36          | ✓             |
| ComponentRegistry              | create                | 267904     | 181         | ✓             |
| ComponentRegistry              | setBaseURI            | 31842      | 2           | ✓             |
| ComponentRegistry              | transferFrom          | 54553      | 1           | ✓             |
| ComponentRegistry              | updateHash            | 67389      | 3           | ✓             |
| ComponentRegistryTest          | changeManager         | 47262      | 1           | ✓             |
| ComponentRegistryTest          | create                | 171917     | 3           | ✓             |
| GnosisSafe                     | execTransaction       | 112490     | 7           | ✓             |
| GnosisSafeMultisig             | create                | 333620     | 6           | ✓             |
| GnosisSafeProxyFactory         | createProxyWithNonce  | 229458     | 4           | ✓             |
| GnosisSafeSameAddressMultisig  | create                | 43583      | 1           | ✓             |
| ReentrancyAttacker             | setAttackOnCreate     | 43696      | 1           | ✓             |
| RegistriesManager              | create                | 210012     | 4           | ✓             |
| RegistriesManager              | pause                 | 27571      | 1           | ✓             |
| RegistriesManager              | unpause               | 27581      | 1           | ✓             |
| RegistriesManager              | updateHash            | 67765      | 4           | ✓             |



## d) Security Approach of the Project

### Successful current security understanding of the project;

1- The project underwent four audits, excluding this innovative assessments on Code4rena, where multiple auditors are scrutinizing the code.

### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)


2- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

3- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/


4- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

5- I also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

6 - As the project team, you can consider applying the multi-stage audit model.

[![sla.png](https://i.postimg.cc/nhR0kN3w/sla.png)](https://postimg.cc/Sn96Q1FW)

Read more about the MPA model;
https://mpa.solodit.xyz/

7 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19




## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-12-autonolas/blob/main/bot-report.md

**Previous Audits**
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/docs/Vulnerabilities_list_governance.pdf

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/docs/Vulnerabilities_list_registries.pdf

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/docs/Vulnerabilities_list_tokenomics.pdf

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/docs/Vulnerabilities_list_tokenomics-solana.pdf


##  f) Packages and Dependencies Analysis 📦

| Package | Version | Usage | 
| --- | --- | --- | 
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts) | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts) |  Project uses version `4.8.3`; consider updating to `5.0.1` 
| [`solhint`](https://github.com/protofire/solhint) | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts) |  Project relies on version `3.4.0`; consider upgrading to `4.1.1` 


## g) New insights and learning from this audit 

The Autonolas protocol is a comprehensive system divided into three main components: governance, registries, and tokenomics. Each plays a distinct role in the protocol's operation and governance.

1. **Governance**: It's designed with multiple control points to manage the Autonolas protocol effectively. Key components include a governance module, a Timelock, and veOLAS (virtualized claim on OLAS tokens). This structure allows the community to propose, vote on, and implement changes. The governance token, veOLAS, represents locked OLAS tokens and uses a voting system where power is determined by the amount of OLAS locked and the duration of the lock. This system is akin to veCRV in Curve Finance. There's also a community-owned multisig wallet (CM) which can execute some changes, bypassing the usual governance process, but with a guard mechanism for alignment and reversibility.

2. **Cross-Chain Governance**: Due to its multi-chain nature, Autonolas implements cross-chain governance to facilitate actions between Ethereum and L2 networks like Polygon and Gnosis Chain. This is achieved through mechanisms like FxPortal for Polygon and Arbitrary Message Bridge (AMB) for Gnosis Chain. Additionally, contracts facilitate L1-L2 token transfers, particularly for tokens created natively on Polygon and bridged to Ethereum.

3. **Registries**: This feature allows developers to register and manage their off-chain code on-chain using NFTs. The focus is on registering and managing agents and components, representing them uniquely on the blockchain.

4. **Tokenomics**: The system aims to grow useful code and capital proportionally. This includes incentivizing the creation and on-chain registration of agents and components through a donations system. Additionally, it encourages liquidity providers to sell their liquidity pairs to the protocol for OLAS at a discount, enhancing the protocol's liquidity.

5. **Bonding on Different Chains**: The protocol involves moving OLAS tokens across chains using low-trust, secure bridges. Liquidity pools are created on the target chain, and LP tokens are transferred back to Ethereum for bonding. A special mechanism is used for bonding on Solana, involving Orca AMM and a liquidity-lockbox contract to maintain compatibility with the existing depository model.


Note: I didn't tracked the time, the time I mentioned is just an estimate


### Time spent:
6 hours