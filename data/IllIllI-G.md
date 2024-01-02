**Note:** I've removed the instances reported by the winning bot and 4naly3er

## Summary

### Gas Optimizations

| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [[G&#x2011;01](#g01-argument-validation-should-be-at-the-top-of-the-functionblock)] | Argument validation should be at the top of the function/block | 41 |  - |
| [[G&#x2011;02](#g02-mappings-are-cheaper-to-use-than-storage-arrays)] | Mappings are cheaper to use than storage arrays | 3 |  6300 |
| [[G&#x2011;03](#g03-state-variables-only-set-in-the-constructor-should-be-declared-immutable)] | State variables only set in the constructor should be declared `immutable` | 1 |  2097 |
| [[G&#x2011;04](#g04-emitting-indexed-constants-wastes-gas)] | Emitting `index`ed `constant`s wastes gas | 10 |  3750 |
| [[G&#x2011;05](#g05-assembly-use-scratch-space-for-building-calldata)] | Assembly: Use scratch space for building calldata | 39 |  8580 |
| [[G&#x2011;06](#g06-use--for-arrays)] | Use `+=` for arrays | 4 |  796 |
| [[G&#x2011;07](#g07-assembly-avoid-fetching-a-low-level-calls-return-data-by-using-assembly)] | Assembly: Avoid fetching a low-level call's return data by using assembly | 2 |  318 |
| [[G&#x2011;08](#g08-avoid-transferring-amounts-of-zero-in-order-to-save-gas)] | Avoid transferring amounts of zero in order to save gas | 2 |  200 |
| [[G&#x2011;09](#g09-combine-mappings-referenced-in-the-same-function-by-the-same-key)] | Combine `mapping`s referenced in the same function by the same key | 7 |  294 |
| [[G&#x2011;10](#g10-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)] | Functions guaranteed to revert when called by normal users can be marked `payable` | 56 |  1176 |
| [[G&#x2011;11](#g11-internal-functions-only-called-once-can-be-inlined-to-save-gas)] | `internal` functions only called once can be inlined to save gas | 2 |  40 |
| [[G&#x2011;12](#g12-assembly-use-scratch-space-when-building-emitted-events-with-two-data-arguments)] | Assembly: Use scratch space when building emitted events with two data arguments | 10 |  150 |
| [[G&#x2011;13](#g13--costs-less-gas-than-)] | `>=` costs less gas than `>` | 51 |  153 |
| [[G&#x2011;14](#g14-consider-using-erc721a-over-erc721)] | Consider using `ERC721A` over `ERC721` | 3 |  - |
| [[G&#x2011;15](#g15-consider-using-soladys-token-contracts-to-save-gas)] | Consider using `solady`'s token contracts to save gas | 3 |  - |
| [[G&#x2011;16](#g16-consider-using-soladys-fixedpointmathlib)] | Consider using solady's `FixedPointMathLib` | 23 |  - |
| [[G&#x2011;17](#g17-initializers-can-be-marked-payable)] | Initializers can be marked `payable` | 1 |  - |
| [[G&#x2011;18](#g18-memory-safe-annotation-missing)] | Memory-safe annotation missing | 6 |  - |
| [[G&#x2011;19](#g19-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct)] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct` | 4 |  - |
| [[G&#x2011;20](#g20-multiple-accesses-of-a-memorycalldata-array-should-use-a-local-variable-cache)] | Multiple accesses of a `memory`/`calldata` array should use a local variable cache | 84 |  - |
| [[G&#x2011;21](#g21-multiple-accesses-of-a-storage-array-should-use-a-local-variable-cache)] | Multiple accesses of a storage array should use a local variable cache | 2 |  84 |
| [[G&#x2011;22](#g22-reduce-deployment-costs-by-tweaking-contracts-metadata)] | Reduce deployment costs by tweaking contracts' metadata | 23 |  - |
| [[G&#x2011;23](#g23-stack-variable-is-only-used-once)] | Stack variable is only used once | 32 |  96 |
| [[G&#x2011;24](#g24-split-revert-checks-to-save-gas)] | Split `revert` checks to save gas | 19 |  38 |
| [[G&#x2011;25](#g25-update-openzeppelin-dependency-to-save-gas)] | Update OpenZeppelin dependency to save gas | 8 |  - |
| [[G&#x2011;26](#g26-use-short-circuit-evaluation-to-avoid-external-calls)] | Use short-circuit evaluation to avoid external calls | 1 |  - |

Total: 437 instances over 26 issues with **24072 gas** saved

Gas totals are estimates based on data from the Ethereum Yellowpaper. The estimates use the lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations


### [G&#x2011;01] Argument validation should be at the top of the function/block
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (**2100 gas***) in a function that may ultimately revert in the unhappy case.

*There are 41 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/OLAS.sol

48           if (newOwner == address(0)) {
49               revert ZeroAddress();
50:          }

63           if (newMinter == address(0)) {
64               revert ZeroAddress();
65:          }

```
*GitHub*: [48](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L48-L50), [63](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L63-L65)

```solidity
File: contracts/bridges/BridgedERC20.sol

37           if (newOwner == address(0)) {
38               revert ZeroAddress();
39:          }

```
*GitHub*: [37](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/BridgedERC20.sol#L37-L39)

```solidity
File: contracts/veOLAS.sol

379          if (amount == 0) {
380              revert ZeroValue();
381:         }

392          if (amount > type(uint96).max) {
393              revert Overflow(amount, type(uint96).max);
394:         }

449          if (amount > type(uint96).max) {
450              revert Overflow(amount, type(uint96).max);
451:         }

461          if (amount == 0) {
462              revert ZeroValue();
463:         }

474          if (amount > type(uint96).max) {
475              revert Overflow(amount, type(uint96).max);
476:         }

```
*GitHub*: [379](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L379-L381), [392](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L392-L394), [449](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L449-L451), [461](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L461-L463), [474](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L474-L476)

```solidity
File: contracts/GenericManager.sol

27           if (newOwner == address(0)) {
28               revert ZeroAddress();
29:          }

```
*GitHub*: [27](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L27-L29)

```solidity
File: contracts/GenericRegistry.sol

44           if (newOwner == address(0)) {
45               revert ZeroAddress();
46:          }

60           if (newManager == address(0)) {
61               revert ZeroAddress();
62:          }

85           if (bytes(bURI).length == 0) {
86               revert ZeroValue();
87:          }

```
*GitHub*: [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L44-L46), [60](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L60-L62), [85](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L85-L87)

```solidity
File: contracts/UnitRegistry.sol

64           if(unitOwner == address(0)) {
65               revert ZeroAddress();
66:          }

69           if (unitHash == 0) {
70               revert ZeroValue();
71:          }

139          if (unitHash == 0) {
140              revert ZeroValue();
141:         }

```
*GitHub*: [64](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L64-L66), [69](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L69-L71), [139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L139-L141)

```solidity
File: contracts/Depository.sol

111          if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) {
112              revert ZeroAddress();
113:         }

130          if (newOwner == address(0)) {
131              revert ZeroAddress();
132:         }

190          if (priceLP == 0) {
191              revert ZeroValue();
192:         }

195          if (priceLP > type(uint160).max) {
196              revert Overflow(priceLP, type(uint160).max);
197:         }

200          if (supply == 0) {
201              revert ZeroValue();
202:         }

205          if (supply > type(uint96).max) {
206              revert Overflow(supply, type(uint96).max);
207:         }

210          if (vesting < MIN_VESTING) {
211              revert LowerThan(vesting, MIN_VESTING);
212:         }

```
*GitHub*: [111](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L111-L113), [130](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L130-L132), [190](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L190-L192), [195](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L195-L197), [200](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L200-L202), [205](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L205-L207), [210](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L210-L212)

```solidity
File: contracts/Dispenser.sol

36           if (_tokenomics == address(0) || _treasury == address(0)) {
37               revert ZeroAddress();
38:          }

53           if (newOwner == address(0)) {
54               revert ZeroAddress();
55:          }

```
*GitHub*: [36](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L36-L38), [53](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L53-L55)

```solidity
File: contracts/DonatorBlacklist.sol

43           if (newOwner == address(0)) {
44               revert ZeroAddress();
45:          }

63           if (accounts.length != statuses.length) {
64               revert WrongArrayLength(accounts.length, statuses.length);
65:          }

```
*GitHub*: [43](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L43-L45), [63](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L63-L65)

```solidity
File: contracts/Tokenomics.sol

283          if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
284              _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
285              _serviceRegistry == address(0)) {
286              revert ZeroAddress();
287:         }

296          if (uint32(_epochLen) < MIN_EPOCH_LENGTH) {
297              revert LowerThan(_epochLen, MIN_EPOCH_LENGTH);
298:         }

301          if (uint32(_epochLen) > ONE_YEAR) {
302              revert Overflow(_epochLen, ONE_YEAR);
303:         }

391          if (implementation == address(0)) {
392              revert ZeroAddress();
393:         }

411          if (newOwner == address(0)) {
412              revert ZeroAddress();
413:         }

576          if (_rewardComponentFraction + _rewardAgentFraction > 100) {
577              revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);
578:         }

581          if (_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction > 100) {
582              revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);
583:         }

1094         if (unitTypes.length != unitIds.length) {
1095             revert WrongArrayLength(unitTypes.length, unitIds.length);
1096:        }

```
*GitHub*: [283](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L283-L287), [296](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L296-L298), [301](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L301-L303), [391](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L391-L393), [411](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L411-L413), [576](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L576-L578), [581](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L581-L583), [1094](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1094-L1096)

```solidity
File: contracts/Treasury.sol

100          if (_olas == address(0) || _tokenomics == address(0) || _depository == address(0) || _dispenser == address(0)) {
101              revert ZeroAddress();
102:         }

144          if (newOwner == address(0)) {
145              revert ZeroAddress();
146:         }

189          if (_minAcceptedETH == 0) {
190              revert ZeroValue();
191:         }

194          if (_minAcceptedETH > type(uint96).max) {
195              revert Overflow(_minAcceptedETH, type(uint96).max);
196:         }

320          if (to == address(this)) {
321              revert TransferFailed(token, address(this), to, tokenAmount);
322:         }

325          if (tokenAmount == 0) {
326              revert ZeroValue();
327:         }

494          if (token == address(0)) {
495              revert ZeroAddress();
496:         }

```
*GitHub*: [100](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L100-L102), [144](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L144-L146), [189](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L189-L191), [194](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L194-L196), [320](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L320-L322), [325](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L325-L327), [494](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L494-L496)

</details>





### [G&#x2011;02] Mappings are cheaper to use than storage arrays
When using storage arrays, solidity adds an internal lookup of the array's length (a Gcoldsload **2100 gas**) to ensure you don't read past the array's end. You can avoid this lookup by using a `mapping` and storing the number of entries in a separate storage variable. In cases where you have sentinel values (e.g. 'zero' means invalid), you can avoid length checks

*There are 3 instances of this issue:*

```solidity
File: contracts/veOLAS.sol

119:     mapping(address => PointVoting[]) public mapUserPoints;

```
*GitHub*: [119](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L119-L119)

```solidity
File: contracts/UnitRegistry.sol

29:      mapping(uint256 => bytes32[]) public mapUnitIdHashes;

31:      mapping(uint256 => uint32[]) public mapSubComponents;

```
*GitHub*: [29](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L29-L29), [31](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L31-L31)



### [G&#x2011;03] State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (**20000 gas**) in the constructor, and replaces the first access in each transaction (Gcoldsload - **2100 gas**) and each access thereafter (Gwarmacces - **100 gas**) with a `PUSH32` (**3 gas**). 

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*There is one instance of this issue:*

```solidity
File: contracts/Treasury.sol

65:      address public olas;

```
*GitHub*: [65](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L65-L65)



### [G&#x2011;04] Emitting `index`ed `constant`s wastes gas
Every `indexed` event parameter adds an extra `Glogtopic` (**375 gas**). You can avoid this extra cost, in cases where you're emitting a constant, by creating a second version of the event, which doesn't have the indexed parameter (and have users look to the contract's variables for its value instead), and using the new event in the cases shown below.

*There are 10 instances of this issue:*

```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

/// @audit arg0 is `indexed rootToken`
/// @audit arg1 is `indexed childToken`
84:          emit FxWithdrawERC20(rootToken, childToken, from, to, amount);

/// @audit arg0 is `indexed childToken`
/// @audit arg1 is `indexed rootToken`
107:         emit FxDepositERC20(childToken, rootToken, msg.sender, to, amount);

```
*GitHub*: [84](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L84-L84), [84](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L84-L84), [107](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L107-L107), [107](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L107-L107)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

/// @audit arg0 is `indexed childToken`
/// @audit arg1 is `indexed rootToken`
79:          emit FxDepositERC20(childToken, rootToken, from, to, amount);

/// @audit arg0 is `indexed rootToken`
/// @audit arg1 is `indexed childToken`
106:         emit FxWithdrawERC20(rootToken, childToken, msg.sender, to, amount);

```
*GitHub*: [79](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L79-L79), [79](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L79-L79), [106](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L106-L106), [106](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L106-L106)

```solidity
File: contracts/Treasury.sol

/// @audit arg0 is `indexed token`
337:                 emit Withdraw(ETH_TOKEN_ADDRESS, to, tokenAmount);

/// @audit arg0 is `indexed token`
405:             emit Withdraw(ETH_TOKEN_ADDRESS, account, accountRewards);

```
*GitHub*: [337](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L337-L337), [405](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L405-L405)



### [G&#x2011;05] Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

*There are 39 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

79:          bool success = IERC20(childToken).transfer(to, amount);

```
*GitHub*: [79](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79-L79)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

77:          IERC20(rootToken).mint(to, amount);

104:         IERC20(rootToken).burn(amount);

```
*GitHub*: [77](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L77-L77), [104](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L104-L104)

```solidity
File: contracts/multisigs/GuardCM.sol

545:             ProposalState state = IGovernor(governor).state(governorCheckProposalId);

```
*GitHub*: [545](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L545-L545)

```solidity
File: contracts/veOLAS.sol

534:         IERC20(token).transfer(msg.sender, amount);

```
*GitHub*: [534](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L534-L534)

```solidity
File: contracts/wveOLAS.sol

164:         sPoint = IVEOLAS(ve).mapSupplyPoints(idx);

171:         slopeChange = IVEOLAS(ve).mapSlopeChanges(ts);

178:         pv = IVEOLAS(ve).getLastUserPoint(account);

185:         userNumPoints = IVEOLAS(ve).getNumUserPoints(account);

195:         uint256 userNumPoints = IVEOLAS(ve).getNumUserPoints(account);

197:             uPoint = IVEOLAS(ve).getUserPoint(account, idx);

204:         balance = IVEOLAS(ve).getVotes(account);

216:             balance = IVEOLAS(ve).getPastVotes(account, blockNumber);

224:         balance = IVEOLAS(ve).balanceOf(account);

236:             balance = IVEOLAS(ve).balanceOfAt(account, blockNumber);

244:         unlockTime = IVEOLAS(ve).lockedEnd(account);

257:         supplyAt = IVEOLAS(ve).totalSupplyAt(blockNumber);

266:         PointVoting memory sPoint = IVEOLAS(ve).mapSupplyPoints(numPoints);

269:             vPower = IVEOLAS(ve).totalSupplyLockedAtT(ts);

286:         vPower = IVEOLAS(ve).getPastTotalSupply(blockNumber);

293:         return IVEOLAS(ve).supportsInterface(interfaceId);

```
*GitHub*: [164](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L164-L164), [171](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L171-L171), [178](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L178-L178), [185](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L185-L185), [195](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L195-L195), [197](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L197-L197), [204](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L204-L204), [216](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L216-L216), [224](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L224-L224), [236](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L236-L236), [244](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L244-L244), [257](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L257-L257), [266](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L266-L266), [269](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L269-L269), [286](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L286-L286), [293](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L293-L293)

```solidity
File: contracts/AgentRegistry.sol

60:              (subComponentIds, ) = IRegistry(componentRegistry).getLocalSubComponents(uint256(unitId));

```
*GitHub*: [60](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L60-L60)

```solidity
File: contracts/Depository.sol

221:         if (!ITreasury(treasury).isEnabled(token)) {

226:         if (!ITokenomics(tokenomics).reserveAmountForBondProgram(supply)) {

262:                 ITokenomics(tokenomics).refundFromBondProgram(supply);

320:         payout = IGenericBondCalculator(bondCalculator).calculatePayoutOLAS(tokenAmount, product.priceLP);

390:         IToken(olas).transfer(msg.sender, payout);

492:         return IGenericBondCalculator(bondCalculator).getCurrentPriceLP(token);

```
*GitHub*: [221](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L221-L221), [226](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L226-L226), [262](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L262-L262), [320](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L320-L320), [390](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L390-L390), [492](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L492-L492)

```solidity
File: contracts/Tokenomics.sol

708:                 address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);

709:                 topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||

710:                     IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;

801:         if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) {

810:             if (!IServiceRegistry(serviceRegistry).exists(serviceIds[i])) {

1063:        if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) {

```
*GitHub*: [708](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L708-L708), [709](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L709-L709), [710](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L710-L710), [801](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L801-L801), [810](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L810-L810), [1063](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1063-L1063)

```solidity
File: contracts/Treasury.sol

228:         if (IToken(token).allowance(account, address(this)) < tokenAmount) {

229:             revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);

242:         IOLAS(olas).mint(msg.sender, olasMintAmount);

362:                 success = IToken(token).transfer(to, tokenAmount);

416:             IOLAS(olas).mint(account, accountTopUps);

```
*GitHub*: [228](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L228-L228), [229](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L229-L229), [242](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L242-L242), [362](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L362-L362), [416](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L416-L416)

</details>





### [G&#x2011;06] Use `+=` for arrays
Using `+=` for arrays saves **[199 gas](https://gist.github.com/IllIllI000/07218008c1568664269314bd8d40d497)** due to not having to re-fetch the array's length

*There are 4 instances of this issue:*

```solidity
File: contracts/Tokenomics.sol

915:         incentives[1] = (incentives[0] * tp.epochPoint.rewardTreasuryFraction) / 100;

917:         incentives[2] = (incentives[0] * tp.unitPoints[0].rewardUnitFraction) / 100;

918:         incentives[3] = (incentives[0] * tp.unitPoints[1].rewardUnitFraction) / 100;

965:             incentives[4] = effectiveBond + incentives[4] - curMaxBond;

```
*GitHub*: [915](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L915-L915), [917](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L917-L917), [918](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L918-L918), [965](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L965-L965)



### [G&#x2011;07] Assembly: Avoid fetching a low-level call's return data by using assembly
Even if you don't assign the call's second return value, it still gets copied to memory. Use assembly instead to prevent this and save **159 [gas](https://gist.github.com/IllIllI000/0e18a40f3afb0b83f9a347b10ee89ad2)**:

`(bool success,) = payable(receiver).call{gas: gas, value: value}("");` -> `bool success; assembly { success := call(gas, receiver, value, 0, 0, 0, 0) }`

*There are 2 instances of this issue:*

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

118:             (bool success, ) = multisig.call(payload);

```
*GitHub*: [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L118-L118)

```solidity
File: contracts/TokenomicsProxy.sol

48:          (bool success, ) = tokenomics.delegatecall(tokenomicsData);

```
*GitHub*: [48](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsProxy.sol#L48-L48)



### [G&#x2011;08] Avoid transferring amounts of zero in order to save gas
Skipping the external call when nothing will be transferred, will save at least **100 gas**

*There are 2 instances of this issue:*

```solidity
File: contracts/veOLAS.sol

534:         IERC20(token).transfer(msg.sender, amount);

```
*GitHub*: [534](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L534-L534)

```solidity
File: contracts/Treasury.sol

235:         bool success = IToken(token).transferFrom(account, address(this), tokenAmount);

```
*GitHub*: [235](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L235-L235)



### [G&#x2011;09] Combine `mapping`s referenced in the same function by the same key
Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot (i.e. runtime gas savings). Even if the values can't be packed, if both fields are accessed in the same function (as is the case for these instances), combining them can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/639032d73e35d7e968ff58d8784bc445) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There are 7 instances of this issue:*

```solidity
File: contracts/UnitRegistry.sol

/// @audit combine into a `struct`: mapSubComponents,mapUnits
49       function create(address unitOwner, bytes32 unitHash, uint32[] memory dependencies)
50           external virtual returns (uint256 unitId)
51       {
52           // Reentrancy guard
53           if (_locked > 1) {
54               revert ReentrancyGuard();
55           }
56           _locked = 2;
57   
58           // Check for the manager privilege for a unit creation
59           if (manager != msg.sender) {
60               revert ManagerOnly(msg.sender, manager);
61           }
62   
63           // Checks for a non-zero owner address
64           if(unitOwner == address(0)) {
65               revert ZeroAddress();
66           }
67   
68           // Check for the non-zero hash value
69           if (unitHash == 0) {
70               revert ZeroValue();
71           }
72           
73           // Check for dependencies validity: must be already allocated, must not repeat
74           unitId = totalSupply;
75           _checkDependencies(dependencies, uint32(unitId));
76   
77           // Unit with Id = 0 is left empty not to do additional checks for the index zero
78           unitId++;
79   
80           // Initialize the unit and mint its token
81           Unit storage unit = mapUnits[unitId];
82           unit.unitHash = unitHash;
83           unit.dependencies = dependencies;
84   
85           // Update the map of subcomponents with calculated subcomponents for the new unit Id
86           // In order to get the correct set of subcomponents, we need to differentiate between the callers of this function
87           // Self contract (unit registry) can only call subcomponents calculation from the component level
88           uint32[] memory subComponentIds = _calculateSubComponents(UnitType.Component, dependencies);
89           // We need to add a current component Id to the set of subcomponents if the unit is a component
90           // For example, if component 3 (c3) has dependencies of [c1, c2], then the subcomponents will return [c1, c2].
91           // The resulting set will be [c1, c2, c3]. So we write into the map of component subcomponents: c3=>[c1, c2, c3].
92           // This is done such that the subcomponents start getting explored, and when the agent calls its subcomponents,
93           // it would have [c1, c2, c3] right away instead of adding c3 manually and then (for services) checking
94           // if another agent also has c3 as a component dependency. The latter will consume additional computation.
95           if (unitType == UnitType.Component) {
96               uint256 numSubComponents = subComponentIds.length;
97               uint32[] memory addSubComponentIds = new uint32[](numSubComponents + 1);
98               for (uint256 i = 0; i < numSubComponents; ++i) {
99                   addSubComponentIds[i] = subComponentIds[i];
100              }
101              // Adding self component Id
102              addSubComponentIds[numSubComponents] = uint32(unitId);
103              subComponentIds = addSubComponentIds;
104          }
105          mapSubComponents[unitId] = subComponentIds;
106  
107          // Set total supply to the unit Id number
108          totalSupply = unitId;
109          // Safe mint is needed since contracts can create units as well
110          _safeMint(unitOwner, unitId);
111  
112          emit CreateUnit(unitId, unitType, unitHash);
113          _locked = 1;
114:     }

```
*GitHub*: [49](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L49-L114)

```solidity
File: contracts/Tokenomics.sol

/// @audit combine into a `struct`: mapEpochTokenomics,unitPoints
264      function initializeTokenomics(
265          address _olas,
266          address _treasury,
267          address _depository,
268          address _dispenser,
269          address _ve,
270          uint256 _epochLen,
271          address _componentRegistry,
272          address _agentRegistry,
273          address _serviceRegistry,
274          address _donatorBlacklist
275      ) external
276      {
277          // Check if the contract is already initialized
278          if (owner != address(0)) {
279              revert AlreadyInitialized();
280          }
281  
282          // Check for at least one zero contract address
283          if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
284              _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
285              _serviceRegistry == address(0)) {
286              revert ZeroAddress();
287          }
288  
289          // Initialize storage variables
290          owner = msg.sender;
291          _locked = 1;
292          epsilonRate = 1e17;
293          veOLASThreshold = 10_000e18;
294  
295          // Check that the epoch length has at least a practical minimal value
296          if (uint32(_epochLen) < MIN_EPOCH_LENGTH) {
297              revert LowerThan(_epochLen, MIN_EPOCH_LENGTH);
298          }
299  
300          // Check that the epoch length is not bigger than one year
301          if (uint32(_epochLen) > ONE_YEAR) {
302              revert Overflow(_epochLen, ONE_YEAR);
303          }
304  
305          // Assign other input variables
306          olas = _olas;
307          treasury = _treasury;
308          depository = _depository;
309          dispenser = _dispenser;
310          ve = _ve;
311          epochLen = uint32(_epochLen);
312          componentRegistry = _componentRegistry;
313          agentRegistry = _agentRegistry;
314          serviceRegistry = _serviceRegistry;
315          donatorBlacklist = _donatorBlacklist;
316  
317          // Time launch of the OLAS contract
318          uint256 _timeLaunch = IOLAS(_olas).timeLaunch();
319          // Check that the tokenomics contract is initialized no later than one year after the OLAS token is deployed
320          if (block.timestamp >= (_timeLaunch + ONE_YEAR)) {
321              revert Overflow(_timeLaunch + ONE_YEAR, block.timestamp);
322          }
323          // Seconds left in the deployment year for the zero year inflation schedule
324          // This value is necessary since it is different from a precise one year time, as the OLAS contract started earlier
325          uint256 zeroYearSecondsLeft = uint32(_timeLaunch + ONE_YEAR - block.timestamp);
326          // Calculating initial inflation per second: (mintable OLAS from getInflationForYear(0)) / (seconds left in a year)
327          // Note that we lose precision here dividing by the number of seconds right away, but to avoid complex calculations
328          // later we consider it less error-prone and sacrifice at most 6 insignificant digits (or 1e-12) of OLAS per year
329          uint256 _inflationPerSecond = getInflationForYear(0) / zeroYearSecondsLeft;
330          inflationPerSecond = uint96(_inflationPerSecond);
331          timeLaunch = uint32(_timeLaunch);
332  
333          // The initial epoch start time is the end time of the zero epoch
334          mapEpochTokenomics[0].epochPoint.endTime = uint32(block.timestamp);
335  
336          // The epoch counter starts from 1
337          epochCounter = 1;
338          TokenomicsPoint storage tp = mapEpochTokenomics[1];
339  
340          // Setting initial parameters and fractions
341          devsPerCapital = 1e18;
342          tp.epochPoint.idf = 1e18;
343  
344          // Reward fractions
345          // 0 stands for components and 1 for agents
346          // The initial target is to distribute around 2/3 of incentives reserved to fund owners of the code
347          // for components royalties and 1/3 for agents royalties
348          tp.unitPoints[0].rewardUnitFraction = 83;
349          tp.unitPoints[1].rewardUnitFraction = 17;
350          // tp.epochPoint.rewardTreasuryFraction is essentially equal to zero
351  
352          // We consider a unit of code as n agents or m components.
353          // Initially we consider 1 unit of code as either 2 agents or 1 component.
354          // E.g. if we have 2 profitable components and 2 profitable agents, this means there are (2 x 2.0 + 2 x 1.0) / 3 = 2
355          // units of code.
356          // We assume that during one epoch the developer can contribute with one piece of code (1 component or 2 agents)
357          codePerDev = 1e18;
358  
359          // Top-up fractions
360          uint256 _maxBondFraction = 50;
361          tp.epochPoint.maxBondFraction = uint8(_maxBondFraction);
362          tp.unitPoints[0].topUpUnitFraction = 41;
363          tp.unitPoints[1].topUpUnitFraction = 9;
364  
365          // Calculate initial effectiveBond based on the maxBond during the first epoch
366          // maxBond = inflationPerSecond * epochLen * maxBondFraction / 100
367          uint256 _maxBond = (_inflationPerSecond * _epochLen * _maxBondFraction) / 100;
368          maxBond = uint96(_maxBond);
369          effectiveBond = uint96(_maxBond);
370:     }

/// @audit combine into a `struct`: mapUnitIncentives,unitPoints
650      function _finalizeIncentivesForUnitId(uint256 epochNum, uint256 unitType, uint256 unitId) internal {
651          // Gets the overall amount of unit rewards for the unit's last epoch
652          // The pendingRelativeReward can be zero if the rewardUnitFraction was zero in the first place
653          // Note that if the rewardUnitFraction is set to zero at the end of epoch, the whole pending reward will be zero
654          // reward = (pendingRelativeReward * rewardUnitFraction) / 100
655          uint256 totalIncentives = mapUnitIncentives[unitType][unitId].pendingRelativeReward;
656          if (totalIncentives > 0) {
657              totalIncentives *= mapEpochTokenomics[epochNum].unitPoints[unitType].rewardUnitFraction;
658              // Add to the final reward for the last epoch
659              totalIncentives = mapUnitIncentives[unitType][unitId].reward + totalIncentives / 100;
660              mapUnitIncentives[unitType][unitId].reward = uint96(totalIncentives);
661              // Setting pending reward to zero
662              mapUnitIncentives[unitType][unitId].pendingRelativeReward = 0;
663          }
664  
665          // Add to the final top-up for the last epoch
666          totalIncentives = mapUnitIncentives[unitType][unitId].pendingRelativeTopUp;
667          // The pendingRelativeTopUp can be zero if the service owner did not stake enough veOLAS
668          // The topUpUnitFraction was checked before and if it were zero, pendingRelativeTopUp would be zero as well
669          if (totalIncentives > 0) {
670              // Summation of all the unit top-ups and total amount of top-ups per epoch
671              // topUp = (pendingRelativeTopUp * totalTopUpsOLAS * topUpUnitFraction) / (100 * sumUnitTopUpsOLAS)
672              totalIncentives *= mapEpochTokenomics[epochNum].epochPoint.totalTopUpsOLAS;
673              totalIncentives *= mapEpochTokenomics[epochNum].unitPoints[unitType].topUpUnitFraction;
674              uint256 sumUnitIncentives = uint256(mapEpochTokenomics[epochNum].unitPoints[unitType].sumUnitTopUpsOLAS) * 100;
675              totalIncentives = mapUnitIncentives[unitType][unitId].topUp + totalIncentives / sumUnitIncentives;
676              mapUnitIncentives[unitType][unitId].topUp = uint96(totalIncentives);
677              // Setting pending top-up to zero
678              mapUnitIncentives[unitType][unitId].pendingRelativeTopUp = 0;
679          }
680:     }

/// @audit combine into a `struct`: mapNewUnits,mapUnitIncentives,unitPoints
687      function _trackServiceDonations(address donator, uint256[] memory serviceIds, uint256[] memory amounts, uint256 curEpoch) internal {
688          // Component / agent registry addresses
689          address[] memory registries = new address[](2);
690          (registries[0], registries[1]) = (componentRegistry, agentRegistry);
691  
692          // Check all the unit fractions and identify those that need accounting of incentives
693          bool[] memory incentiveFlags = new bool[](4);
694          incentiveFlags[0] = (mapEpochTokenomics[curEpoch].unitPoints[0].rewardUnitFraction > 0);
695          incentiveFlags[1] = (mapEpochTokenomics[curEpoch].unitPoints[1].rewardUnitFraction > 0);
696          incentiveFlags[2] = (mapEpochTokenomics[curEpoch].unitPoints[0].topUpUnitFraction > 0);
697          incentiveFlags[3] = (mapEpochTokenomics[curEpoch].unitPoints[1].topUpUnitFraction > 0);
698  
699          // Get the number of services
700          uint256 numServices = serviceIds.length;
701          // Loop over service Ids to calculate their partial contributions
702          for (uint256 i = 0; i < numServices; ++i) {
703              // Check if the service owner or donator stakes enough OLAS for its components / agents to get a top-up
704              // If both component and agent owner top-up fractions are zero, there is no need to call external contract
705              // functions to check each service owner veOLAS balance
706              bool topUpEligible;
707              if (incentiveFlags[2] || incentiveFlags[3]) {
708                  address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
709                  topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
710                      IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
711              }
712  
713              // Loop over component and agent Ids
714              for (uint256 unitType = 0; unitType < 2; ++unitType) {
715                  // Get the number and set of units in the service
716                  (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).
717                      getUnitIdsOfService(IServiceRegistry.UnitType(unitType), serviceIds[i]);
718                  // Service has to be deployed at least once to be able to receive donations,
719                  // otherwise its components and agents are undefined
720                  if (numServiceUnits == 0) {
721                      revert ServiceNeverDeployed(serviceIds[i]);
722                  }
723                  // Record amounts data only if at least one incentive unit fraction is not zero
724                  if (incentiveFlags[unitType] || incentiveFlags[unitType + 2]) {
725                      // The amount has to be adjusted for the number of units in the service
726                      uint96 amount = uint96(amounts[i] / numServiceUnits);
727                      // Accumulate amounts for each unit Id
728                      for (uint256 j = 0; j < numServiceUnits; ++j) {
729                          // Get the last epoch number the incentives were accumulated for
730                          uint256 lastEpoch = mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch;
731                          // Check if there were no donations in previous epochs and set the current epoch
732                          if (lastEpoch == 0) {
733                              mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);
734                          } else if (lastEpoch < curEpoch) {
735                              // Finalize unit rewards and top-ups if there were pending ones from the previous epoch
736                              // Pending incentives are getting finalized during the next epoch the component / agent
737                              // receives donations. If this is not the case before claiming incentives, the finalization
738                              // happens in the accountOwnerIncentives() where the incentives are issued
739                              _finalizeIncentivesForUnitId(lastEpoch, unitType, serviceUnitIds[j]);
740                              // Change the last epoch number
741                              mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);
742                          }
743                          // Sum the relative amounts for the corresponding components / agents
744                          if (incentiveFlags[unitType]) {
745                              mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;
746                          }
747                          // If eligible, add relative top-up weights in the form of donation amounts.
748                          // These weights will represent the fraction of top-ups for each component / agent relative
749                          // to the overall amount of top-ups that must be allocated
750                          if (topUpEligible && incentiveFlags[unitType + 2]) {
751                              mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752                              mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;
753                          }
754                      }
755                  }
756  
757                  // Record new units and new unit owners
758                  for (uint256 j = 0; j < numServiceUnits; ++j) {
759                      // Check if the component / agent is used for the first time
760                      if (!mapNewUnits[unitType][serviceUnitIds[j]]) {
761                          mapNewUnits[unitType][serviceUnitIds[j]] = true;
762                          mapEpochTokenomics[curEpoch].unitPoints[unitType].numNewUnits++;
763                          // Check if the owner has introduced component / agent for the first time
764                          // This is done together with the new unit check, otherwise it could be just a new unit owner
765                          address unitOwner = IToken(registries[unitType]).ownerOf(serviceUnitIds[j]);
766                          if (!mapNewOwners[unitOwner]) {
767                              mapNewOwners[unitOwner] = true;
768                              mapEpochTokenomics[curEpoch].epochPoint.numNewOwners++;
769                          }
770                      }
771                  }
772              }
773          }
774:     }

```
*GitHub*: [264](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L264-L370), [650](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L650-L680), [687](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L687-L774)

```solidity
File: contracts/Treasury.sol

/// @audit combine into a `struct`: mapEnabledTokens,mapTokenReserves
212      function depositTokenForOLAS(address account, uint256 tokenAmount, address token, uint256 olasMintAmount) external {
213          // Check for the depository access
214          if (depository != msg.sender) {
215              revert ManagerOnly(msg.sender, depository);
216          }
217  
218          // Check if the token is authorized by the registry
219          if (!mapEnabledTokens[token]) {
220              revert UnauthorizedToken(token);
221          }
222  
223          // Increase the amount of LP token reserves
224          uint256 reserves = mapTokenReserves[token] + tokenAmount;
225          mapTokenReserves[token] = reserves;
226  
227          // Uniswap allowance implementation does not revert with the accurate message, need to check before the transfer is engaged
228          if (IToken(token).allowance(account, address(this)) < tokenAmount) {
229              revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);
230          }
231  
232          // Transfer tokens from account to treasury and add to the token treasury reserves
233          // We assume that authorized LP tokens in the protocol are safe as they are enabled via the governance
234          // UniswapV2ERC20 realization has a standard transferFrom() function that returns a boolean value
235          bool success = IToken(token).transferFrom(account, address(this), tokenAmount);
236          if (!success) {
237              revert TransferFailed(token, account, address(this), tokenAmount);
238          }
239  
240          // Mint specified number of OLAS tokens corresponding to tokens bonding deposit
241          // The olasMintAmount is guaranteed by the product supply limit, which is limited by the effectiveBond
242          IOLAS(olas).mint(msg.sender, olasMintAmount);
243  
244          emit DepositTokenFromAccount(account, token, tokenAmount, olasMintAmount);
245:     }

/// @audit combine into a `struct`: mapEnabledTokens,mapTokenReserves
313      function withdraw(address to, uint256 tokenAmount, address token) external returns (bool success) {
314          // Check for the contract ownership
315          if (msg.sender != owner) {
316              revert OwnerOnly(msg.sender, owner);
317          }
318  
319          // Check that the withdraw address is not treasury itself
320          if (to == address(this)) {
321              revert TransferFailed(token, address(this), to, tokenAmount);
322          }
323  
324          // Check for the zero withdraw amount
325          if (tokenAmount == 0) {
326              revert ZeroValue();
327          }
328  
329          // ETH address is taken separately, and all the LP tokens must be validated with corresponding token reserves
330          if (token == ETH_TOKEN_ADDRESS) {
331              uint256 amountOwned = ETHOwned;
332              // Check if treasury has enough amount of owned ETH
333              if (amountOwned >= tokenAmount) {
334                  // This branch is used to transfer ETH to a specified address
335                  amountOwned -= tokenAmount;
336                  ETHOwned = uint96(amountOwned);
337                  emit Withdraw(ETH_TOKEN_ADDRESS, to, tokenAmount);
338                  // Send ETH to the specified address
339                  (success, ) = to.call{value: tokenAmount}("");
340                  if (!success) {
341                      revert TransferFailed(ETH_TOKEN_ADDRESS, address(this), to, tokenAmount);
342                  }
343              } else {
344                  // Insufficient amount of treasury owned ETH
345                  revert LowerThan(tokenAmount, amountOwned);
346              }
347          } else {
348              // Only approved token reserves can be used for redemptions
349              if (!mapEnabledTokens[token]) {
350                  revert UnauthorizedToken(token);
351              }
352              // Decrease the global LP token reserves record
353              uint256 reserves = mapTokenReserves[token];
354              if (reserves >= tokenAmount) {
355                  reserves -= tokenAmount;
356                  mapTokenReserves[token] = reserves;
357  
358                  emit Withdraw(token, to, tokenAmount);
359                  // Transfer LP tokens
360                  // We assume that LP tokens enabled in the protocol are safe by default
361                  // UniswapV2ERC20 realization has a standard transfer() function
362                  success = IToken(token).transfer(to, tokenAmount);
363                  if (!success) {
364                      revert TransferFailed(token, address(this), to, tokenAmount);
365                  }
366              }  else {
367                  // Insufficient amount of LP tokens
368                  revert LowerThan(tokenAmount, reserves);
369              }
370          }
371:     }

/// @audit combine into a `struct`: mapEnabledTokens,mapTokenReserves
507      function disableToken(address token) external {
508          // Check for the contract ownership
509          if (msg.sender != owner) {
510              revert OwnerOnly(msg.sender, owner);
511          }
512  
513          if (mapEnabledTokens[token]) {
514              // The reserves of a token must be zero in order to disable it
515              if (mapTokenReserves[token] > 0) {
516                  revert NonZeroValue();
517              }
518              mapEnabledTokens[token] = false;
519              emit DisableToken(token);
520          }
521:     }

```
*GitHub*: [212](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L212-L245), [313](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L313-L371), [507](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L507-L521)



### [G&#x2011;10] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, or a function checks `msg.sender` some other way, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 56 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/OLAS.sol

43:      function changeOwner(address newOwner) external {

58:      function changeMinter(address newMinter) external {

75:      function mint(address account, uint256 amount) external {

```
*GitHub*: [43](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L43-L43), [58](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L58-L58), [75](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L75-L75)

```solidity
File: contracts/bridges/BridgedERC20.sol

30:      function changeOwner(address newOwner) external {

48:      function mint(address account, uint256 amount) external {

59:      function burn(uint256 amount) external {

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/BridgedERC20.sol#L30-L30), [48](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/BridgedERC20.sol#L48-L48), [59](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/BridgedERC20.sol#L59-L59)

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

81:      function changeRootGovernor(address newRootGovernor) external {

107:     function processMessageFromRoot(uint256 stateId, address rootMessageSender, bytes memory data) external override {

```
*GitHub*: [81](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L81-L81), [107](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L107-L107)

```solidity
File: contracts/bridges/HomeMediator.sol

81:      function changeForeignGovernor(address newForeignGovernor) external {

105:     function processMessageFromForeign(bytes memory data) external {

```
*GitHub*: [81](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L81-L81), [105](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L105-L105)

```solidity
File: contracts/multisigs/GuardCM.sol

154:     function changeGovernor(address newGovernor) external {

170:     function changeGovernorCheckProposalId(uint256 proposalId) external {

441      function setTargetSelectorChainIds(
442          address[] memory targets,
443          bytes4[] memory selectors,
444          uint256[] memory chainIds,
445          bool[] memory statuses
446:     ) external {

495      function setBridgeMediatorChainIds(
496          address[] memory bridgeMediatorL1s,
497          address[] memory bridgeMediatorL2s,
498          uint256[] memory chainIds
499:     ) external {

560:     function unpause() external {

```
*GitHub*: [154](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L154-L154), [170](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L170-L170), [441](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L441-L446), [495](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L495-L499), [560](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L560-L560)

```solidity
File: contracts/GenericManager.sol

20:      function changeOwner(address newOwner) external virtual {

36:      function pause() external virtual {

47:      function unpause() external virtual {

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L20-L20), [36](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L36-L36), [47](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericManager.sol#L47-L47)

```solidity
File: contracts/GenericRegistry.sol

37:      function changeOwner(address newOwner) external virtual {

54:      function changeManager(address newManager) external virtual {

78:      function setBaseURI(string memory bURI) external virtual {

```
*GitHub*: [37](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L37-L37), [54](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L54-L54), [78](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L78-L78)

```solidity
File: contracts/UnitRegistry.sol

49       function create(address unitOwner, bytes32 unitHash, uint32[] memory dependencies)
50           external virtual returns (uint256 unitId)
51:      {

121      function updateHash(address unitOwner, uint256 unitId, bytes32 unitHash) external virtual
122          returns (bool success)
123:     {

```
*GitHub*: [49](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L49-L51), [121](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L121-L123)

```solidity
File: contracts/Depository.sol

123:     function changeOwner(address newOwner) external {

143:     function changeManagers(address _tokenomics, address _treasury) external {

163:     function changeBondCalculator(address _bondCalculator) external {

183:     function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {

244:     function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {

356:     function redeem(uint256[] memory bondIds) external returns (uint256 payout) {

```
*GitHub*: [123](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L123-L123), [143](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L143-L143), [163](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L163-L163), [183](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L183-L183), [244](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L244-L244), [356](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L356-L356)

```solidity
File: contracts/Dispenser.sol

46:      function changeOwner(address newOwner) external {

64:      function changeManagers(address _tokenomics, address _treasury) external {

```
*GitHub*: [46](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L46-L46), [64](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L64-L64)

```solidity
File: contracts/DonatorBlacklist.sol

36:      function changeOwner(address newOwner) external {

56:      function setDonatorsStatuses(address[] memory accounts, bool[] memory statuses) external returns (bool success) {

```
*GitHub*: [36](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L36-L36), [56](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L56-L56)

```solidity
File: contracts/Tokenomics.sol

384:     function changeTokenomicsImplementation(address implementation) external {

404:     function changeOwner(address newOwner) external {

423:     function changeManagers(address _treasury, address _depository, address _dispenser) external {

450:     function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {

474:     function changeDonatorBlacklist(address _donatorBlacklist) external {

497      function changeTokenomicsParameters(
498          uint256 _devsPerCapital,
499          uint256 _codePerDev,
500          uint256 _epsilonRate,
501          uint256 _epochLen,
502          uint256 _veOLASThreshold
503      ) external
504:     {

562      function changeIncentiveFractions(
563          uint256 _rewardComponentFraction,
564          uint256 _rewardAgentFraction,
565          uint256 _maxBondFraction,
566          uint256 _topUpComponentFraction,
567          uint256 _topUpAgentFraction
568      ) external
569:     {

609:     function reserveAmountForBondProgram(uint256 amount) external returns (bool success) {

630:     function refundFromBondProgram(uint256 amount) external {

788      function trackServiceDonations(
789          address donator,
790          uint256[] memory serviceIds,
791          uint256[] memory amounts,
792          uint256 donationETH
793:     ) external {

1085     function accountOwnerIncentives(address account, uint256[] memory unitTypes, uint256[] memory unitIds) external
1086         returns (uint256 reward, uint256 topUp)
1087:    {

```
*GitHub*: [384](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L384-L384), [404](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L404-L404), [423](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L423-L423), [450](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L450-L450), [474](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L474-L474), [497](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L497-L504), [562](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L562-L569), [609](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L609-L609), [630](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L630-L630), [788](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L788-L793), [1085](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1085-L1087)

```solidity
File: contracts/Treasury.sol

137:     function changeOwner(address newOwner) external {

156:     function changeManagers(address _tokenomics, address _depository, address _dispenser) external {

182:     function changeMinAcceptedETH(uint256 _minAcceptedETH) external {

212:     function depositTokenForOLAS(address account, uint256 tokenAmount, address token, uint256 olasMintAmount) external {

313:     function withdraw(address to, uint256 tokenAmount, address token) external returns (bool success) {

387      function withdrawToAccount(address account, uint256 accountRewards, uint256 accountTopUps) external
388          returns (bool success)
389:     {

428:     function rebalanceTreasury(uint256 treasuryRewards) external returns (bool success) {

466:     function drainServiceSlashedFunds() external returns (uint256 amount) {

487:     function enableToken(address token) external {

507:     function disableToken(address token) external {

531:     function pause() external {

542:     function unpause() external {

```
*GitHub*: [137](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L137-L137), [156](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L156-L156), [182](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L182-L182), [212](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L212-L212), [313](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L313-L313), [387](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L387-L389), [428](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L428-L428), [466](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L466-L466), [487](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L487-L487), [507](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L507-L507), [531](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L531-L531), [542](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L542-L542)

</details>





### [G&#x2011;11] `internal` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls. The inliner can do it only for 'simple' cases:
> Now to get back to the point why we require the routine to be simple: As soon as you do more complicated things like for example branching, calling external contracts, the Common Subexpression Eliminator cannot re-construct the code anymore or does not do full symbolic evaluation of the expressions. 

https://soliditylang.org/blog/2021/03/02/saving-gas-with-simple-inliner/

Therefore, the instances below contain branching or use op-codes with side-effects

*There are 2 instances of this issue:*

```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

70       function _processMessageFromRoot(
71           uint256 /* stateId */,
72           address sender,
73           bytes memory message
74:      ) internal override validateSender(sender) {

```
*GitHub*: [70](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L70-L74)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

72:      function _processMessageFromChild(bytes memory message) internal override {

```
*GitHub*: [72](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L72-L72)



### [G&#x2011;12] Assembly: Use scratch space when building emitted events with two data arguments
Using the [scratch space](https://gist.github.com/IllIllI000/87c4f03139fa03780fa548b8e4b02b5b) for more than one, but at most two words worth of data (non-indexed arguments) will save gas over needing Solidity's abi memory expansion used for emitting normally.

*There are 10 instances of this issue:*

```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

84:          emit FxWithdrawERC20(rootToken, childToken, from, to, amount);

107:         emit FxDepositERC20(childToken, rootToken, msg.sender, to, amount);

```
*GitHub*: [84](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L84-L84), [107](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L107-L107)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

79:          emit FxDepositERC20(childToken, rootToken, from, to, amount);

106:         emit FxWithdrawERC20(rootToken, childToken, msg.sender, to, amount);

```
*GitHub*: [79](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L79-L79), [106](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L106-L106)

```solidity
File: contracts/veOLAS.sol

368:         emit Supply(supplyBefore, supplyAfter);

530:         emit Withdraw(msg.sender, amount, block.timestamp);

531:         emit Supply(supplyBefore, supplyAfter);

```
*GitHub*: [368](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L368-L368), [530](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L530-L530), [531](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L531-L531)

```solidity
File: contracts/Dispenser.sol

112:         emit IncentivesClaimed(msg.sender, reward, topUp);

```
*GitHub*: [112](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L112-L112)

```solidity
File: contracts/Treasury.sol

244:         emit DepositTokenFromAccount(account, token, tokenAmount, olasMintAmount);

452:                 emit UpdateTreasuryBalances(amountETHOwned, amountETHFromServices);

```
*GitHub*: [244](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L244-L244), [452](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L452-L452)



### [G&#x2011;13] `>=` costs less gas than `>`
The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, [which saves **3 gas**](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde). If `<` is being used, the condition can be inverted. In cases where a for-loop is being used, one can count down rather than up, in order to use the optimal operator

*There are 51 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/OLAS.sol

108:             for (uint256 i = 0; i < numYears; ++i) {

```
*GitHub*: [108](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L108-L108)

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

125:         for (uint256 i = 0; i < dataLength;) {

153:             for (uint256 j = 0; j < payloadLength; ++j) {

```
*GitHub*: [125](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L125-L125), [153](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L153-L153)

```solidity
File: contracts/bridges/HomeMediator.sol

125:         for (uint256 i = 0; i < dataLength;) {

153:             for (uint256 j = 0; j < payloadLength; ++j) {

```
*GitHub*: [125](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L125-L125), [153](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L153-L153)

```solidity
File: contracts/multisigs/GuardCM.sol

212:         for (uint256 i = 0; i < data.length;) {

237:             for (uint256 j = 0; j < payloadLength; ++j) {

273:             for (uint256 i = 0; i < payload.length; ++i) {

292:             for (uint256 i = 0; i < bridgePayload.length; ++i) {

318:             for (uint256 i = 0; i < payload.length; ++i) {

340:         for (uint256 i = 0; i < payload.length; ++i) {

360:         for (uint i = 0; i < targets.length; ++i) {

458:         for (uint256 i = 0; i < targets.length; ++i) {

511:         for (uint256 i = 0; i < chainIds.length; ++i) {

```
*GitHub*: [212](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L212-L212), [237](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L237-L237), [273](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L273-L273), [292](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L292-L292), [318](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L318-L318), [340](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L340-L340), [360](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L360-L360), [458](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L458-L458), [511](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L511-L511)

```solidity
File: contracts/veOLAS.sol

232:             for (uint256 i = 0; i < 255; ++i) {

561:         for (uint256 i = 0; i < 128; ++i) {

693:         for (uint256 i = 0; i < 255; ++i) {

```
*GitHub*: [232](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L232-L232), [561](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L561-L561), [693](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L693-L693)

```solidity
File: contracts/AgentRegistry.sol

40:          for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {

```
*GitHub*: [40](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L40-L40)

```solidity
File: contracts/ComponentRegistry.sol

29:          for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {

```
*GitHub*: [29](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L29-L29)

```solidity
File: contracts/UnitRegistry.sol

98:              for (uint256 i = 0; i < numSubComponents; ++i) {

211:         for (uint32 i = 0; i < numUnits; ++i) {

227:         for (counter = 0; counter < maxNumComponents; ++counter) {

234:             for (uint32 i = 0; i < numUnits; ++i) {

236:                 for (; processedComponents[i] < numComponents[i]; ++processedComponents[i]) {

262:         for (uint32 i = 0; i < counter; ++i) {

```
*GitHub*: [98](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L98-L98), [211](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L211-L211), [227](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L227-L227), [234](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L234-L234), [236](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L236-L236), [262](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L262-L262)

```solidity
File: contracts/multisigs/GnosisSafeMultisig.sol

79:                  for (uint256 i = 0; i < payloadLength; ++i) {

```
*GitHub*: [79](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L79-L79)

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

113:             for (uint256 i = 0; i < payloadLength; ++i) {

139:         for (uint256 i = 0; i < numOwners; ++i) {

```
*GitHub*: [113](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L113-L113), [139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L139-L139)

```solidity
File: contracts/Depository.sol

255:         for (uint256 i = 0; i < numProducts; ++i) {

274:         for (uint256 i = 0; i < numClosedProducts; ++i) {

357:         for (uint256 i = 0; i < bondIds.length; ++i) {

402:         for (uint256 i = 0; i < numProducts; ++i) {

413:         for (uint256 i = 0; i < numProducts; ++i) {

448:         for (uint256 i = 0; i < numBonds; ++i) {

468:         for (uint256 i = 0; i < numBonds; ++i) {

```
*GitHub*: [255](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L255-L255), [274](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L274-L274), [357](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L357-L357), [402](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L402-L402), [413](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L413-L413), [448](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L448-L448), [468](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L468-L468)

```solidity
File: contracts/DonatorBlacklist.sol

67:          for (uint256 i = 0; i < accounts.length; ++i) {

```
*GitHub*: [67](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L67-L67)

```solidity
File: contracts/Tokenomics.sol

702:         for (uint256 i = 0; i < numServices; ++i) {

714:             for (uint256 unitType = 0; unitType < 2; ++unitType) {

728:                     for (uint256 j = 0; j < numServiceUnits; ++j) {

758:                 for (uint256 j = 0; j < numServiceUnits; ++j) {

808:         for (uint256 i = 0; i < numServices; ++i) {

978:             for (uint256 i = 0; i < 2; ++i) {

1104:        for (uint256 i = 0; i < 2; ++i) {

1110:        for (uint256 i = 0; i < unitIds.length; ++i) {

1132:        for (uint256 i = 0; i < unitIds.length; ++i) {

1174:        for (uint256 i = 0; i < 2; ++i) {

1180:        for (uint256 i = 0; i < unitIds.length; ++i) {

1202:        for (uint256 i = 0; i < unitIds.length; ++i) {

```
*GitHub*: [702](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L702-L702), [714](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L714-L714), [728](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L728-L728), [758](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L758-L758), [808](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L808-L808), [978](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L978-L978), [1104](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1104-L1104), [1110](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1110-L1110), [1132](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1132-L1132), [1174](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1174-L1174), [1180](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1180-L1180), [1202](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1202-L1202)

```solidity
File: contracts/TokenomicsConstants.sol

55:              for (uint256 i = 0; i < numYears; ++i) {

92:              for (uint256 i = 1; i < numYears; ++i) {

```
*GitHub*: [55](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L55-L55), [92](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L92-L92)

```solidity
File: contracts/Treasury.sol

276:         for (uint256 i = 0; i < numServices; ++i) {

```
*GitHub*: [276](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L276-L276)

</details>





### [G&#x2011;14] Consider using `ERC721A` over `ERC721`
[ERC721A](https://www.azuki.com/erc721a) is much more gas-efficient for minting multiple NFTs in the same transaction

*There are 3 instances of this issue:*

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
File: contracts/UnitRegistry.sol

8:   abstract contract UnitRegistry is GenericRegistry {

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L8-L8)



### [G&#x2011;15] Consider using `solady`'s token contracts to save gas
They're written using heavily-optimized assembly, and will your users a lot of gas

*There are 3 instances of this issue:*

```solidity
File: contracts/AgentRegistry.sol

/// @audit ERC721
9    contract AgentRegistry is UnitRegistry {
10:      // Component registry

```
*GitHub*: [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L9-L10)

```solidity
File: contracts/ComponentRegistry.sol

/// @audit ERC721
8    contract ComponentRegistry is UnitRegistry {
9:       // Component registry version number

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L8-L9)

```solidity
File: contracts/UnitRegistry.sol

/// @audit ERC721
8:   abstract contract UnitRegistry is GenericRegistry {

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L8-L8)



### [G&#x2011;16] Consider using solady's `FixedPointMathLib`
Saves gas, and works to avoid unnecessary [overflows](https://github.com/Vectorized/solady/blob/6cce088e69d6e46671f2f622318102bd5db77a65/src/utils/FixedPointMathLib.sol#L267).

*There are 23 instances of this issue:*

```solidity
File: contracts/OLAS.sol

101:         uint256 numYears = (block.timestamp - timeLaunch) / oneYear;

109:                 supplyCap += (supplyCap * maxMintCapFraction) / 100;

```
*GitHub*: [101](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L101-L101), [109](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L109-L109)

```solidity
File: contracts/veOLAS.sol

225:             block_slope = (1e18 * uint256(block.number - lastPoint.blockNumber)) / uint256(block.timestamp - lastPoint.ts);

258:                 lastPoint.blockNumber = initialPoint.blockNumber + uint64((block_slope * uint256(tStep - initialPoint.ts)) / 1e18);

433:             unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;

487:             unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;

565:             uint256 mid = (minPointNumber + maxPointNumber + 1) / 2;

664:             blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

```
*GitHub*: [225](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L225-L225), [258](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L258-L258), [433](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L433-L433), [487](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L487-L487), [565](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L565-L565), [664](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L664-L664)

```solidity
File: contracts/GenericBondCalculator.sol

89:                  priceLP = (reserve1 * 1e18) / totalSupply;

```
*GitHub*: [89](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L89-L89)

```solidity
File: contracts/Tokenomics.sol

367:         uint256 _maxBond = (_inflationPerSecond * _epochLen * _maxBondFraction) / 100;

915:         incentives[1] = (incentives[0] * tp.epochPoint.rewardTreasuryFraction) / 100;

917:         incentives[2] = (incentives[0] * tp.unitPoints[0].rewardUnitFraction) / 100;

918:         incentives[3] = (incentives[0] * tp.unitPoints[1].rewardUnitFraction) / 100;

925:         uint256 numYears = (block.timestamp - timeLaunch) / ONE_YEAR;

951:         incentives[4] = (inflationPerEpoch * tp.epochPoint.maxBondFraction) / 100;

1010:        numYears = (block.timestamp + curEpochLen - timeLaunch) / ONE_YEAR;

1023:            curMaxBond = (inflationPerEpoch * nextEpochPoint.epochPoint.maxBondFraction) / 100;

1030:            curMaxBond = (curEpochLen * curInflationPerSecond * nextEpochPoint.epochPoint.maxBondFraction) / 100;

1054:        incentives[5] = (inflationPerEpoch * tp.unitPoints[0].topUpUnitFraction) / 100;

1056:        incentives[6] = (inflationPerEpoch * tp.unitPoints[1].topUpUnitFraction) / 100;

```
*GitHub*: [367](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L367-L367), [915](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L915-L915), [917](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L917-L917), [918](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L918-L918), [925](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L925-L925), [951](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L951-L951), [1010](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1010-L1010), [1023](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1023-L1023), [1030](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1030-L1030), [1054](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1054-L1054), [1056](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1056-L1056)

```solidity
File: contracts/TokenomicsConstants.sol

56:                  supplyCap += (supplyCap * maxMintCapFraction) / 100;

93:                  supplyCap += (supplyCap * maxMintCapFraction) / 100;

97:              inflationAmount = (supplyCap * maxMintCapFraction) / 100;

```
*GitHub*: [56](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L56-L56), [93](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L93-L93), [97](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L97-L97)



### [G&#x2011;17] Initializers can be marked `payable`
Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. An initializer can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.

*There is one instance of this issue:*

```solidity
File: contracts/Tokenomics.sol

264      function initializeTokenomics(
265          address _olas,
266          address _treasury,
267          address _depository,
268          address _dispenser,
269          address _ve,
270          uint256 _epochLen,
271          address _componentRegistry,
272          address _agentRegistry,
273          address _serviceRegistry,
274          address _donatorBlacklist
275      ) external
276:     {

```
*GitHub*: [264](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L264-L276)



### [G&#x2011;18] Memory-safe annotation missing
Use `assembly ("memory-safe") { ... }` for the assembly blocks below since they dont't read or modify memory, and therefore are [memory-safe](https://docs.soliditylang.org/en/latest/assembly.html#memory-safety). This will help the optimizer to create more optimal gas-efficient code. Use the comment variant if prior to Solidity version 0.8.13

*There are 6 instances of this issue:*

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

129              // solhint-disable-next-line no-inline-assembly
130              assembly {
131                  // First 20 bytes is the address (160 bits)
132                  i := add(i, 20)
133                  target := mload(add(data, i))
134                  // Offset the data by 12 bytes of value (96 bits)
135                  i := add(i, 12)
136                  value := mload(add(data, i))
137                  // Offset the data by 4 bytes of payload length (32 bits)
138                  i := add(i, 4)
139                  payloadLength := mload(add(data, i))
140:             }

```
*GitHub*: [129](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L129-L140)

```solidity
File: contracts/bridges/HomeMediator.sol

129              // solhint-disable-next-line no-inline-assembly
130              assembly {
131                  // First 20 bytes is the address (160 bits)
132                  i := add(i, 20)
133                  target := mload(add(data, i))
134                  // Offset the data by 12 bytes of value (96 bits)
135                  i := add(i, 12)
136                  value := mload(add(data, i))
137                  // Offset the data by 4 bytes of payload length (32 bits)
138                  i := add(i, 4)
139                  payloadLength := mload(add(data, i))
140:             }

```
*GitHub*: [129](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L129-L140)

```solidity
File: contracts/multisigs/GuardCM.sol

215              // solhint-disable-next-line no-inline-assembly
216              assembly {
217                  // First 20 bytes is the address (160 bits)
218                  i := add(i, 20)
219                  target := mload(add(data, i))
220                  // Offset the data by 12 bytes of value (96 bits) and by 4 bytes of payload length (32 bits)
221                  i := add(i, 16)
222                  payloadLength := mload(add(data, i))
223:             }

```
*GitHub*: [215](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L215-L223)

```solidity
File: contracts/multisigs/GnosisSafeMultisig.sol

56               // Read the first 144 bytes of data
57               assembly {
58                   // Read all the addresses first (80 bytes)
59                   let offset := 20
60                   to := mload(add(data, offset))
61                   offset := add(offset, 20)
62                   fallbackHandler := mload(add(data, offset))
63                   offset := add(offset, 20)
64                   paymentToken := mload(add(data, offset))
65                   offset := add(offset, 20)
66                   paymentReceiver := mload(add(data, offset))
67   
68                   // Read all the uints (64 more bytes, a total of 144 bytes)
69                   offset := add(offset, 32)
70                   payment := mload(add(data, offset))
71                   offset := add(offset, 32)
72                   nonce := mload(add(data, offset))
73:              }

```
*GitHub*: [56](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L56-L73)

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

97           // Read the proxy multisig address (20 bytes) and the multisig-related data
98           assembly {
99               multisig := mload(add(data, DEFAULT_DATA_LENGTH))
100:         }

```
*GitHub*: [97](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L97-L100)

```solidity
File: contracts/TokenomicsProxy.sol

55       fallback() external {
56           assembly {
57               let tokenomics := sload(PROXY_TOKENOMICS)
58               // Otherwise continue with the delegatecall to the tokenomics implementation
59               calldatacopy(0, 0, calldatasize())
60               let success := delegatecall(gas(), tokenomics, 0, calldatasize(), 0, 0)
61               returndatacopy(0, 0, returndatasize())
62               if eq(success, 0) {
63                   revert(0, returndatasize())
64               }
65               return(0, returndatasize())
66:          }

```
*GitHub*: [55](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsProxy.sol#L55-L66)



### [G&#x2011;19] Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`
Saves a storage slot for each of the removed mappings (i.e. this finding is not about runtime savings). The instances below refer to both mappings using the same key in the same function, so the mappings are related.

*There are 4 instances of this issue:*

```solidity
File: contracts/UnitRegistry.sol

/// @audit combine into a `struct`: mapSubComponents,mapUnits
8:   abstract contract UnitRegistry is GenericRegistry {

```
*GitHub*: [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L8-L8)

```solidity
File: contracts/Tokenomics.sol

/// @audit combine into a `struct`: mapUnitIncentives,unitPoints
/// @audit combine into a `struct`: mapEpochTokenomics,unitPoints
/// @audit combine into a `struct`: mapNewUnits,mapUnitIncentives,unitPoints
118: contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {

```
*GitHub*: [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L118-L118), [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L118-L118), [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L118-L118)



### [G&#x2011;20] Multiple accesses of a `memory`/`calldata` array should use a local variable cache
The instances below point to the second+ access of a value inside a `memory`/`calldata` array, within a function. Caching avoids recalculating the array offsets into `memory`/`calldata`

*There are 84 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/multisigs/GuardCM.sol

/// @audit targets...[]
376:                 _verifyData(targets[i], callDatas[i], block.chainid);

/// @audit targets...[]
476:             uint256 targetSelectorChainId = uint256(uint160(targets[i]));

/// @audit selectors...[]
478:             targetSelectorChainId |= uint256(uint32(selectors[i])) << 160;

/// @audit chainIds...[]
480:             targetSelectorChainId |= chainIds[i] << 192;

/// @audit bridgeMediatorL2s...[]
525:             uint256 bridgeMediatorL2ChainId = uint256(uint160(bridgeMediatorL2s[i]));

/// @audit bridgeMediatorL1s...[]
528:             mapBridgeMediatorL1L2ChainIds[bridgeMediatorL1s[i]] = bridgeMediatorL2ChainId;

```
*GitHub*: [376](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L376-L376), [476](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L476-L476), [478](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L478-L478), [480](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L480-L480), [525](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L525-L525), [528](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L528-L528)

```solidity
File: contracts/AgentRegistry.sol

/// @audit dependencies...[]
44:              lastId = dependencies[iDep];

```
*GitHub*: [44](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L44-L44)

```solidity
File: contracts/ComponentRegistry.sol

/// @audit dependencies...[]
33:              lastId = dependencies[iDep];

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L33-L33)

```solidity
File: contracts/UnitRegistry.sol

/// @audit components...[]
214:             numComponents[i] = uint32(components[i].length);

/// @audit numComponents...[]
215:             maxNumComponents += numComponents[i];

```
*GitHub*: [214](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L214-L214), [215](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L215-L215)

```solidity
File: contracts/Depository.sol

/// @audit productIds...[]
266:                 ids[numClosedProducts] = productIds[i];

/// @audit bondIds...[]
360:             bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;

/// @audit bondIds...[]
364:                 revert BondNotRedeemable(bondIds[i]);

/// @audit bondIds...[]
368:             if (mapUserBonds[bondIds[i]].account != msg.sender) {

/// @audit bondIds...[]
369:                 revert OwnerOnly(msg.sender, mapUserBonds[bondIds[i]].account);

/// @audit bondIds...[]
376:             uint256 productId = mapUserBonds[bondIds[i]].productId;

/// @audit bondIds...[]
379:             delete mapUserBonds[bondIds[i]];

/// @audit bondIds...[]
380:             emit RedeemBond(productId, msg.sender, bondIds[i]);

```
*GitHub*: [266](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L266-L266), [360](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L360-L360), [364](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L364-L364), [368](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L368-L368), [369](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L369-L369), [376](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L376-L376), [379](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L379-L379), [380](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L380-L380)

```solidity
File: contracts/DonatorBlacklist.sol

/// @audit accounts...[]
73:              mapBlacklistedDonators[accounts[i]] = statuses[i];

/// @audit accounts...[]
/// @audit statuses...[]
74:              emit DonatorBlacklistStatus(accounts[i], statuses[i]);

```
*GitHub*: [73](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L73-L73), [74](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L74-L74), [74](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/DonatorBlacklist.sol#L74-L74)

```solidity
File: contracts/Tokenomics.sol

/// @audit incentiveFlags...[]
/// @audit incentiveFlags...[]
707:             if (incentiveFlags[2] || incentiveFlags[3]) {

/// @audit serviceIds...[]
721:                     revert ServiceNeverDeployed(serviceIds[i]);

/// @audit serviceUnitIds...[]
733:                             mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);

/// @audit serviceUnitIds...[]
739:                             _finalizeIncentivesForUnitId(lastEpoch, unitType, serviceUnitIds[j]);

/// @audit serviceUnitIds...[]
741:                             mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);

/// @audit serviceUnitIds...[]
745:                             mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

/// @audit serviceUnitIds...[]
751:                             mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;

/// @audit serviceUnitIds...[]
765:                         address unitOwner = IToken(registries[unitType]).ownerOf(serviceUnitIds[j]);

/// @audit incentives...[]
915:         incentives[1] = (incentives[0] * tp.epochPoint.rewardTreasuryFraction) / 100;

/// @audit incentives...[]
917:         incentives[2] = (incentives[0] * tp.unitPoints[0].rewardUnitFraction) / 100;

/// @audit incentives...[]
918:         incentives[3] = (incentives[0] * tp.unitPoints[1].rewardUnitFraction) / 100;

/// @audit incentives...[]
963:         if (incentives[4] > curMaxBond) {

/// @audit incentives...[]
/// @audit incentives...[]
965:             incentives[4] = effectiveBond + incentives[4] - curMaxBond;

/// @audit incentives...[]
966:             effectiveBond = uint96(incentives[4]);

/// @audit incentives...[]
1041:        if (incentives[0] > 0) {

/// @audit incentives...[]
1043:            uint256 idf = _calculateIDF(incentives[1], tp.epochPoint.numNewOwners);

/// @audit incentives...[]
/// @audit incentives...[]
1052:        uint256 accountRewards = incentives[2] + incentives[3];

/// @audit incentives...[]
/// @audit incentives...[]
1060:        uint256 accountTopUps = incentives[5] + incentives[6];

/// @audit incentives...[]
/// @audit incentives...[]
1063:        if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) {

/// @audit incentives...[]
1065:            emit EpochSettled(eCounter, incentives[1], accountRewards, accountTopUps);

/// @audit unitTypes...[]
/// @audit unitTypes...[]
1117:            if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {

/// @audit unitTypes...[]
1118:                revert WrongUnitId(unitIds[i], unitTypes[i]);

/// @audit unitTypes...[]
/// @audit unitIds...[]
1120:            lastIds[unitTypes[i]] = unitIds[i];

/// @audit unitTypes...[]
/// @audit unitIds...[]
1123:            address unitOwner = IToken(registries[unitTypes[i]]).ownerOf(unitIds[i]);

/// @audit unitTypes...[]
/// @audit unitIds...[]
1139:                _finalizeIncentivesForUnitId(lastEpoch, unitTypes[i], unitIds[i]);

/// @audit unitTypes...[]
/// @audit unitIds...[]
1141:                mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch = 0;

/// @audit unitTypes...[]
/// @audit unitIds...[]
1145:            reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

/// @audit unitTypes...[]
/// @audit unitIds...[]
1146:            mapUnitIncentives[unitTypes[i]][unitIds[i]].reward = 0;

/// @audit unitTypes...[]
/// @audit unitIds...[]
1148:            topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

/// @audit unitTypes...[]
/// @audit unitIds...[]
1149:            mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp = 0;

/// @audit unitTypes...[]
/// @audit unitTypes...[]
1187:            if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {

/// @audit unitTypes...[]
1188:                revert WrongUnitId(unitIds[i], unitTypes[i]);

/// @audit unitTypes...[]
/// @audit unitIds...[]
1190:            lastIds[unitTypes[i]] = unitIds[i];

/// @audit unitTypes...[]
/// @audit unitIds...[]
1193:            address unitOwner = IToken(registries[unitTypes[i]]).ownerOf(unitIds[i]);

/// @audit unitTypes...[]
/// @audit unitIds...[]
1209:                uint256 totalIncentives = mapUnitIncentives[unitTypes[i]][unitIds[i]].pendingRelativeReward;

/// @audit unitTypes...[]
1211:                    totalIncentives *= mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].rewardUnitFraction;

/// @audit unitTypes...[]
/// @audit unitIds...[]
1216:                totalIncentives = mapUnitIncentives[unitTypes[i]][unitIds[i]].pendingRelativeTopUp;

/// @audit unitTypes...[]
1221:                    totalIncentives *= mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].topUpUnitFraction;

/// @audit unitTypes...[]
1222:                    uint256 sumUnitIncentives = uint256(mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].sumUnitTopUpsOLAS) * 100;

/// @audit unitTypes...[]
/// @audit unitIds...[]
1229:            reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

/// @audit unitTypes...[]
/// @audit unitIds...[]
1231:            topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

```
*GitHub*: [707](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L707-L707), [707](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L707-L707), [721](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L721-L721), [733](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L733-L733), [739](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L739-L739), [741](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L741-L741), [745](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L745-L745), [751](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L751-L751), [765](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L765-L765), [915](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L915-L915), [917](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L917-L917), [918](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L918-L918), [963](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L963-L963), [965](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L965-L965), [965](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L965-L965), [966](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L966-L966), [1041](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1041-L1041), [1043](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1043-L1043), [1052](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1052-L1052), [1052](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1052-L1052), [1060](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1060-L1060), [1060](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1060-L1060), [1063](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1063-L1063), [1063](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1063-L1063), [1065](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1065-L1065), [1117](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1117-L1117), [1117](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1117-L1117), [1118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1118-L1118), [1120](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1120-L1120), [1120](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1120-L1120), [1123](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1123-L1123), [1123](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1123-L1123), [1139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1139-L1139), [1139](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1139-L1139), [1141](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1141-L1141), [1141](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1141-L1141), [1145](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1145-L1145), [1145](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1145-L1145), [1146](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1146-L1146), [1146](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1146-L1146), [1148](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1148-L1148), [1148](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1148-L1148), [1149](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1149-L1149), [1149](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1149-L1149), [1187](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1187-L1187), [1187](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1187-L1187), [1188](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1188-L1188), [1190](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1190-L1190), [1190](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1190-L1190), [1193](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1193-L1193), [1193](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1193-L1193), [1209](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1209-L1209), [1209](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1209-L1209), [1211](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1211-L1211), [1216](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1216-L1216), [1216](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1216-L1216), [1221](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1221-L1221), [1222](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1222-L1222), [1229](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1229-L1229), [1229](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1229-L1229), [1231](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1231-L1231), [1231](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1231-L1231)

```solidity
File: contracts/Treasury.sol

/// @audit amounts...[]
280:             totalAmount += amounts[i];

```
*GitHub*: [280](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L280-L280)

</details>





### [G&#x2011;21] Multiple accesses of a storage array should use a local variable cache
The instances below point to the second+ access of a value inside a storage array, within a function. Caching an array index value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the array's keccak256 hash (`Gkeccak256` - **30 gas**) and that calculation's associated stack operations.

*There are 2 instances of this issue:*

```solidity
File: contracts/veOLAS.sol

/// @audit mapUserPoints...[]
148:             pv = mapUserPoints[account][lastPointNumber - 1];

/// @audit mapUserPoints...[]
596:             PointVoting memory uPoint = mapUserPoints[account][pointNumber - 1];

```
*GitHub*: [148](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L148-L148), [596](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L596-L596)



### [G&#x2011;22] Reduce deployment costs by tweaking contracts' metadata
See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

*There are 23 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/GovernorOLAS.sol

15:  contract GovernorOLAS is Governor, GovernorSettings, GovernorCompatibilityBravo, GovernorVotes, GovernorVotesQuorumFraction, GovernorTimelockControl {

```
*GitHub*: [15](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L15-L15)

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





### [G&#x2011;23] Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the **3 gas** the extra stack assignment would spend

*There are 32 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/OLAS.sol

92:          uint256 remainder = inflationRemainder();

99:          uint256 _totalSupply = totalSupply;

```
*GitHub*: [92](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L92-L92), [99](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L99-L99)

```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

76:          (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));

79:          bool success = IERC20(childToken).transfer(to, amount);

97:          bytes memory message = abi.encode(msg.sender, to, amount);

102:         bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

```
*GitHub*: [76](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L76-L76), [79](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79-L79), [97](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L97-L97), [102](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102-L102)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

74:          (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));

93:          bytes memory message = abi.encode(msg.sender, to, amount);

98:          bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);

```
*GitHub*: [74](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L74-L74), [93](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L93-L93), [98](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L98)

```solidity
File: contracts/multisigs/GuardCM.sol

297:             (bytes memory l2Message) = abi.decode(bridgePayload, (bytes));

323:             (address fxGovernorTunnel, bytes memory l2Message) = abi.decode(payload, (address, bytes));

```
*GitHub*: [297](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L297-L297), [323](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L323-L323)

```solidity
File: contracts/veOLAS.sol

677:         (, uint256 blockTime) = _getBlockTime(blockNumber);

753:         (PointVoting memory sPoint, uint256 blockTime) = _getBlockTime(blockNumber);

```
*GitHub*: [677](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L677-L677), [753](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L753-L753)

```solidity
File: contracts/wveOLAS.sol

195:         uint256 userNumPoints = IVEOLAS(ve).getNumUserPoints(account);

265:         uint256 numPoints = IVEOLAS(ve).totalNumPoints();

```
*GitHub*: [195](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L195-L195), [265](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L265-L265)

```solidity
File: contracts/multisigs/GnosisSafeMultisig.sol

98           (address to, address fallbackHandler, address paymentToken, address payable paymentReceiver, uint256 payment,
99:              uint256 nonce, bytes memory payload) = _parseData(data);

102          bytes memory safeParams = abi.encodeWithSelector(GNOSIS_SAFE_SETUP_SELECTOR, owners, threshold,
103:             to, payload, fallbackHandler, paymentToken, payment, paymentReceiver);

```
*GitHub*: [98](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L98-L99), [102](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L102-L103)

```solidity
File: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

103:         bytes32 multisigProxyHash = keccak256(multisig.code);

118:             (bool success, ) = multisig.call(payload);

```
*GitHub*: [103](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L103-L103), [118](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L118-L118)

```solidity
File: contracts/GenericBondCalculator.sol

76:              address token1 = pair.token1();

```
*GitHub*: [76](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L76-L76)

```solidity
File: contracts/Tokenomics.sol

325:         uint256 zeroYearSecondsLeft = uint32(_timeLaunch + ONE_YEAR - block.timestamp);

674:             uint256 sumUnitIncentives = uint256(mapEpochTokenomics[epochNum].unitPoints[unitType].sumUnitTopUpsOLAS) * 100;

841:         UD60x18 codeUnits = UD60x18.wrap(codePerDev);

846:         UD60x18 fpDevsPerCapital = UD60x18.wrap(devsPerCapital);

848:         UD60x18 fpNumNewOwners = convert(numNewOwners);

1052:        uint256 accountRewards = incentives[2] + incentives[3];

1060:        uint256 accountTopUps = incentives[5] + incentives[6];

```
*GitHub*: [325](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L325-L325), [674](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L674-L674), [841](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L841-L841), [846](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L846-L846), [848](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L848-L848), [1052](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1052-L1052), [1060](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1060-L1060)

```solidity
File: contracts/TokenomicsConstants.sol

33               uint96[10] memory supplyCaps = [
34                   529_659_000_00e16,
35                   569_913_084_00e16,
36                   641_152_219_50e16,
37                   708_500_141_72e16,
38                   771_039_876_00e16,
39                   828_233_282_97e16,
40                   879_860_040_11e16,
41                   925_948_139_65e16,
42                   966_706_331_40e16,
43                   1_000_000_000e18
44:              ];

70               uint88[10] memory inflationAmounts = [
71                   3_159_000_00e16,
72                   40_254_084_00e16,
73                   71_239_135_50e16,
74                   67_347_922_22e16,
75                   62_539_734_28e16,
76                   57_193_406_97e16,
77                   51_626_757_14e16,
78                   46_088_099_54e16,
79                   40_758_191_75e16,
80                   33_293_668_60e16
81:              ];

```
*GitHub*: [33](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L33-L44), [70](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsConstants.sol#L70-L81)

```solidity
File: contracts/TokenomicsProxy.sol

48:          (bool success, ) = tokenomics.delegatecall(tokenomicsData);

```
*GitHub*: [48](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/TokenomicsProxy.sol#L48-L48)

```solidity
File: contracts/Treasury.sol

224:         uint256 reserves = mapTokenReserves[token] + tokenAmount;

235:         bool success = IToken(token).transferFrom(account, address(this), tokenAmount);

```
*GitHub*: [224](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L224-L224), [235](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L235-L235)

</details>





### [G&#x2011;24] Split `revert` checks to save gas
Splitting the conditions into two separate checks [saves](https://gist.github.com/IllIllI000/7e25b0fca6bd9d57d9b9bcb9d2018959) **2 gas**

*There are 19 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: contracts/bridges/FxERC20ChildTunnel.sol

39           if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0)) {
40               revert ZeroAddress();
41:          }

```
*GitHub*: [39](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39-L41)

```solidity
File: contracts/bridges/FxERC20RootTunnel.sol

42           if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
43               _rootToken == address(0)) {
44               revert ZeroAddress();
45:          }

```
*GitHub*: [42](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxERC20RootTunnel.sol#L42-L45)

```solidity
File: contracts/bridges/FxGovernorTunnel.sol

64           if (_fxChild == address(0) || _rootGovernor == address(0)) {
65               revert ZeroAddress();
66:          }

```
*GitHub*: [64](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/FxGovernorTunnel.sol#L64-L66)

```solidity
File: contracts/bridges/HomeMediator.sol

64           if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) {
65               revert ZeroAddress();
66:          }

```
*GitHub*: [64](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/bridges/HomeMediator.sol#L64-L66)

```solidity
File: contracts/multisigs/GuardCM.sol

144          if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
145              revert ZeroAddress();
146:         }

453          if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) {
454              revert WrongArrayLength(targets.length, selectors.length, statuses.length, chainIds.length);
455:         }

506          if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) {
507              revert WrongArrayLength(bridgeMediatorL1s.length, bridgeMediatorL2s.length, chainIds.length, chainIds.length);
508:         }

513              if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0)) {
514                  revert ZeroAddress();
515:             }

```
*GitHub*: [144](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L144-L146), [453](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L453-L455), [506](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L506-L508), [513](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L513-L515)

```solidity
File: contracts/wveOLAS.sol

147          if (_ve == address(0) || _token == address(0)) {
148              revert ZeroAddress();
149:         }

```
*GitHub*: [147](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L147-L149)

```solidity
File: contracts/AgentRegistry.sol

41               if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) {
42                   revert ComponentNotFound(dependencies[iDep]);
43:              }

```
*GitHub*: [41](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/AgentRegistry.sol#L41-L43)

```solidity
File: contracts/ComponentRegistry.sol

30               if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) {
31                   revert ComponentNotFound(dependencies[iDep]);
32:              }

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L30-L32)

```solidity
File: contracts/Depository.sol

111          if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) {
112              revert ZeroAddress();
113:         }

363              if (pay == 0 || !matured) {
364                  revert BondNotRedeemable(bondIds[i]);
365:             }

```
*GitHub*: [111](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L111-L113), [363](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Depository.sol#L363-L365)

```solidity
File: contracts/Dispenser.sol

36           if (_tokenomics == address(0) || _treasury == address(0)) {
37               revert ZeroAddress();
38:          }

```
*GitHub*: [36](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Dispenser.sol#L36-L38)

```solidity
File: contracts/GenericBondCalculator.sol

31           if (_olas == address(0) || _tokenomics == address(0)) {
32               revert ZeroAddress();
33:          }

```
*GitHub*: [31](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L31-L33)

```solidity
File: contracts/Tokenomics.sol

283          if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
284              _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
285              _serviceRegistry == address(0)) {
286              revert ZeroAddress();
287:         }

1117             if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {
1118                 revert WrongUnitId(unitIds[i], unitTypes[i]);
1119:            }

1187             if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {
1188                 revert WrongUnitId(unitIds[i], unitTypes[i]);
1189:            }

```
*GitHub*: [283](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L283-L287), [1117](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1117-L1119), [1187](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L1187-L1189)

```solidity
File: contracts/Treasury.sol

100          if (_olas == address(0) || _tokenomics == address(0) || _depository == address(0) || _dispenser == address(0)) {
101              revert ZeroAddress();
102:         }

```
*GitHub*: [100](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L100-L102)

</details>





### [G&#x2011;25] Update OpenZeppelin dependency to save gas
Every release contains new gas optimizations. Use the latest version to take advantage of this

*There are 8 instances of this issue:*

```solidity
File: contracts/GovernorOLAS.sol

5:   import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

7:   import {GovernorCompatibilityBravo} from "@openzeppelin/contracts/governance/compatibility/GovernorCompatibilityBravo.sol";

8:   import {GovernorVotes, IVotes} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

9:   import {GovernorVotesQuorumFraction} from "@openzeppelin/contracts/governance/extensions/GovernorVotesQuorumFraction.sol";

10:  import {GovernorTimelockControl, TimelockController} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L5-L5), [7](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L8-L8), [9](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L9-L9), [10](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/GovernorOLAS.sol#L10-L10)

```solidity
File: contracts/Timelock.sol

4:   import "@openzeppelin/contracts/governance/TimelockController.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/Timelock.sol#L4-L4)

```solidity
File: contracts/veOLAS.sol

4:   import "@openzeppelin/contracts/governance/utils/IVotes.sol";

5:   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/veOLAS.sol#L5-L5)



### [G&#x2011;26] Use short-circuit evaluation to avoid external calls
By evaluating expressions involving constants, literals, and local variables before ones involving external calls, you can avoid unnecessarily executing the calls in the unhappy path.

*There is one instance of this issue:*

```solidity
File: contracts/Tokenomics.sol

/// @audit move the expression involving `getVotes()` to the right
709                  topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
710:                     IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;

```
*GitHub*: [709](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L709-L710)
