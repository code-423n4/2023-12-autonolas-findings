**Note:** I've removed the instances reported by the winning bot and 4naly3er

## Summary

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-else-block-not-required)] | `else`-block not required | 1 | 
| [[N&#x2011;02](#n02-public-functions-not-called-by-the-contract-should-be-declared-external-instead)] | `public` functions not called by the contract should be declared `external` instead | 2 | 
| [[N&#x2011;03](#n03-common-functions-should-be-refactored-to-a-common-base-contract)] | Common functions should be refactored to a common base contract | 8 | 
| [[N&#x2011;04](#n04-consider-making-contracts-upgradeable)] | Consider making contracts `Upgradeable` | 22 | 
| [[N&#x2011;05](#n05-consider-moving-duplicated-strings-to-constants)] | Consider moving duplicated strings to constants | 4 | 
| [[N&#x2011;06](#n06-consider-using-delete-rather-than-assigning-zerofalse-to-clear-values)] | Consider using `delete` rather than assigning zero/false to clear values | 3 | 
| [[N&#x2011;07](#n07-consider-using-descriptive-constants-when-passing-zero-as-a-function-argument)] | Consider using descriptive `constant`s when passing zero as a function argument | 22 | 
| [[N&#x2011;08](#n08-consider-using-named-function-arguments)] | Consider using named function arguments | 19 | 
| [[N&#x2011;09](#n09-consider-using-named-returns)] | Consider using named returns | 63 | 
| [[N&#x2011;10](#n10-constants-in-comparisons-should-appear-on-the-left-side)] | Constants in comparisons should appear on the left side | 10 | 
| [[N&#x2011;11](#n11-contract-timekeeping-will-break-earlier-than-the-ethereum-network-itself-will-stop-working)] | Contract timekeeping will break earlier than the Ethereum network itself will stop working | 1 | 
| [[N&#x2011;12](#n12-contractslibrariesinterfaces-should-each-be-defined-in-separate-files)] | Contracts/libraries/interfaces should each be defined in separate files | 12 | 
| [[N&#x2011;13](#n13-function-state-mutability-can-be-restricted-to-view)] | Function state mutability can be restricted to `view` | 6 | 
| [[N&#x2011;14](#n14-inconsistent-spacing-in-comments)] | Inconsistent spacing in comments | 2 | 
| [[N&#x2011;15](#n15-interfaces-should-be-defined-in-separate-files-from-their-usage)] | Interfaces should be defined in separate files from their usage | 1 | 
| [[N&#x2011;16](#n16-named-imports-of-parent-contracts-are-missing)] | Named imports of parent contracts are missing | 17 | 
| [[N&#x2011;17](#n17-natspec-event-param-tag-is-missing)] | NatSpec: Event `@param` tag is missing | 149 | 
| [[N&#x2011;18](#n18-natspec-function-param-tag-is-missing)] | NatSpec: Function `@param` tag is missing | 1 | 
| [[N&#x2011;19](#n19-natspec-function-return-tag-is-missing)] | NatSpec: Function `@return` tag is missing | 8 | 
| [[N&#x2011;20](#n20-not-using-the-named-return-variables-anywhere-in-the-function-is-confusing)] | Not using the named return variables anywhere in the function is confusing | 4 | 
| [[N&#x2011;21](#n21-style-guide-local-and-state-variables-should-be-named-using-lowercamelcase)] | Style guide: Local and state variables should be named using lowerCamelCase | 1 | 
| [[N&#x2011;22](#n22-style-guide-non-externalpublic-variable-names-should-begin-with-an-underscore)] | Style guide: Non-`external`/`public` variable names should begin with an underscore | 3 | 
| [[N&#x2011;23](#n23-style-guide-top-level-declarations-should-be-separated-by-at-least-two-lines)] | Style guide: Top-level declarations should be separated by at least two lines | 2 | 
| [[N&#x2011;24](#n24-unnecessary-cast)] | Unnecessary cast | 3 | 
| [[N&#x2011;25](#n25-unused-error-definition)] | Unused `error` definition | 9 | 
| [[N&#x2011;26](#n26-unused-public-contract-variable)] | Unused `public` contract variable | 9 | 
| [[N&#x2011;27](#n27-use-bytesconcat-on-bytes-instead-of-abiencodepacked-for-clearer-semantic-meaning)] | Use `bytes.concat()` on bytes instead of `abi.encodePacked()` for clearer semantic meaning | 1 | 
| [[N&#x2011;28](#n28-use-bit-shifts-to-represent-powers-of-two)] | Use bit shifts to represent powers of two | 7 | 
| [[N&#x2011;29](#n29-use-of-override-is-unnecessary)] | Use of `override` is unnecessary | 14 | 

Total: 404 instances over 29 issues





## Non-critical Issues


### [N&#x2011;01] `else`-block not required
One level of nesting can be removed by not having an `else` block when the `if`-block returns, and `if (foo) { return 1; } else { return 2; }` becomes `if (foo) { return 1; } return 2;`. A following `else if` can become `if`

*There is one instance of this issue:*

```solidity
File: contracts/veOLAS.sol

265                  if (tStep == block.timestamp) {
266                      lastPoint.blockNumber = uint64(block.number);
267                      lastPoint.balance = curSupply;
268:                     break;

```
*GitHub*: [265](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L265-L274)



### [N&#x2011;02] `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There are 2 instances of this issue:*

```solidity
File: contracts/veOLAS.sol

761:      function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
*GitHub*: [761](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L761)

```solidity
File: contracts/GenericRegistry.sol

135:      function tokenURI(uint256 unitId) public view virtual override returns (string memory) {

```
*GitHub*: [135](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L135)



### [N&#x2011;03] Common functions should be refactored to a common base contract
The functions below have the same implementation as is seen in other files. The functions should be refactored into functions of a common base contract

*There are 8 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/GenericManager.sol

20       function changeOwner(address newOwner) external virtual {
21           // Check for the ownership
22           if (msg.sender != owner) {
23               revert OwnerOnly(msg.sender, owner);
24           }
25   
26           // Check for the zero address
27           if (newOwner == address(0)) {
28               revert ZeroAddress();
29           }
30   
31           owner = newOwner;
32           emit OwnerUpdated(newOwner);
33:      }

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L20-L33)

```solidity
File: contracts/GenericRegistry.sol

37       function changeOwner(address newOwner) external virtual {
38           // Check for the ownership
39           if (msg.sender != owner) {
40               revert OwnerOnly(msg.sender, owner);
41           }
42   
43           // Check for the zero address
44           if (newOwner == address(0)) {
45               revert ZeroAddress();
46           }
47   
48           owner = newOwner;
49           emit OwnerUpdated(newOwner);
50:      }

```
*GitHub*: [37](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L37-L50)

```solidity
File: contracts/Depository.sol

123      function changeOwner(address newOwner) external {
124          // Check for the contract ownership
125          if (msg.sender != owner) {
126              revert OwnerOnly(msg.sender, owner);
127          }
128  
129          // Check for the zero address
130          if (newOwner == address(0)) {
131              revert ZeroAddress();
132          }
133  
134          owner = newOwner;
135          emit OwnerUpdated(newOwner);
136:     }

143      function changeManagers(address _tokenomics, address _treasury) external {
144          // Check for the contract ownership
145          if (msg.sender != owner) {
146              revert OwnerOnly(msg.sender, owner);
147          }
148  
149          // Change Tokenomics contract address
150          if (_tokenomics != address(0)) {
151              tokenomics = _tokenomics;
152              emit TokenomicsUpdated(_tokenomics);
153          }
154          // Change Treasury contract address
155          if (_treasury != address(0)) {
156              treasury = _treasury;
157              emit TreasuryUpdated(_treasury);
158          }
159:     }

```
*GitHub*: [123](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L123-L136), [143](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L143-L159)

```solidity
File: contracts/Dispenser.sol

46       function changeOwner(address newOwner) external {
47           // Check for the contract ownership
48           if (msg.sender != owner) {
49               revert OwnerOnly(msg.sender, owner);
50           }
51   
52           // Check for the zero address
53           if (newOwner == address(0)) {
54               revert ZeroAddress();
55           }
56   
57           owner = newOwner;
58           emit OwnerUpdated(newOwner);
59:      }

```
*GitHub*: [46](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L46-L59)

```solidity
File: contracts/DonatorBlacklist.sol

36       function changeOwner(address newOwner) external {
37           // Check for the contract ownership
38           if (msg.sender != owner) {
39               revert OwnerOnly(msg.sender, owner);
40           }
41   
42           // Check for the zero address
43           if (newOwner == address(0)) {
44               revert ZeroAddress();
45           }
46   
47           owner = newOwner;
48           emit OwnerUpdated(newOwner);
49:      }

```
*GitHub*: [36](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L36-L49)

```solidity
File: contracts/Tokenomics.sol

404      function changeOwner(address newOwner) external {
405          // Check for the contract ownership
406          if (msg.sender != owner) {
407              revert OwnerOnly(msg.sender, owner);
408          }
409  
410          // Check for the zero address
411          if (newOwner == address(0)) {
412              revert ZeroAddress();
413          }
414  
415          owner = newOwner;
416          emit OwnerUpdated(newOwner);
417:     }

```
*GitHub*: [404](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L404-L417)

```solidity
File: contracts/Treasury.sol

137      function changeOwner(address newOwner) external {
138          // Check for the contract ownership
139          if (msg.sender != owner) {
140              revert OwnerOnly(msg.sender, owner);
141          }
142  
143          // Check for the zero address
144          if (newOwner == address(0)) {
145              revert ZeroAddress();
146          }
147  
148          owner = newOwner;
149          emit OwnerUpdated(newOwner);
150:     }

```
*GitHub*: [137](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L137-L150)

</details>





### [N&#x2011;04] Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of _significantly_ increasing centralization.

*There are 22 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/OLAS.sol

17:  contract OLAS is ERC20 {

```
*GitHub*: [17](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L17-L17)

```solidity
File: contracts/Timelock.sol

9:   contract Timelock is TimelockController {

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/Timelock.sol#L9-L9)

```solidity
File: contracts/bridges/BridgedERC20.sol

18:  contract BridgedERC20 is ERC20 {

```
*GitHub*: [18](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/BridgedERC20.sol#L18-L18)

```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

24:  contract FxERC20ChildTunnel is FxBaseChildTunnel {

```
*GitHub*: [24](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L24-L24)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

24:  contract FxERC20RootTunnel is FxBaseRootTunnel {

```
*GitHub*: [24](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L24-L24)

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

46:  contract FxGovernorTunnel is IFxMessageProcessor {

```
*GitHub*: [46](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L46-L46)

```solidity
File: contracts/bridges/HomeMediator.sol

46:  contract HomeMediator {

```
*GitHub*: [46](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L46-L46)

```solidity
File: contracts/multisigs/GuardCM.sol

88:  contract GuardCM {

```
*GitHub*: [88](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L88-L88)

```solidity
File: contracts/veOLAS.sol

86:  contract veOLAS is IErrors, IVotes, IERC20, IERC165 {

```
*GitHub*: [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L86-L86)

```solidity
File: contracts/wveOLAS.sol

130  contract wveOLAS {
131:     // veOLAS address

```
*GitHub*: [130](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L130-L131)

```solidity
File: contracts/AgentRegistry.sol

9    contract AgentRegistry is UnitRegistry {
10:      // Component registry

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L9-L10)

```solidity
File: contracts/ComponentRegistry.sol

8    contract ComponentRegistry is UnitRegistry {
9:       // Component registry version number

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L8-L9)

```solidity
File: contracts/RegistriesManager.sol

9    contract RegistriesManager is GenericManager {
10:      // Component registry address

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/RegistriesManager.sol#L9-L10)

```solidity
File: contracts/multisigs/GnosisSafeMultisig.sol

24   contract GnosisSafeMultisig {
25:      // Selector of the Gnosis Safe setup function

```
*GitHub*: [24](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L24-L25)

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

50   contract GnosisSafeSameAddressMultisig {
51       // Default data size to be parsed as an address of a Gnosis Safe multisig proxy address
52:      // This exact size suggests that all the changes to the multisig have been performed and only validation is needed

```
*GitHub*: [50](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L50-L52)

```solidity
File: contracts/Depository.sol

62:  contract Depository is IErrorsTokenomics {

```
*GitHub*: [62](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L62-L62)

```solidity
File: contracts/Dispenser.sol

11:  contract Dispenser is IErrorsTokenomics {

```
*GitHub*: [11](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L11-L11)

```solidity
File: contracts/DonatorBlacklist.sol

20:  contract DonatorBlacklist {

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L20-L20)

```solidity
File: contracts/GenericBondCalculator.sol

20   contract GenericBondCalculator {
21:      // OLAS contract address

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L20-L21)

```solidity
File: contracts/Tokenomics.sol

118: contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {

```
*GitHub*: [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L118-L118)

```solidity
File: contracts/TokenomicsProxy.sol

26   contract TokenomicsProxy {
27:      // Code position in storage is keccak256("PROXY_TOKENOMICS") = "0xbd5523e7c3b6a94aa0e3b24d1120addc2f95c7029e097b466b2bedc8d4b4362f"

```
*GitHub*: [26](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsProxy.sol#L26-L27)

```solidity
File: contracts/Treasury.sol

39:  contract Treasury is IErrorsTokenomics {

```
*GitHub*: [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L39-L39)

</details>





### [N&#x2011;05] Consider moving duplicated strings to constants


*There are 4 instances of this issue:*

```solidity
File: contracts/AgentRegistry.sol

13:      string public constant VERSION = "1.0.0";

```
*GitHub*: [13](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L13-L13)

```solidity
File: contracts/ComponentRegistry.sol

10:      string public constant VERSION = "1.0.0";

```
*GitHub*: [10](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L10-L10)

```solidity
File: contracts/Depository.sol

77:      string public constant VERSION = "1.0.1";

```
*GitHub*: [77](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L77-L77)

```solidity
File: contracts/TokenomicsConstants.sol

11:      string public constant VERSION = "1.0.1";

```
*GitHub*: [11](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L11-L11)



### [N&#x2011;06] Consider using `delete` rather than assigning zero/false to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic

*There are 3 instances of this issue:*

```solidity
File: contracts/GenericManager.sol

53:          paused = false;

```
*GitHub*: [53](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L53-L53)

```solidity
File: contracts/Treasury.sol

455:                 success = false;

518:             mapEnabledTokens[token] = false;

```
*GitHub*: [455](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L455-L455), [518](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L518-L518)



### [N&#x2011;07] Consider using descriptive `constant`s when passing zero as a function argument
Passing zero as a function argument can sometimes result in a security issue (e.g. passing zero as the slippage parameter). Consider using a `constant` variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

*There are 22 instances of this issue:*

```solidity
File: contracts/veOLAS.sol

139:         mapSupplyPoints[0] = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);

139:         mapSupplyPoints[0] = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);

139:         mapSupplyPoints[0] = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);

215:             lastPoint = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);

215:             lastPoint = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);

215:             lastPoint = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);

321:         _checkpoint(address(0), LockedBalance(0, 0), LockedBalance(0, 0), uint128(supply));

321:         _checkpoint(address(0), LockedBalance(0, 0), LockedBalance(0, 0), uint128(supply));

321:         _checkpoint(address(0), LockedBalance(0, 0), LockedBalance(0, 0), uint128(supply));

321:         _checkpoint(address(0), LockedBalance(0, 0), LockedBalance(0, 0), uint128(supply));

321:         _checkpoint(address(0), LockedBalance(0, 0), LockedBalance(0, 0), uint128(supply));

396:         _depositFor(account, amount, 0, lockedBalance, DepositType.DEPOSIT_FOR_TYPE);

478:         _depositFor(msg.sender, amount, 0, lockedBalance, DepositType.INCREASE_LOCK_AMOUNT);

506:         _depositFor(msg.sender, 0, unlockTime, lockedBalance, DepositType.INCREASE_UNLOCK_TIME);

517:         mapLockedBalances[msg.sender] = LockedBalance(0, 0);

517:         mapLockedBalances[msg.sender] = LockedBalance(0, 0);

528:         _checkpoint(msg.sender, lockedBalance, LockedBalance(0, 0), uint128(supplyAfter));

528:         _checkpoint(msg.sender, lockedBalance, LockedBalance(0, 0), uint128(supplyAfter));

650:         (point, minPointNumber) = _findPointByBlock(blockNumber, address(0));

728:         (PointVoting memory sPoint, ) = _findPointByBlock(blockNumber, address(0));

```
*GitHub*: [139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L139-L139), [139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L139-L139), [139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L139-L139), [215](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L215-L215), [215](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L215-L215), [215](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L215-L215), [321](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L321-L321), [321](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L321-L321), [321](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L321-L321), [321](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L321-L321), [321](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L321-L321), [396](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L396-L396), [478](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L478-L478), [506](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L506-L506), [517](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L517-L517), [517](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L517-L517), [528](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L528-L528), [528](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L528-L528), [650](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L650-L650), [728](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L728-L728)

```solidity
File: contracts/Tokenomics.sol

329:         uint256 _inflationPerSecond = getInflationForYear(0) / zeroYearSecondsLeft;

```
*GitHub*: [329](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L329-L329)

```solidity
File: contracts/Treasury.sol

408:                 revert TransferFailed(address(0), address(this), account, accountRewards);

```
*GitHub*: [408](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L408-L408)



### [N&#x2011;08] Consider using named function arguments
When calling functions in external contracts with multiple arguments, consider using [named](https://docs.soliditylang.org/en/latest/control-structures.html#function-calls-with-named-parameters) function parameters, rather than positional ones.

*There are 19 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

77:          IERC20(rootToken).mint(to, amount);

```
*GitHub*: [77](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L77-L77)

```solidity
File: contracts/wveOLAS.sol

197:             uPoint = IVEOLAS(ve).getUserPoint(account, idx);

216:             balance = IVEOLAS(ve).getPastVotes(account, blockNumber);

236:             balance = IVEOLAS(ve).balanceOfAt(account, blockNumber);

```
*GitHub*: [197](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L197-L197), [216](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L216-L216), [236](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L236-L236)

```solidity
File: contracts/RegistriesManager.sol

39:              unitId = IRegistry(componentRegistry).create(unitOwner, unitHash, dependencies);

41:              unitId = IRegistry(agentRegistry).create(unitOwner, unitHash, dependencies);

52:              success = IRegistry(componentRegistry).updateHash(msg.sender, unitId, unitHash);

54:              success = IRegistry(agentRegistry).updateHash(msg.sender, unitId, unitHash);

```
*GitHub*: [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/RegistriesManager.sol#L39-L39), [41](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/RegistriesManager.sol#L41-L41), [52](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/RegistriesManager.sol#L52-L52), [54](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/RegistriesManager.sol#L54-L54)

```solidity
File: contracts/multisigs/GnosisSafeMultisig.sol

106:         multisig = IGnosisSafeProxyFactory(gnosisSafeProxyFactory).createProxyWithNonce(gnosisSafe, safeParams, nonce);

```
*GitHub*: [106](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L106-L106)

```solidity
File: contracts/Depository.sol

320:         payout = IGenericBondCalculator(bondCalculator).calculatePayoutOLAS(tokenAmount, product.priceLP);

337:         ITreasury(treasury).depositTokenForOLAS(msg.sender, tokenAmount, token, payout);

```
*GitHub*: [320](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L320-L320), [337](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L337-L337)

```solidity
File: contracts/Dispenser.sol

99:          (reward, topUp) = ITokenomics(tokenomics).accountOwnerIncentives(msg.sender, unitTypes, unitIds);

104:             success = ITreasury(treasury).withdrawToAccount(msg.sender, reward, topUp);

```
*GitHub*: [99](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L99-L99), [104](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L104-L104)

```solidity
File: contracts/Tokenomics.sol

716                  (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).
717:                     getUnitIdsOfService(IServiceRegistry.UnitType(unitType), serviceIds[i]);

```
*GitHub*: [716](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L716-L717)

```solidity
File: contracts/Treasury.sol

228:         if (IToken(token).allowance(account, address(this)) < tokenAmount) {

229:             revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);

242:         IOLAS(olas).mint(msg.sender, olasMintAmount);

298:         ITokenomics(tokenomics).trackServiceDonations(msg.sender, serviceIds, amounts, msg.value);

416:             IOLAS(olas).mint(account, accountTopUps);

```
*GitHub*: [228](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L228-L228), [229](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L229-L229), [242](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L242-L242), [298](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L298-L298), [416](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L416-L416)

</details>





### [N&#x2011;09] Consider using named returns
Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

*There are 63 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/GovernorOLAS.sol

33       function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
34           returns (ProposalState)
35:      {

45       function propose(
46           address[] memory targets,
47           uint256[] memory values,
48           bytes[] memory calldatas,
49           string memory description
50       ) public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)
51:      {

57       function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)
58:      {

85       function _cancel(
86           address[] memory targets,
87           uint256[] memory values,
88           bytes[] memory calldatas,
89           bytes32 descriptionHash
90       ) internal override(Governor, GovernorTimelockControl) returns (uint256)
91:      {

97       function _executor() internal view override(Governor, GovernorTimelockControl) returns (address)
98:      {

105      function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
106          returns (bool)
107:     {

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L33-L35), [45](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L45-L51), [57](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L57-L58), [85](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L85-L91), [97](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L97-L98), [105](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L105-L107)

```solidity
File: contracts/OLAS.sol

91:      function inflationControl(uint256 amount) public view returns (bool) {

128:     function decreaseAllowance(address spender, uint256 amount) external returns (bool) {

145:     function increaseAllowance(address spender, uint256 amount) external returns (bool) {

```
*GitHub*: [91](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L91-L91), [128](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L128-L128), [145](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L145-L145)

```solidity
File: contracts/bridges/HomeMediator.sol

6:       function messageSender() external view returns (address);

```
*GitHub*: [6](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L6-L6)

```solidity
File: contracts/interfaces/IERC20.sol

9:       function balanceOf(address account) external view returns (uint256);

13:      function totalSupply() external view returns (uint256);

19:      function allowance(address owner, address spender) external view returns (uint256);

25:      function approve(address spender, uint256 amount) external returns (bool);

31:      function transfer(address to, uint256 amount) external returns (bool);

38:      function transferFrom(address from, address to, uint256 amount) external returns (bool);

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IERC20.sol#L9-L9), [13](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IERC20.sol#L13-L13), [19](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IERC20.sol#L19-L19), [25](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IERC20.sol#L25-L25), [31](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IERC20.sol#L31-L31), [38](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IERC20.sol#L38-L38)

```solidity
File: contracts/multisigs/GuardCM.sol

7:       function state(uint256 proposalId) external returns (ProposalState);

```
*GitHub*: [7](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L7-L7)

```solidity
File: contracts/veOLAS.sol

164:     function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory) {

633:     function getVotes(address account) public view override returns (uint256) {

719:     function totalSupply() public view override returns (uint256) {

738:     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

745:     function totalSupplyLocked() public view returns (uint256) {

752:     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761:     function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

767:     function transfer(address to, uint256 amount) external virtual override returns (bool) {

772:     function approve(address spender, uint256 amount) external virtual override returns (bool) {

777:     function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {

782      function allowance(address owner, address spender) external view virtual override returns (uint256)
783:     {

788      function delegates(address account) external view virtual override returns (address)
789:     {

```
*GitHub*: [164](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L164-L164), [633](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L633-L633), [719](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L719-L719), [738](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L738-L738), [745](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L745-L745), [752](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L752-L752), [761](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L761-L761), [767](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L767-L767), [772](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L772-L772), [777](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L777-L777), [782](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L782-L783), [788](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L788-L789)

```solidity
File: contracts/wveOLAS.sol

98:      function supportsInterface(bytes4 interfaceId) external view returns (bool);

101:     function allowance(address owner, address spender) external view returns (uint256);

104:     function delegates(address account) external view returns (address);

292:     function supportsInterface(bytes4 interfaceId) external view returns (bool) {

297:     function transfer(address, uint256) external returns (bool) {

302:     function approve(address, uint256) external returns (bool) {

307:     function transferFrom(address, address, uint256) external returns (bool) {

312:     function allowance(address, address) external view returns (uint256) {

317:     function delegates(address) external view returns (address) {

```
*GitHub*: [98](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L98-L98), [101](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L101-L101), [104](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L104-L104), [292](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L292-L292), [297](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L297-L297), [302](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L302-L302), [307](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L307-L307), [312](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L312-L312), [317](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L317-L317)

```solidity
File: contracts/GenericRegistry.sol

72:      function exists(uint256 unitId) external view virtual returns (bool) {

129:     function _getUnitHash(uint256 unitId) internal view virtual returns (bytes32);

135:     function tokenURI(uint256 unitId) public view virtual override returns (string memory) {

```
*GitHub*: [72](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L72-L72), [129](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L129-L129), [135](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L135-L135)

```solidity
File: contracts/UnitRegistry.sol

270:     function _getUnitHash(uint256 unitId) internal view override returns (bytes32) {

```
*GitHub*: [270](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L270-L270)

```solidity
File: contracts/interfaces/IRegistry.sol

16       function create(
17           address unitOwner,
18           bytes32 unitHash,
19           uint32[] memory dependencies
20:      ) external returns (uint256);

48:      function totalSupply() external view returns (uint256);

```
*GitHub*: [16](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/interfaces/IRegistry.sol#L16-L20), [48](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/interfaces/IRegistry.sol#L48-L48)

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

8:       function getOwners() external view returns (address[] memory);

12:      function getThreshold() external view returns (uint256);

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L8-L8), [12](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L12-L12)

```solidity
File: contracts/interfaces/IOLAS.sol

12:      function timeLaunch() external view returns (uint256);

```
*GitHub*: [12](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IOLAS.sol#L12-L12)

```solidity
File: contracts/interfaces/IServiceRegistry.sol

14:      function exists(uint256 serviceId) external view returns (bool);

```
*GitHub*: [14](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L14)

```solidity
File: contracts/interfaces/IToken.sol

9:       function balanceOf(address account) external view returns (uint256);

14:      function ownerOf(uint256 tokenId) external view returns (address);

18:      function totalSupply() external view returns (uint256);

24:      function transfer(address to, uint256 amount) external returns (bool);

30:      function allowance(address owner, address spender) external view returns (uint256);

36:      function approve(address spender, uint256 amount) external returns (bool);

43:      function transferFrom(address from, address to, uint256 amount) external returns (bool);

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IToken.sol#L9-L9), [14](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IToken.sol#L14-L14), [18](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IToken.sol#L18-L18), [24](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IToken.sol#L24-L24), [30](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IToken.sol#L30-L30), [36](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IToken.sol#L36-L36), [43](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IToken.sol#L43-L43)

```solidity
File: contracts/interfaces/ITokenomics.sol

8:       function effectiveBond() external pure returns (uint256);

11:      function checkpoint() external returns (bool);

30:      function reserveAmountForBondProgram(uint256 amount) external returns(bool);

51:      function serviceRegistry() external view returns (address);

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/ITokenomics.sol#L8-L8), [11](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/ITokenomics.sol#L11-L11), [30](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/ITokenomics.sol#L30-L30), [51](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/ITokenomics.sol#L51-L51)

```solidity
File: contracts/interfaces/IUniswapV2Pair.sol

6:       function totalSupply() external view returns (uint);

7:       function token0() external view returns (address);

8:       function token1() external view returns (address);

```
*GitHub*: [6](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L8-L8)

```solidity
File: contracts/interfaces/IVotingEscrow.sol

8:       function getVotes(address account) external view returns (uint256);

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IVotingEscrow.sol#L8-L8)

</details>





### [N&#x2011;10] Constants in comparisons should appear on the left side
Doing so will prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html) where the first '!', '>', '<', or '=' at the beginning of the operator is missing.

*There are 10 instances of this issue:*

```solidity
File: contracts/GenericBondCalculator.sol

82:              if (token0 == olas || token1 == olas) {

82:              if (token0 == olas || token1 == olas) {

84:                  if (token0 == olas) {

```
*GitHub*: [82](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L82-L82), [82](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L82-L82), [84](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L84-L84)

```solidity
File: contracts/Tokenomics.sol

296:         if (uint32(_epochLen) < MIN_EPOCH_LENGTH) {

301:         if (uint32(_epochLen) > ONE_YEAR) {

511:         if (uint72(_devsPerCapital) > MIN_PARAM_VALUE) {

519:         if (uint72(_codePerDev) > MIN_PARAM_VALUE) {

536:         if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) {

536:         if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) {

902:         if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) {

```
*GitHub*: [296](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L296-L296), [301](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L301-L301), [511](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L511-L511), [519](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L519-L519), [536](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L536-L536), [536](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L536-L536), [902](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L902-L902)



### [N&#x2011;11] Contract timekeeping will break earlier than the Ethereum network itself will stop working
When a timestamp is downcast from `uint256` to `uint32`, the value will wrap in the year 2106, and the contracts will break. Other downcasts will have different endpoints. Consider whether your contract is intended to live past the size of the type being used.

*There is one instance of this issue:*

```solidity
File: contracts/Tokenomics.sol

325:         uint256 zeroYearSecondsLeft = uint32(_timeLaunch + ONE_YEAR - block.timestamp);

```
*GitHub*: [325](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L325-L325)



### [N&#x2011;12] Contracts/libraries/interfaces should each be defined in separate files
This helps to make tracking changes across commits easier, among other reasons. They can be combined into a single file later by importing multiple contracts, and using that file as the common import.

*There are 12 instances of this issue:*

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

/// @audit FxGovernorTunnel,IFxMessageProcessor
5:   interface IFxMessageProcessor {

/// @audit FxGovernorTunnel,IFxMessageProcessor
46:  contract FxGovernorTunnel is IFxMessageProcessor {

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L5-L5), [46](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L46-L46)

```solidity
File: contracts/bridges/HomeMediator.sol

/// @audit IAMB,HomeMediator
5:   interface IAMB {

/// @audit IAMB,HomeMediator
46:  contract HomeMediator {

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L5-L5), [46](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L46-L46)

```solidity
File: contracts/multisigs/GuardCM.sol

/// @audit GuardCM,IGovernor
6:   interface IGovernor {

/// @audit GuardCM,IGovernor
88:  contract GuardCM {

```
*GitHub*: [6](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L6-L6), [88](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L88-L88)

```solidity
File: contracts/wveOLAS.sol

/// @audit IVEOLAS,wveOLAS
13   interface IVEOLAS {
14       /// @dev Gets the total number of supply points.
15:      /// @return numPoints Number of supply points.

/// @audit IVEOLAS,wveOLAS
130  contract wveOLAS {
131:     // veOLAS address

```
*GitHub*: [13](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L13-L15), [130](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L130-L131)

```solidity
File: contracts/multisigs/GnosisSafeMultisig.sol

/// @audit IGnosisSafeProxyFactory,GnosisSafeMultisig
5    interface IGnosisSafeProxyFactory {
6        /// @dev Allows to create new proxy contact and execute a message call to the new proxy within one transaction.
7        /// @param _singleton Address of singleton contract.
8        /// @param initializer Payload for message call sent to new proxy contract.
9:       /// @param saltNonce Nonce that will be used to generate the salt to calculate the address of the new proxy contract.

/// @audit IGnosisSafeProxyFactory,GnosisSafeMultisig
24   contract GnosisSafeMultisig {
25:      // Selector of the Gnosis Safe setup function

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L5-L9), [24](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L24-L25)

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

/// @audit GnosisSafeSameAddressMultisig,IGnosisSafe
5    interface IGnosisSafe {
6        /// @dev Gets set of owners.
7:       /// @return Set of Safe owners.

/// @audit GnosisSafeSameAddressMultisig,IGnosisSafe
50   contract GnosisSafeSameAddressMultisig {
51       // Default data size to be parsed as an address of a Gnosis Safe multisig proxy address
52:      // This exact size suggests that all the changes to the multisig have been performed and only validation is needed

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L5-L7), [50](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L50-L52)



### [N&#x2011;13] Function state mutability can be restricted to `view`
Note that when overriding, mutability is allowed to be changed to be [more](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) strict than the parent function's mutability

*There are 6 instances of this issue:*

```solidity
File: contracts/multisigs/GuardCM.sol

189:     function _verifyData(address target, bytes memory data, uint256 chainId) internal {

```
*GitHub*: [189](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L189-L189)

```solidity
File: contracts/wveOLAS.sol

297:     function transfer(address, uint256) external returns (bool) {

302:     function approve(address, uint256) external returns (bool) {

307:     function transferFrom(address, address, uint256) external returns (bool) {

322      function delegate(address) external
323:     {

328      function delegateBySig(address, uint256, uint256, uint8, bytes32, bytes32) external
329:     {

```
*GitHub*: [297](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L297-L297), [302](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L302-L302), [307](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L307-L307), [322](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L322-L323), [328](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L328-L329)



### [N&#x2011;14] Inconsistent spacing in comments
Some lines use `// x` and some use `//x`. The instances below point out the usages that don't follow the majority, within each file

*There are 2 instances of this issue:*

```solidity
File: contracts/Tokenomics.sol

873:      ///$result == true && (block.timestamp - timeLaunch) / ONE_YEAR == old(currentYear)

```
*GitHub*: [873](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L873)

```solidity
File: contracts/Treasury.sol

464:      ///if_succeeds {:msg "slashed funds check"} IServiceRegistry(ITokenomics(tokenomics).serviceRegistry()).slashedFunds() >= minAcceptedETH

```
*GitHub*: [464](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L464)



### [N&#x2011;15] Interfaces should be defined in separate files from their usage
The interfaces below should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project

*There is one instance of this issue:*

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

5:   interface IFxMessageProcessor {

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L5-L5)



### [N&#x2011;16] Named imports of parent contracts are missing


*There are 17 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/OLAS.sol

/// @audit ERC20
17:  contract OLAS is ERC20 {

```
*GitHub*: [17](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L17-L17)

```solidity
File: contracts/Timelock.sol

/// @audit TimelockController
9:   contract Timelock is TimelockController {

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/Timelock.sol#L9-L9)

```solidity
File: contracts/veOLAS.sol

/// @audit IErrors
/// @audit IVotes
/// @audit IERC20
/// @audit IERC165
86:  contract veOLAS is IErrors, IVotes, IERC20, IERC165 {

```
*GitHub*: [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L86-L86), [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L86-L86), [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L86-L86), [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L86-L86)

```solidity
File: contracts/AgentRegistry.sol

/// @audit UnitRegistry
9:   contract AgentRegistry is UnitRegistry {

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L9-L9)

```solidity
File: contracts/ComponentRegistry.sol

/// @audit UnitRegistry
8:   contract ComponentRegistry is UnitRegistry {

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L8-L8)

```solidity
File: contracts/GenericManager.sol

/// @audit IErrorsRegistries
8:   abstract contract GenericManager is IErrorsRegistries {

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L8-L8)

```solidity
File: contracts/GenericRegistry.sol

/// @audit IErrorsRegistries
/// @audit ERC721
9:   abstract contract GenericRegistry is IErrorsRegistries, ERC721 {

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L9-L9), [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L9-L9)

```solidity
File: contracts/RegistriesManager.sol

/// @audit GenericManager
9:   contract RegistriesManager is GenericManager {

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/RegistriesManager.sol#L9-L9)

```solidity
File: contracts/UnitRegistry.sol

/// @audit GenericRegistry
8:   abstract contract UnitRegistry is GenericRegistry {

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L8-L8)

```solidity
File: contracts/Dispenser.sol

/// @audit IErrorsTokenomics
11:  contract Dispenser is IErrorsTokenomics {

```
*GitHub*: [11](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L11-L11)

```solidity
File: contracts/Tokenomics.sol

/// @audit TokenomicsConstants
/// @audit IErrorsTokenomics
118: contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {

```
*GitHub*: [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L118-L118), [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L118-L118)

```solidity
File: contracts/Treasury.sol

/// @audit IErrorsTokenomics
39:  contract Treasury is IErrorsTokenomics {

```
*GitHub*: [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L39-L39)

</details>





### [N&#x2011;17] NatSpec: Event `@param` tag is missing


*There are 149 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/OLAS.sol

/// @audit Missing '@param minter'
13   
14   /// @title OLAS - Smart contract for the OLAS token.
15   /// @author AL
16   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
17   contract OLAS is ERC20 {
18:      event MinterUpdated(address indexed minter);

/// @audit Missing '@param owner'
14   /// @title OLAS - Smart contract for the OLAS token.
15   /// @author AL
16   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
17   contract OLAS is ERC20 {
18       event MinterUpdated(address indexed minter);
19:      event OwnerUpdated(address indexed owner);

```
*GitHub*: [13](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L13-L18), [14](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L14-L19)

```solidity
File: contracts/bridges/BridgedERC20.sol

/// @audit Missing '@param owner'
14   
15   /// @title BridgedERC20 - Smart contract for bridged ERC20 token
16   /// @dev Bridged token contract is owned by the bridge mediator contract, and thus the token representation from
17   ///      another chain must be minted and burned solely by the bridge mediator contract.
18   contract BridgedERC20 is ERC20 {
19:      event OwnerUpdated(address indexed owner);

```
*GitHub*: [14](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/BridgedERC20.sol#L14-L19)

```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

/// @audit Missing '@param childToken'
/// @audit Missing '@param rootToken'
/// @audit Missing '@param from'
/// @audit Missing '@param to'
/// @audit Missing '@param amount'
20   /// @title FxERC20ChildTunnel - Smart contract for the L2 token management part
21   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
22   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
23   /// @author Mariapia Moscatiello - <mariapia.moscatiello@valory.xyz>
24   contract FxERC20ChildTunnel is FxBaseChildTunnel {
25:      event FxDepositERC20(address indexed childToken, address indexed rootToken, address from, address indexed to, uint256 amount);

/// @audit Missing '@param rootToken'
/// @audit Missing '@param childToken'
/// @audit Missing '@param from'
/// @audit Missing '@param to'
/// @audit Missing '@param amount'
21   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
22   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
23   /// @author Mariapia Moscatiello - <mariapia.moscatiello@valory.xyz>
24   contract FxERC20ChildTunnel is FxBaseChildTunnel {
25       event FxDepositERC20(address indexed childToken, address indexed rootToken, address from, address indexed to, uint256 amount);
26:      event FxWithdrawERC20(address indexed rootToken, address indexed childToken, address from, address indexed to, uint256 amount);

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L20-L25), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L21-L26)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

/// @audit Missing '@param childToken'
/// @audit Missing '@param rootToken'
/// @audit Missing '@param from'
/// @audit Missing '@param to'
/// @audit Missing '@param amount'
20   /// @title FxERC20RootTunnel - Smart contract for the L1 token management part
21   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
22   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
23   /// @author Mariapia Moscatiello - <mariapia.moscatiello@valory.xyz>
24   contract FxERC20RootTunnel is FxBaseRootTunnel {
25:      event FxDepositERC20(address indexed childToken, address indexed rootToken, address from, address indexed to, uint256 amount);

/// @audit Missing '@param rootToken'
/// @audit Missing '@param childToken'
/// @audit Missing '@param from'
/// @audit Missing '@param to'
/// @audit Missing '@param amount'
21   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
22   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
23   /// @author Mariapia Moscatiello - <mariapia.moscatiello@valory.xyz>
24   contract FxERC20RootTunnel is FxBaseRootTunnel {
25       event FxDepositERC20(address indexed childToken, address indexed rootToken, address from, address indexed to, uint256 amount);
26:      event FxWithdrawERC20(address indexed rootToken, address indexed childToken, address from, address indexed to, uint256 amount);

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L20-L25), [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L20-L25), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L21-L26), [21](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L21-L26)

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

/// @audit Missing '@param sender'
/// @audit Missing '@param value'
42   
43   /// @title FxGovernorTunnel - Smart contract for the governor child tunnel bridge implementation
44   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
45   /// @author AL
46   contract FxGovernorTunnel is IFxMessageProcessor {
47:      event FundsReceived(address indexed sender, uint256 value);

/// @audit Missing '@param rootMessageSender'
43   /// @title FxGovernorTunnel - Smart contract for the governor child tunnel bridge implementation
44   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
45   /// @author AL
46   contract FxGovernorTunnel is IFxMessageProcessor {
47       event FundsReceived(address indexed sender, uint256 value);
48:      event RootGovernorUpdated(address indexed rootMessageSender);

/// @audit Missing '@param stateId'
/// @audit Missing '@param rootMessageSender'
/// @audit Missing '@param data'
44   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
45   /// @author AL
46   contract FxGovernorTunnel is IFxMessageProcessor {
47       event FundsReceived(address indexed sender, uint256 value);
48       event RootGovernorUpdated(address indexed rootMessageSender);
49:      event MessageReceived(uint256 indexed stateId, address indexed rootMessageSender, bytes data);

```
*GitHub*: [42](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L42-L47), [42](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L42-L47), [43](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L43-L48), [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L44-L49), [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L44-L49), [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L44-L49)

```solidity
File: contracts/bridges/HomeMediator.sol

/// @audit Missing '@param sender'
/// @audit Missing '@param value'
42   
43   /// @title HomeMediator - Smart contract for the governor home (gnosis chain) bridge implementation
44   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
45   /// @author AL
46   contract HomeMediator {
47:      event FundsReceived(address indexed sender, uint256 value);

/// @audit Missing '@param foreignMessageSender'
43   /// @title HomeMediator - Smart contract for the governor home (gnosis chain) bridge implementation
44   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
45   /// @author AL
46   contract HomeMediator {
47       event FundsReceived(address indexed sender, uint256 value);
48:      event ForeignGovernorUpdated(address indexed foreignMessageSender);

/// @audit Missing '@param foreignMessageSender'
/// @audit Missing '@param data'
44   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
45   /// @author AL
46   contract HomeMediator {
47       event FundsReceived(address indexed sender, uint256 value);
48       event ForeignGovernorUpdated(address indexed foreignMessageSender);
49:      event MessageReceived(address indexed foreignMessageSender, bytes data);

```
*GitHub*: [42](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L42-L47), [42](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L42-L47), [43](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L43-L48), [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L44-L49), [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L44-L49)

```solidity
File: contracts/multisigs/GuardCM.sol

/// @audit Missing '@param governor'
84   
85   /// @title GuardCM - Smart contract for Gnosis Safe community multisig (CM) guard
86   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
87   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
88   contract GuardCM {
89:      event GovernorUpdated(address indexed governor);

/// @audit Missing '@param targets'
/// @audit Missing '@param selectors'
/// @audit Missing '@param chainIds'
/// @audit Missing '@param statuses'
85   /// @title GuardCM - Smart contract for Gnosis Safe community multisig (CM) guard
86   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
87   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
88   contract GuardCM {
89       event GovernorUpdated(address indexed governor);
90:      event SetTargetSelectors(address[] indexed targets, bytes4[] indexed selectors, uint256[] chainIds, bool[] statuses);

/// @audit Missing '@param bridgeMediatorL1s'
/// @audit Missing '@param bridgeMediatorL2s'
/// @audit Missing '@param chainIds'
86   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
87   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
88   contract GuardCM {
89       event GovernorUpdated(address indexed governor);
90       event SetTargetSelectors(address[] indexed targets, bytes4[] indexed selectors, uint256[] chainIds, bool[] statuses);
91:      event SetBridgeMediators(address[] indexed bridgeMediatorL1s, address[] indexed bridgeMediatorL2s, uint256[] chainIds);

/// @audit Missing '@param proposalId'
87   /// @author Andrey Lebedev - <andrey.lebedev@valory.xyz>
88   contract GuardCM {
89       event GovernorUpdated(address indexed governor);
90       event SetTargetSelectors(address[] indexed targets, bytes4[] indexed selectors, uint256[] chainIds, bool[] statuses);
91       event SetBridgeMediators(address[] indexed bridgeMediatorL1s, address[] indexed bridgeMediatorL2s, uint256[] chainIds);
92:      event GovernorCheckProposalIdChanged(uint256 indexed proposalId);

/// @audit Missing '@param account'
88   contract GuardCM {
89       event GovernorUpdated(address indexed governor);
90       event SetTargetSelectors(address[] indexed targets, bytes4[] indexed selectors, uint256[] chainIds, bool[] statuses);
91       event SetBridgeMediators(address[] indexed bridgeMediatorL1s, address[] indexed bridgeMediatorL2s, uint256[] chainIds);
92       event GovernorCheckProposalIdChanged(uint256 indexed proposalId);
93:      event GuardPaused(address indexed account);

```
*GitHub*: [84](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L84-L89), [85](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L85-L90), [85](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L85-L90), [85](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L85-L90), [85](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L85-L90), [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L86-L91), [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L86-L91), [86](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L86-L91), [87](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L87-L92), [88](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L88-L93)

```solidity
File: contracts/veOLAS.sol

/// @audit Missing '@param account'
/// @audit Missing '@param amount'
/// @audit Missing '@param locktime'
/// @audit Missing '@param depositType'
/// @audit Missing '@param ts'
89           CREATE_LOCK_TYPE,
90           INCREASE_LOCK_AMOUNT,
91           INCREASE_UNLOCK_TIME
92       }
93   
94:      event Deposit(address indexed account, uint256 amount, uint256 locktime, DepositType depositType, uint256 ts);

/// @audit Missing '@param account'
/// @audit Missing '@param amount'
/// @audit Missing '@param ts'
90           INCREASE_LOCK_AMOUNT,
91           INCREASE_UNLOCK_TIME
92       }
93   
94       event Deposit(address indexed account, uint256 amount, uint256 locktime, DepositType depositType, uint256 ts);
95:      event Withdraw(address indexed account, uint256 amount, uint256 ts);

/// @audit Missing '@param previousSupply'
/// @audit Missing '@param currentSupply'
91           INCREASE_UNLOCK_TIME
92       }
93   
94       event Deposit(address indexed account, uint256 amount, uint256 locktime, DepositType depositType, uint256 ts);
95       event Withdraw(address indexed account, uint256 amount, uint256 ts);
96:      event Supply(uint256 previousSupply, uint256 currentSupply);

```
*GitHub*: [89](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L89-L94), [89](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L89-L94), [89](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L89-L94), [89](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L89-L94), [89](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L89-L94), [90](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L90-L95), [90](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L90-L95), [90](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L90-L95), [91](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L91-L96), [91](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L91-L96)

```solidity
File: contracts/GenericManager.sol

/// @audit Missing '@param owner'
4    import "./interfaces/IErrorsRegistries.sol";
5    
6    /// @title Generic Manager - Smart contract for generic registry manager template
7    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
8    abstract contract GenericManager is IErrorsRegistries {
9:       event OwnerUpdated(address indexed owner);

/// @audit Missing '@param owner'
5    
6    /// @title Generic Manager - Smart contract for generic registry manager template
7    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
8    abstract contract GenericManager is IErrorsRegistries {
9        event OwnerUpdated(address indexed owner);
10:      event Pause(address indexed owner);

/// @audit Missing '@param owner'
6    /// @title Generic Manager - Smart contract for generic registry manager template
7    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
8    abstract contract GenericManager is IErrorsRegistries {
9        event OwnerUpdated(address indexed owner);
10       event Pause(address indexed owner);
11:      event Unpause(address indexed owner);

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L4-L9), [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L5-L10), [6](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L6-L11)

```solidity
File: contracts/GenericRegistry.sol

/// @audit Missing '@param owner'
5    import "./interfaces/IErrorsRegistries.sol";
6    
7    /// @title Generic Registry - Smart contract for generic registry template
8    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
9    abstract contract GenericRegistry is IErrorsRegistries, ERC721 {
10:      event OwnerUpdated(address indexed owner);

/// @audit Missing '@param manager'
6    
7    /// @title Generic Registry - Smart contract for generic registry template
8    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
9    abstract contract GenericRegistry is IErrorsRegistries, ERC721 {
10       event OwnerUpdated(address indexed owner);
11:      event ManagerUpdated(address indexed manager);

/// @audit Missing '@param baseURI'
7    /// @title Generic Registry - Smart contract for generic registry template
8    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
9    abstract contract GenericRegistry is IErrorsRegistries, ERC721 {
10       event OwnerUpdated(address indexed owner);
11       event ManagerUpdated(address indexed manager);
12:      event BaseURIChanged(string baseURI);

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L5-L10), [6](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L6-L11), [7](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L7-L12)

```solidity
File: contracts/UnitRegistry.sol

/// @audit Missing '@param unitId'
/// @audit Missing '@param uType'
/// @audit Missing '@param unitHash'
4    import "./GenericRegistry.sol";
5    
6    /// @title Unit Registry - Smart contract for registering generalized units / units
7    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
8    abstract contract UnitRegistry is GenericRegistry {
9:       event CreateUnit(uint256 unitId, UnitType uType, bytes32 unitHash);

/// @audit Missing '@param unitId'
/// @audit Missing '@param uType'
/// @audit Missing '@param unitHash'
5    
6    /// @title Unit Registry - Smart contract for registering generalized units / units
7    /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
8    abstract contract UnitRegistry is GenericRegistry {
9        event CreateUnit(uint256 unitId, UnitType uType, bytes32 unitHash);
10:      event UpdateUnitHash(uint256 unitId, UnitType uType, bytes32 unitHash);

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L4-L9), [4](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L4-L9), [4](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L4-L9), [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L5-L10), [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L5-L10), [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L5-L10)

```solidity
File: contracts/Depository.sol

/// @audit Missing '@param owner'
58   
59   /// @title Bond Depository - Smart contract for OLAS Bond Depository
60   /// @author AL
61   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
62   contract Depository is IErrorsTokenomics {
63:      event OwnerUpdated(address indexed owner);

/// @audit Missing '@param tokenomics'
59   /// @title Bond Depository - Smart contract for OLAS Bond Depository
60   /// @author AL
61   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
62   contract Depository is IErrorsTokenomics {
63       event OwnerUpdated(address indexed owner);
64:      event TokenomicsUpdated(address indexed tokenomics);

/// @audit Missing '@param treasury'
60   /// @author AL
61   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
62   contract Depository is IErrorsTokenomics {
63       event OwnerUpdated(address indexed owner);
64       event TokenomicsUpdated(address indexed tokenomics);
65:      event TreasuryUpdated(address indexed treasury);

/// @audit Missing '@param bondCalculator'
61   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
62   contract Depository is IErrorsTokenomics {
63       event OwnerUpdated(address indexed owner);
64       event TokenomicsUpdated(address indexed tokenomics);
65       event TreasuryUpdated(address indexed treasury);
66:      event BondCalculatorUpdated(address indexed bondCalculator);

/// @audit Missing '@param token'
/// @audit Missing '@param productId'
/// @audit Missing '@param owner'
/// @audit Missing '@param bondId'
62   contract Depository is IErrorsTokenomics {
63       event OwnerUpdated(address indexed owner);
64       event TokenomicsUpdated(address indexed tokenomics);
65       event TreasuryUpdated(address indexed treasury);
66       event BondCalculatorUpdated(address indexed bondCalculator);
67:      event CreateBond(address indexed token, uint256 indexed productId, address indexed owner, uint256 bondId,

/// @audit Missing '@param amountOLAS'
/// @audit Missing '@param tokenAmount'
/// @audit Missing '@param maturity'
63       event OwnerUpdated(address indexed owner);
64       event TokenomicsUpdated(address indexed tokenomics);
65       event TreasuryUpdated(address indexed treasury);
66       event BondCalculatorUpdated(address indexed bondCalculator);
67       event CreateBond(address indexed token, uint256 indexed productId, address indexed owner, uint256 bondId,
68:          uint256 amountOLAS, uint256 tokenAmount, uint256 maturity);

/// @audit Missing '@param productId'
/// @audit Missing '@param owner'
/// @audit Missing '@param bondId'
64       event TokenomicsUpdated(address indexed tokenomics);
65       event TreasuryUpdated(address indexed treasury);
66       event BondCalculatorUpdated(address indexed bondCalculator);
67       event CreateBond(address indexed token, uint256 indexed productId, address indexed owner, uint256 bondId,
68           uint256 amountOLAS, uint256 tokenAmount, uint256 maturity);
69:      event RedeemBond(uint256 indexed productId, address indexed owner, uint256 bondId);

/// @audit Missing '@param token'
/// @audit Missing '@param productId'
/// @audit Missing '@param supply'
/// @audit Missing '@param priceLP'
65       event TreasuryUpdated(address indexed treasury);
66       event BondCalculatorUpdated(address indexed bondCalculator);
67       event CreateBond(address indexed token, uint256 indexed productId, address indexed owner, uint256 bondId,
68           uint256 amountOLAS, uint256 tokenAmount, uint256 maturity);
69       event RedeemBond(uint256 indexed productId, address indexed owner, uint256 bondId);
70:      event CreateProduct(address indexed token, uint256 indexed productId, uint256 supply, uint256 priceLP,

/// @audit Missing '@param vesting'
66       event BondCalculatorUpdated(address indexed bondCalculator);
67       event CreateBond(address indexed token, uint256 indexed productId, address indexed owner, uint256 bondId,
68           uint256 amountOLAS, uint256 tokenAmount, uint256 maturity);
69       event RedeemBond(uint256 indexed productId, address indexed owner, uint256 bondId);
70       event CreateProduct(address indexed token, uint256 indexed productId, uint256 supply, uint256 priceLP,
71:          uint256 vesting);

/// @audit Missing '@param token'
/// @audit Missing '@param productId'
/// @audit Missing '@param supply'
67       event CreateBond(address indexed token, uint256 indexed productId, address indexed owner, uint256 bondId,
68           uint256 amountOLAS, uint256 tokenAmount, uint256 maturity);
69       event RedeemBond(uint256 indexed productId, address indexed owner, uint256 bondId);
70       event CreateProduct(address indexed token, uint256 indexed productId, uint256 supply, uint256 priceLP,
71           uint256 vesting);
72:      event CloseProduct(address indexed token, uint256 indexed productId, uint256 supply);

```
*GitHub*: [58](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L58-L63), [59](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L59-L64), [60](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L60-L65), [61](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L61-L66), [62](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L62-L67), [62](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L62-L67), [62](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L62-L67), [62](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L62-L67), [63](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L63-L68), [63](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L63-L68), [63](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L63-L68), [64](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L64-L69), [64](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L64-L69), [64](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L64-L69), [65](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L65-L70), [65](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L65-L70), [65](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L65-L70), [65](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L65-L70), [66](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L66-L71), [67](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L67-L72), [67](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L67-L72), [67](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L67-L72)

```solidity
File: contracts/Dispenser.sol

/// @audit Missing '@param owner'
7    
8    /// @title Dispenser - Smart contract for distributing incentives
9    /// @author AL
10   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
11   contract Dispenser is IErrorsTokenomics {
12:      event OwnerUpdated(address indexed owner);

/// @audit Missing '@param tokenomics'
8    /// @title Dispenser - Smart contract for distributing incentives
9    /// @author AL
10   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
11   contract Dispenser is IErrorsTokenomics {
12       event OwnerUpdated(address indexed owner);
13:      event TokenomicsUpdated(address indexed tokenomics);

/// @audit Missing '@param treasury'
9    /// @author AL
10   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
11   contract Dispenser is IErrorsTokenomics {
12       event OwnerUpdated(address indexed owner);
13       event TokenomicsUpdated(address indexed tokenomics);
14:      event TreasuryUpdated(address indexed treasury);

/// @audit Missing '@param owner'
/// @audit Missing '@param reward'
/// @audit Missing '@param topUp'
10   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
11   contract Dispenser is IErrorsTokenomics {
12       event OwnerUpdated(address indexed owner);
13       event TokenomicsUpdated(address indexed tokenomics);
14       event TreasuryUpdated(address indexed treasury);
15:      event IncentivesClaimed(address indexed owner, uint256 reward, uint256 topUp);

```
*GitHub*: [7](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L7-L12), [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L8-L13), [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L9-L14), [10](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L10-L15), [10](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L10-L15), [10](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L10-L15)

```solidity
File: contracts/DonatorBlacklist.sol

/// @audit Missing '@param owner'
16   
17   /// @title DonatorBlacklist - Smart contract for donator address blacklisting
18   /// @author AL
19   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
20   contract DonatorBlacklist {
21:      event OwnerUpdated(address indexed owner);

/// @audit Missing '@param account'
/// @audit Missing '@param status'
17   /// @title DonatorBlacklist - Smart contract for donator address blacklisting
18   /// @author AL
19   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
20   contract DonatorBlacklist {
21       event OwnerUpdated(address indexed owner);
22:      event DonatorBlacklistStatus(address indexed account, bool status);

```
*GitHub*: [16](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L16-L21), [17](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L17-L22), [17](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L17-L22)

```solidity
File: contracts/Tokenomics.sol

/// @audit Missing '@param owner'
114  
115  /// @title Tokenomics - Smart contract for tokenomics logic with incentives for unit owners and discount factor regulations for bonds.
116  /// @author AL
117  /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
118  contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
119:     event OwnerUpdated(address indexed owner);

/// @audit Missing '@param treasury'
115  /// @title Tokenomics - Smart contract for tokenomics logic with incentives for unit owners and discount factor regulations for bonds.
116  /// @author AL
117  /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
118  contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
119      event OwnerUpdated(address indexed owner);
120:     event TreasuryUpdated(address indexed treasury);

/// @audit Missing '@param depository'
116  /// @author AL
117  /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
118  contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
119      event OwnerUpdated(address indexed owner);
120      event TreasuryUpdated(address indexed treasury);
121:     event DepositoryUpdated(address indexed depository);

/// @audit Missing '@param dispenser'
117  /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
118  contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
119      event OwnerUpdated(address indexed owner);
120      event TreasuryUpdated(address indexed treasury);
121      event DepositoryUpdated(address indexed depository);
122:     event DispenserUpdated(address indexed dispenser);

/// @audit Missing '@param epochLen'
118  contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
119      event OwnerUpdated(address indexed owner);
120      event TreasuryUpdated(address indexed treasury);
121      event DepositoryUpdated(address indexed depository);
122      event DispenserUpdated(address indexed dispenser);
123:     event EpochLengthUpdated(uint256 epochLen);

/// @audit Missing '@param effectiveBond'
119      event OwnerUpdated(address indexed owner);
120      event TreasuryUpdated(address indexed treasury);
121      event DepositoryUpdated(address indexed depository);
122      event DispenserUpdated(address indexed dispenser);
123      event EpochLengthUpdated(uint256 epochLen);
124:     event EffectiveBondUpdated(uint256 effectiveBond);

/// @audit Missing '@param idf'
120      event TreasuryUpdated(address indexed treasury);
121      event DepositoryUpdated(address indexed depository);
122      event DispenserUpdated(address indexed dispenser);
123      event EpochLengthUpdated(uint256 epochLen);
124      event EffectiveBondUpdated(uint256 effectiveBond);
125:     event IDFUpdated(uint256 idf);

/// @audit Missing '@param epochNumber'
/// @audit Missing '@param devsPerCapital'
/// @audit Missing '@param codePerDev'
121      event DepositoryUpdated(address indexed depository);
122      event DispenserUpdated(address indexed dispenser);
123      event EpochLengthUpdated(uint256 epochLen);
124      event EffectiveBondUpdated(uint256 effectiveBond);
125      event IDFUpdated(uint256 idf);
126:     event TokenomicsParametersUpdateRequested(uint256 indexed epochNumber, uint256 devsPerCapital, uint256 codePerDev,

/// @audit Missing '@param epsilonRate'
/// @audit Missing '@param epochLen'
/// @audit Missing '@param veOLASThreshold'
122      event DispenserUpdated(address indexed dispenser);
123      event EpochLengthUpdated(uint256 epochLen);
124      event EffectiveBondUpdated(uint256 effectiveBond);
125      event IDFUpdated(uint256 idf);
126      event TokenomicsParametersUpdateRequested(uint256 indexed epochNumber, uint256 devsPerCapital, uint256 codePerDev,
127:         uint256 epsilonRate, uint256 epochLen, uint256 veOLASThreshold);

/// @audit Missing '@param epochNumber'
123      event EpochLengthUpdated(uint256 epochLen);
124      event EffectiveBondUpdated(uint256 effectiveBond);
125      event IDFUpdated(uint256 idf);
126      event TokenomicsParametersUpdateRequested(uint256 indexed epochNumber, uint256 devsPerCapital, uint256 codePerDev,
127          uint256 epsilonRate, uint256 epochLen, uint256 veOLASThreshold);
128:     event TokenomicsParametersUpdated(uint256 indexed epochNumber);

/// @audit Missing '@param epochNumber'
/// @audit Missing '@param rewardComponentFraction'
124      event EffectiveBondUpdated(uint256 effectiveBond);
125      event IDFUpdated(uint256 idf);
126      event TokenomicsParametersUpdateRequested(uint256 indexed epochNumber, uint256 devsPerCapital, uint256 codePerDev,
127          uint256 epsilonRate, uint256 epochLen, uint256 veOLASThreshold);
128      event TokenomicsParametersUpdated(uint256 indexed epochNumber);
129:     event IncentiveFractionsUpdateRequested(uint256 indexed epochNumber, uint256 rewardComponentFraction,

/// @audit Missing '@param rewardAgentFraction'
/// @audit Missing '@param maxBondFraction'
/// @audit Missing '@param topUpComponentFraction'
/// @audit Missing '@param topUpAgentFraction'
125      event IDFUpdated(uint256 idf);
126      event TokenomicsParametersUpdateRequested(uint256 indexed epochNumber, uint256 devsPerCapital, uint256 codePerDev,
127          uint256 epsilonRate, uint256 epochLen, uint256 veOLASThreshold);
128      event TokenomicsParametersUpdated(uint256 indexed epochNumber);
129      event IncentiveFractionsUpdateRequested(uint256 indexed epochNumber, uint256 rewardComponentFraction,
130:         uint256 rewardAgentFraction, uint256 maxBondFraction, uint256 topUpComponentFraction, uint256 topUpAgentFraction);

/// @audit Missing '@param epochNumber'
126      event TokenomicsParametersUpdateRequested(uint256 indexed epochNumber, uint256 devsPerCapital, uint256 codePerDev,
127          uint256 epsilonRate, uint256 epochLen, uint256 veOLASThreshold);
128      event TokenomicsParametersUpdated(uint256 indexed epochNumber);
129      event IncentiveFractionsUpdateRequested(uint256 indexed epochNumber, uint256 rewardComponentFraction,
130          uint256 rewardAgentFraction, uint256 maxBondFraction, uint256 topUpComponentFraction, uint256 topUpAgentFraction);
131:     event IncentiveFractionsUpdated(uint256 indexed epochNumber);

/// @audit Missing '@param componentRegistry'
127          uint256 epsilonRate, uint256 epochLen, uint256 veOLASThreshold);
128      event TokenomicsParametersUpdated(uint256 indexed epochNumber);
129      event IncentiveFractionsUpdateRequested(uint256 indexed epochNumber, uint256 rewardComponentFraction,
130          uint256 rewardAgentFraction, uint256 maxBondFraction, uint256 topUpComponentFraction, uint256 topUpAgentFraction);
131      event IncentiveFractionsUpdated(uint256 indexed epochNumber);
132:     event ComponentRegistryUpdated(address indexed componentRegistry);

/// @audit Missing '@param agentRegistry'
128      event TokenomicsParametersUpdated(uint256 indexed epochNumber);
129      event IncentiveFractionsUpdateRequested(uint256 indexed epochNumber, uint256 rewardComponentFraction,
130          uint256 rewardAgentFraction, uint256 maxBondFraction, uint256 topUpComponentFraction, uint256 topUpAgentFraction);
131      event IncentiveFractionsUpdated(uint256 indexed epochNumber);
132      event ComponentRegistryUpdated(address indexed componentRegistry);
133:     event AgentRegistryUpdated(address indexed agentRegistry);

/// @audit Missing '@param serviceRegistry'
129      event IncentiveFractionsUpdateRequested(uint256 indexed epochNumber, uint256 rewardComponentFraction,
130          uint256 rewardAgentFraction, uint256 maxBondFraction, uint256 topUpComponentFraction, uint256 topUpAgentFraction);
131      event IncentiveFractionsUpdated(uint256 indexed epochNumber);
132      event ComponentRegistryUpdated(address indexed componentRegistry);
133      event AgentRegistryUpdated(address indexed agentRegistry);
134:     event ServiceRegistryUpdated(address indexed serviceRegistry);

/// @audit Missing '@param blacklist'
130          uint256 rewardAgentFraction, uint256 maxBondFraction, uint256 topUpComponentFraction, uint256 topUpAgentFraction);
131      event IncentiveFractionsUpdated(uint256 indexed epochNumber);
132      event ComponentRegistryUpdated(address indexed componentRegistry);
133      event AgentRegistryUpdated(address indexed agentRegistry);
134      event ServiceRegistryUpdated(address indexed serviceRegistry);
135:     event DonatorBlacklistUpdated(address indexed blacklist);

/// @audit Missing '@param epochCounter'
/// @audit Missing '@param treasuryRewards'
/// @audit Missing '@param accountRewards'
/// @audit Missing '@param accountTopUps'
131      event IncentiveFractionsUpdated(uint256 indexed epochNumber);
132      event ComponentRegistryUpdated(address indexed componentRegistry);
133      event AgentRegistryUpdated(address indexed agentRegistry);
134      event ServiceRegistryUpdated(address indexed serviceRegistry);
135      event DonatorBlacklistUpdated(address indexed blacklist);
136:     event EpochSettled(uint256 indexed epochCounter, uint256 treasuryRewards, uint256 accountRewards, uint256 accountTopUps);

/// @audit Missing '@param implementation'
132      event ComponentRegistryUpdated(address indexed componentRegistry);
133      event AgentRegistryUpdated(address indexed agentRegistry);
134      event ServiceRegistryUpdated(address indexed serviceRegistry);
135      event DonatorBlacklistUpdated(address indexed blacklist);
136      event EpochSettled(uint256 indexed epochCounter, uint256 treasuryRewards, uint256 accountRewards, uint256 accountTopUps);
137:     event TokenomicsImplementationUpdated(address indexed implementation);

```
*GitHub*: [114](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L114-L119), [115](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L115-L120), [116](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L116-L121), [117](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L117-L122), [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L118-L123), [119](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L119-L124), [120](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L120-L125), [121](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L121-L126), [121](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L121-L126), [121](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L121-L126), [122](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L122-L127), [122](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L122-L127), [122](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L122-L127), [123](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L123-L128), [124](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L124-L129), [124](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L124-L129), [125](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L125-L130), [125](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L125-L130), [125](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L125-L130), [125](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L125-L130), [126](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L126-L131), [127](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L127-L132), [128](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L128-L133), [129](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L129-L134), [130](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L130-L135), [131](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L131-L136), [131](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L131-L136), [131](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L131-L136), [131](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L131-L136), [132](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L132-L137)

```solidity
File: contracts/Treasury.sol

/// @audit Missing '@param owner'
35   /// @author Aleksandr Kuperman - <aleksandr.kuperman@valory.xyz>
36   /// Invariant does not support a failing call() function while transferring ETH when using the CEI pattern:
37   /// revert TransferFailed(address(0), address(this), to, tokenAmount);
38   /// invariant {:msg "broken conservation law"} address(this).balance == ETHFromServices + ETHOwned;
39   contract Treasury is IErrorsTokenomics {
40:      event OwnerUpdated(address indexed owner);

/// @audit Missing '@param tokenomics'
36   /// Invariant does not support a failing call() function while transferring ETH when using the CEI pattern:
37   /// revert TransferFailed(address(0), address(this), to, tokenAmount);
38   /// invariant {:msg "broken conservation law"} address(this).balance == ETHFromServices + ETHOwned;
39   contract Treasury is IErrorsTokenomics {
40       event OwnerUpdated(address indexed owner);
41:      event TokenomicsUpdated(address indexed tokenomics);

/// @audit Missing '@param depository'
37   /// revert TransferFailed(address(0), address(this), to, tokenAmount);
38   /// invariant {:msg "broken conservation law"} address(this).balance == ETHFromServices + ETHOwned;
39   contract Treasury is IErrorsTokenomics {
40       event OwnerUpdated(address indexed owner);
41       event TokenomicsUpdated(address indexed tokenomics);
42:      event DepositoryUpdated(address indexed depository);

/// @audit Missing '@param dispenser'
38   /// invariant {:msg "broken conservation law"} address(this).balance == ETHFromServices + ETHOwned;
39   contract Treasury is IErrorsTokenomics {
40       event OwnerUpdated(address indexed owner);
41       event TokenomicsUpdated(address indexed tokenomics);
42       event DepositoryUpdated(address indexed depository);
43:      event DispenserUpdated(address indexed dispenser);

/// @audit Missing '@param account'
/// @audit Missing '@param token'
/// @audit Missing '@param tokenAmount'
/// @audit Missing '@param olasAmount'
39   contract Treasury is IErrorsTokenomics {
40       event OwnerUpdated(address indexed owner);
41       event TokenomicsUpdated(address indexed tokenomics);
42       event DepositoryUpdated(address indexed depository);
43       event DispenserUpdated(address indexed dispenser);
44:      event DepositTokenFromAccount(address indexed account, address indexed token, uint256 tokenAmount, uint256 olasAmount);

/// @audit Missing '@param sender'
/// @audit Missing '@param serviceIds'
/// @audit Missing '@param amounts'
/// @audit Missing '@param donation'
40       event OwnerUpdated(address indexed owner);
41       event TokenomicsUpdated(address indexed tokenomics);
42       event DepositoryUpdated(address indexed depository);
43       event DispenserUpdated(address indexed dispenser);
44       event DepositTokenFromAccount(address indexed account, address indexed token, uint256 tokenAmount, uint256 olasAmount);
45:      event DonateToServicesETH(address indexed sender, uint256[] serviceIds, uint256[] amounts, uint256 donation);

/// @audit Missing '@param token'
/// @audit Missing '@param to'
/// @audit Missing '@param tokenAmount'
41       event TokenomicsUpdated(address indexed tokenomics);
42       event DepositoryUpdated(address indexed depository);
43       event DispenserUpdated(address indexed dispenser);
44       event DepositTokenFromAccount(address indexed account, address indexed token, uint256 tokenAmount, uint256 olasAmount);
45       event DonateToServicesETH(address indexed sender, uint256[] serviceIds, uint256[] amounts, uint256 donation);
46:      event Withdraw(address indexed token, address indexed to, uint256 tokenAmount);

/// @audit Missing '@param token'
42       event DepositoryUpdated(address indexed depository);
43       event DispenserUpdated(address indexed dispenser);
44       event DepositTokenFromAccount(address indexed account, address indexed token, uint256 tokenAmount, uint256 olasAmount);
45       event DonateToServicesETH(address indexed sender, uint256[] serviceIds, uint256[] amounts, uint256 donation);
46       event Withdraw(address indexed token, address indexed to, uint256 tokenAmount);
47:      event EnableToken(address indexed token);

/// @audit Missing '@param token'
43       event DispenserUpdated(address indexed dispenser);
44       event DepositTokenFromAccount(address indexed account, address indexed token, uint256 tokenAmount, uint256 olasAmount);
45       event DonateToServicesETH(address indexed sender, uint256[] serviceIds, uint256[] amounts, uint256 donation);
46       event Withdraw(address indexed token, address indexed to, uint256 tokenAmount);
47       event EnableToken(address indexed token);
48:      event DisableToken(address indexed token);

/// @audit Missing '@param sender'
/// @audit Missing '@param amount'
44       event DepositTokenFromAccount(address indexed account, address indexed token, uint256 tokenAmount, uint256 olasAmount);
45       event DonateToServicesETH(address indexed sender, uint256[] serviceIds, uint256[] amounts, uint256 donation);
46       event Withdraw(address indexed token, address indexed to, uint256 tokenAmount);
47       event EnableToken(address indexed token);
48       event DisableToken(address indexed token);
49:      event ReceiveETH(address indexed sender, uint256 amount);

/// @audit Missing '@param ETHOwned'
/// @audit Missing '@param ETHFromServices'
45       event DonateToServicesETH(address indexed sender, uint256[] serviceIds, uint256[] amounts, uint256 donation);
46       event Withdraw(address indexed token, address indexed to, uint256 tokenAmount);
47       event EnableToken(address indexed token);
48       event DisableToken(address indexed token);
49       event ReceiveETH(address indexed sender, uint256 amount);
50:      event UpdateTreasuryBalances(uint256 ETHOwned, uint256 ETHFromServices);

/// @audit Missing '@param amount'
48       event DisableToken(address indexed token);
49       event ReceiveETH(address indexed sender, uint256 amount);
50       event UpdateTreasuryBalances(uint256 ETHOwned, uint256 ETHFromServices);
51       event PauseTreasury();
52       event UnpauseTreasury();
53:      event MinAcceptedETHUpdated(uint256 amount);

```
*GitHub*: [35](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L35-L40), [36](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L36-L41), [37](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L37-L42), [38](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L38-L43), [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L39-L44), [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L39-L44), [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L39-L44), [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L39-L44), [40](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L40-L45), [40](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L40-L45), [40](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L40-L45), [40](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L40-L45), [41](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L41-L46), [41](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L41-L46), [41](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L41-L46), [42](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L42-L47), [43](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L43-L48), [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L44-L49), [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L44-L49), [45](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L45-L50), [45](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L45-L50), [48](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L48-L53)

</details>





### [N&#x2011;18] NatSpec: Function `@param` tag is missing


*There is one instance of this issue:*

```solidity
File: contracts/GovernorOLAS.sol

/// @audit Missing '@param quorumFraction'
17           IVotes governanceToken,
18           TimelockController timelock,
19           uint256 initialVotingDelay,
20           uint256 initialVotingPeriod,
21           uint256 initialProposalThreshold,
22:          uint256 quorumFraction

```
*GitHub*: [17](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L17-L22)



### [N&#x2011;19] NatSpec: Function `@return` tag is missing


*There are 8 instances of this issue:*

```solidity
File: contracts/bridges/HomeMediator.sol

/// @audit Missing '@return  '
6:       function messageSender() external view returns (address);

```
*GitHub*: [6](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L6-L6)

```solidity
File: contracts/multisigs/GuardCM.sol

/// @audit Missing '@return  '
7:       function state(uint256 proposalId) external returns (ProposalState);

```
*GitHub*: [7](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L7-L7)

```solidity
File: contracts/interfaces/IUniswapV2Pair.sol

/// @audit Missing '@return  '
6:       function totalSupply() external view returns (uint);

/// @audit Missing '@return  '
7:       function token0() external view returns (address);

/// @audit Missing '@return  '
8:       function token1() external view returns (address);

/// @audit Missing '@return reserve0'
/// @audit Missing '@return reserve1'
/// @audit Missing '@return blockTimestampLast'
9:       function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
*GitHub*: [6](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L8-L8), [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L9-L9), [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L9-L9), [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L9-L9)



### [N&#x2011;20] Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.

*There are 4 instances of this issue:*

```solidity
File: contracts/UnitRegistry.sol

/// @audit numDependencies
/// @audit dependencies
160      function getDependencies(uint256 unitId) external view virtual
161          returns (uint256 numDependencies, uint32[] memory dependencies)
162:     {

/// @audit numHashes
171      function getUpdatedHashes(uint256 unitId) external view virtual
172          returns (uint256 numHashes, bytes32[] memory unitHashes)
173:     {

```
*GitHub*: [160](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L160-L162), [160](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L160-L162), [171](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L171-L173)

```solidity
File: contracts/Depository.sol

/// @audit priceLP
491:     function getCurrentPriceLP(address token) external view returns (uint256 priceLP) {

```
*GitHub*: [491](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L491-L491)



### [N&#x2011;21] Style guide: Local and state variables should be named using lowerCamelCase
The Solidity style guide [says](https://docs.soliditylang.org/en/latest/style-guide.html#function-argument-names) to use mixedCase for function argument names

*There is one instance of this issue:*

```solidity
File: contracts/bridges/HomeMediator.sol

62:      constructor(address _AMBContractProxyHome, address _foreignGovernor) {

```
*GitHub*: [62](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L62-L62)



### [N&#x2011;22] Style guide: Non-`external`/`public` variable names should begin with an underscore
According to the Solidity Style Guide, non-`external`/`public` variable names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*There are 3 instances of this issue:*

```solidity
File: contracts/veOLAS.sol

99:      uint64 internal constant WEEK = 1 weeks;

101:     uint256 internal constant MAXTIME = 4 * 365 * 86400;

103:     int128 internal constant IMAXTIME = 4 * 365 * 86400;

```
*GitHub*: [99](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L99-L99), [101](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L101-L101), [103](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L103-L103)



### [N&#x2011;23] Style guide: Top-level declarations should be separated by at least two lines
And functions within contracts should be separate by a [single](https://docs.soliditylang.org/en/latest/style-guide.html#blank-lines) line

*There are 2 instances of this issue:*

```solidity
File: contracts/multisigs/GuardCM.sol

4     import {Enum} from "@gnosis.pm/safe-contracts/contracts/common/Enum.sol";
5     
6:    interface IGovernor {

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L4-L6)

```solidity
File: contracts/wveOLAS.sol

11    }
12    
13:   interface IVEOLAS {

```
*GitHub*: [11](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L11-L13)



### [N&#x2011;24] Unnecessary cast
The variable is being cast to its own type

*There are 3 instances of this issue:*

```solidity
File: contracts/veOLAS.sol

/// @audit uint256
/// @audit uint256
225:             block_slope = (1e18 * uint256(block.number - lastPoint.blockNumber)) / uint256(block.timestamp - lastPoint.ts);

```
*GitHub*: [225](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L225-L225), [225](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L225-L225)

```solidity
File: contracts/UnitRegistry.sol

/// @audit uint256
185:         subComponentIds = mapSubComponents[uint256(unitId)];

```
*GitHub*: [185](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L185-L185)



### [N&#x2011;25] Unused `error` definition
Note that there may be cases where an error superficially appears to be used, but this is only because there are multiple definitions of the error in different files. In such cases, the error definition should be moved into a separate file. The instances below are the unused definitions.

*There are 9 instances of this issue:*

```solidity
File: contracts/interfaces/IErrors.sol

9:       error OwnerOnly(address sender, address owner);

18:      error NonZeroValue();

23:      error WrongArrayLength(uint256 numValues1, uint256 numValues2);

41:      error InsufficientAllowance(uint256 provided, uint256 expected);

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IErrors.sol#L9-L9), [18](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IErrors.sol#L18-L18), [23](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IErrors.sol#L23-L23), [41](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/interfaces/IErrors.sol#L41-L41)

```solidity
File: contracts/interfaces/IErrorsRegistries.sol

29:      error WrongArrayLength(uint256 numValues1, uint256 numValues2);

43:      error WrongThreshold(uint256 currentThreshold, uint256 minThreshold, uint256 maxThreshold);

95:      error UnauthorizedMultisig(address multisig);

114:     error TransferFailed(address token, address from, address to, uint256 value);

```
*GitHub*: [29](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/interfaces/IErrorsRegistries.sol#L29-L29), [43](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/interfaces/IErrorsRegistries.sol#L43-L43), [95](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/interfaces/IErrorsRegistries.sol#L95-L95), [114](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/interfaces/IErrorsRegistries.sol#L114-L114)

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

19:  error ZeroAddress();

```
*GitHub*: [19](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L19-L19)



### [N&#x2011;26] Unused `public` contract variable
Note that there may be cases where a variable superficially appears to be used, but this is only because there are multiple definitions of the variable in different files. In such cases, the variable definition should be moved into a separate file. The instances below are the unused variables.

*There are 9 instances of this issue:*

```solidity
File: contracts/AgentRegistry.sol

13:      string public constant VERSION = "1.0.0";

```
*GitHub*: [13](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L13-L13)

```solidity
File: contracts/ComponentRegistry.sol

10:      string public constant VERSION = "1.0.0";

```
*GitHub*: [10](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L10-L10)

```solidity
File: contracts/Depository.sol

77:      string public constant VERSION = "1.0.1";

```
*GitHub*: [77](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L77-L77)

```solidity
File: contracts/Tokenomics.sol

217:     mapping(uint256 => uint256) public mapServiceAmounts;

219:     mapping(address => uint256) public mapOwnerRewards;

221:     mapping(address => uint256) public mapOwnerTopUps;

```
*GitHub*: [217](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L217-L217), [219](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L219-L219), [221](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L221-L221)

```solidity
File: contracts/TokenomicsConstants.sol

11:      string public constant VERSION = "1.0.1";

14:      bytes32 public constant PROXY_TOKENOMICS = 0xbd5523e7c3b6a94aa0e3b24d1120addc2f95c7029e097b466b2bedc8d4b4362f;

```
*GitHub*: [11](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L11-L11), [14](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L14-L14)

```solidity
File: contracts/TokenomicsProxy.sol

28:      bytes32 public constant PROXY_TOKENOMICS = 0xbd5523e7c3b6a94aa0e3b24d1120addc2f95c7029e097b466b2bedc8d4b4362f;

```
*GitHub*: [28](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsProxy.sol#L28-L28)



### [N&#x2011;27] Use `bytes.concat()` on bytes instead of `abi.encodePacked()` for clearer semantic meaning
Starting with version 0.8.4, Solidity has the `bytes.concat()` function, which allows one to concatenate a list of bytes/strings, without extra padding. Using this function rather than `abi.encodePacked()` makes the intended operation more clear, leading to less reviewer confusion.

*There is one instance of this issue:*

```solidity
File: contracts/GenericRegistry.sol

139          return string(abi.encodePacked(baseURI, CID_PREFIX, _toHex16(bytes16(unitHash)),
140:             _toHex16(bytes16(unitHash << 128))));

```
*GitHub*: [139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L139-L140)



### [N&#x2011;28] Use bit shifts to represent powers of two
Use bit shifts in an immutable variable rather than long hex strings for powers of two, for readability (e.g. `0x1000000000000000000000000` -> `1 << 96`)

*There are 7 instances of this issue:*

```solidity
File: contracts/Tokenomics.sol

550:         tokenomicsParametersUpdated = tokenomicsParametersUpdated | 0x01;

599:         tokenomicsParametersUpdated = tokenomicsParametersUpdated | 0x02;

942:             tokenomicsParametersUpdated = tokenomicsParametersUpdated | 0x04;

973:         if (tokenomicsParametersUpdated & 0x02 == 0x02) {

973:         if (tokenomicsParametersUpdated & 0x02 == 0x02) {

987:         if (tokenomicsParametersUpdated & 0x01 == 0x01) {

987:         if (tokenomicsParametersUpdated & 0x01 == 0x01) {

```
*GitHub*: [550](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L550-L550), [599](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L599-L599), [942](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L942-L942), [973](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L973-L973), [973](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L973-L973), [987](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L987-L987), [987](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L987-L987)



### [N&#x2011;29] Use of `override` is unnecessary
Starting with Solidity version [0.8.8](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding), using the `override` keyword when the function solely overrides an interface function, and the function doesn't exist in multiple base contracts, is unnecessary.

*There are 14 instances of this issue:*

```solidity
File: contracts/GovernorOLAS.sol

33       function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
34           returns (ProposalState)
35:      {

45       function propose(
46           address[] memory targets,
47           uint256[] memory values,
48           bytes[] memory calldatas,
49           string memory description
50       ) public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)
51:      {

57       function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)
58:      {

68       function _execute(
69           uint256 proposalId,
70           address[] memory targets,
71           uint256[] memory values,
72           bytes[] memory calldatas,
73           bytes32 descriptionHash
74       ) internal override(Governor, GovernorTimelockControl)
75:      {

85       function _cancel(
86           address[] memory targets,
87           uint256[] memory values,
88           bytes[] memory calldatas,
89           bytes32 descriptionHash
90       ) internal override(Governor, GovernorTimelockControl) returns (uint256)
91:      {

97       function _executor() internal view override(Governor, GovernorTimelockControl) returns (address)
98:      {

105      function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
106          returns (bool)
107:     {

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L33-L35), [45](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L45-L51), [57](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L57-L58), [68](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L68-L75), [85](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L85-L91), [97](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L97-L98), [105](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L105-L107)

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

107:     function processMessageFromRoot(uint256 stateId, address rootMessageSender, bytes memory data) external override {

```
*GitHub*: [107](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L107-L107)

```solidity
File: contracts/veOLAS.sol

607:     function balanceOf(address account) public view override returns (uint256 balance) {

719:     function totalSupply() public view override returns (uint256) {

767:     function transfer(address to, uint256 amount) external virtual override returns (bool) {

772:     function approve(address spender, uint256 amount) external virtual override returns (bool) {

777:     function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {

782      function allowance(address owner, address spender) external view virtual override returns (uint256)
783:     {

```
*GitHub*: [607](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L607-L607), [719](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L719-L719), [767](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L767-L767), [772](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L772-L772), [777](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L777-L777), [782](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L782-L783)
