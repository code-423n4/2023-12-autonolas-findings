## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= ) | 1 | - |
| [G-02] | Require()/Revert() string longer than 32 bytes cost extra gas | 4 | - |
| [G-03] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 5 | - |
| [G-04] | Should use arguments instead of state variable | 3 | - |
| [G-05] | Before transfer of  some functions, we should check some variables for possible gas save | 1 | - |
| [G-06] | Use bitmap to save gas | 3 | - |
| [G-07] | State Variable can be packed into fewer storage slots | 4 | - |
| [G-08] | Bytes constants are more efficient than String constants | 3 | - |
| [G-09] | Duplicated require()/if() checks should be refactored to a modifier or function | 2 | - |
| [G-10] | Use constants instead of type(uintx).max | 8 | - |
| [G-11] | Use nested if and, avoid multiple check combinations | 4 | - |
| [G-12] | Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it) | 6 | - |
| [G-13] | Use selfbalance() instead of address(this).balance | 4 | - |
| [G-14] | abi.encode() is less efficient than  abi.encodepacked() | 2 | - |


## Gas Optimizations  

## [G-01] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES.
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).


```solidity
file: /solidity/liquidity_lockbox.sol

///@audit ' totalLiquidity ' is state variable.

189        totalLiquidity += positionLiquidity;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189C1-L189C45


## [G-02] Require()/Revert() string longer than 32 bytes cost extra gas  

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas.
when these strings get longer than 32 bytes, they cost more gas
Use shorter require()/revert() strings


```solidity
file: /solidity/liquidity_lockbox.sol

154            revert("Wrong bridged token mint account");

224            revert("No liquidity on a provided token account");

229            revert("Amount exceeds a position liquidity");

342            revert ("Requested amount is too big for the total available liquidity");

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L154C1-L154C56


## [G-03] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor. 

```solidity
file: /governance/contracts/multisigs/GuardCM.sol

97  bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));
    // scheduleBatch selector
99  bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));
    // requireToPassMessage selector (Gnosis chain)
101  bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));
    // processMessageFromForeign selector (Gnosis chain)
103  bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));
    // sendMessageToChild selector (Polygon)
105    bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L97C1-L105C114


## [G-04] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity
file: /contracts/bridges/FxERC20ChildTunnel.sol

107        emit FxDepositERC20(childToken, rootToken, msg.sender, to, amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L107C1-L107C76


```solidity
file: /contracts/bridges/FxERC20RootTunnel.sol

79        emit FxDepositERC20(childToken, rootToken, from, to, amount);

106        emit FxWithdrawERC20(rootToken, childToken, msg.sender, to, amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79C1-L79C70


## [G-05] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /contracts/veOLAS.sol

534        IERC20(token).transfer(msg.sender, amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L534C1-L534C52


## [G-06] Use bitmap to save gas

```solidity
file: /contracts/multisigs/GuardCM.sol

130    mapping(uint256 => bool) public mapAllowedTargetSelectorChainIds;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L130C1-L130C70


```solidity
file: /contracts/Tokenomics.sol

225    mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L225C1-L225C69


```solidity 
file: /contracts/GenericManager.sol

42        paused = true;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L42C1-L42C23


## [G-07] State Variable can be packed into fewer storage slots    				

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas).
Reads of the variables are also cheaper.

```solidity
file: /contracts/veOLAS.sol

99     uint64 internal constant WEEK = 1 weeks;
    // Maximum lock time (4 years)
101    uint256 internal constant MAXTIME = 4 * 365 * 86400;
    // Maximum lock time (4 years) in int128
103    int128 internal constant IMAXTIME = 4 * 365 * 86400;
    // Number of decimals
105    uint8 public constant decimals = 18;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L99C1-L105C41

## [G-08] Bytes constants are more efficient than String constants

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

```solidity
file: /contracts/GenericRegistry.sol

33    string public constant CID_PREFIX = "f01701220";

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L33C1-L33C53


```solidity
file: /contracts/ComponentRegistry.sol

10    string public constant VERSION = "1.0.0";

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L10C1-L10C46


```solidity
file: /contracts/Depository.sol

77    string public constant VERSION = "1.0.1";

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L77C1-L77C46


## [G-09] Duplicated require()/if() checks should be refactored to a modifier or function   

   to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /contracts/veOLAS.sol

///@audit the bellow if condition is duplicated on line : 278 and 293 

185        if (account != address(0)) {

///@audit the bellow if condition is duplicated on line : 449 and 474 

392        if (amount > type(uint96).max) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185C1-L185C37


## [G-10]  Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/OLAS.sol

131        if (spenderAllowance != type(uint256).max) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131C1-L131C53


```solidity
file: /contracts/veOLAS.sol

392        if (amount > type(uint96).max) {
393            revert Overflow(amount, type(uint96).max);

449        if (amount > type(uint96).max) {
450            revert Overflow(amount, type(uint96).max);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392C1-L393C55


```solidity
file: /contracts/UnitRegistry.sol

232            uint32 tryMinComponent = type(uint32).max;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232C1-L232C55


```solidity
file: /contracts/Depository.sol

195        if (priceLP > type(uint160).max) {
196            revert Overflow(priceLP, type(uint160).max);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195C1-L196C57


## [G-11] Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
file: /contracts/veOLAS.sol

188            if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) {

192            if (newLocked.endTime > block.timestamp && newLocked.amount > 0) {    

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188C1-L188C79


```solidity
file: /contracts/multisigs/GuardCM.sol

519            if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519C1-L519C92


```solidity
file: /contracts/Depository.sol

404            if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404C1-L404C108


## [G-12] Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it)


```solidity
file: /contracts/Tokenomics.sol

290        owner = msg.sender;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L290C1-L290C28


```solidity
file: /contracts/OLAS.sol

36        owner = msg.sender;
37        minter = msg.sender;
38        timeLaunch = block.timestamp;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L36C1-L38C29


```solidity
file: /contracts/bridges/BridgedERC20.sol

25        owner = msg.sender;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L25C1-L25C28


```solidity
file: /contracts/ComponentRegistry.sol

21        owner = msg.sender;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L21C1-L21C28


## [G-13] Use selfbalance() instead of address(this).balance

Using selfbalance() instead of address(this).balance can save gas in Solidity because selfbalance() is a built-in function that returns the balance of the current contract directly as a uint256 value, without the need to create a new address object.
On the other hand, using address(this).balance to get the balance of the current contract creates a new address object, which consumes more gas and can make the code less efficient.
By using selfbalance(), you can reduce the amount of gas consumed by your Solidity code and make your contracts more efficient.

```solidity
file: /contracts/bridges/FxGovernorTunnel.sol

147            if (value > address(this).balance) {
148                revert InsufficientBalance(value, address(this).balance);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147C1-L148C74


```solidity
file: /contracts/bridges/HomeMediator.sol

147            if (value > address(this).balance) {
148                revert InsufficientBalance(value, address(this).balance);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147C1-L148C74


## [G-14] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/bridges/FxERC20ChildTunnel.sol

97        bytes memory message = abi.encode(msg.sender, to, amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L97C1-L97C67


```solidity
file: /contracts/bridges/FxERC20RootTunnel.sol

93        bytes memory message = abi.encode(msg.sender, to, amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L93C1-L93C67

