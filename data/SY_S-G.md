## Summary

### Gas Optimization

no |Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Add unchecked {} for subtractions where the operands can’t underflow because of previous checks  |  |4|--| 
| [G-02] |Using assembly to revert with an error message |  |1|--| 
| [G-03] |Use assembly to reuse memory space when making more than one external call |  |11|--| 
| [G-04] |Don’t make variables public unless necessary |  |2|--| 
| [G-05] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= ) |  | 1 |
| [G-06] | Require()/Revert() string longer than 32 bytes cost extra gas |  | 4 |
| [G-07] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant |  | 5 |
| [G-08] | Should use arguments instead of state variable | | 6 |
| [G-09] | Before transfer of  some functions, we should check some variables for possible gas save |  | 1 |
| [G-10] | Use bitmap to save gas |  | 3 |
| [G-11] | State Variable can be packed into fewer storage slots |  | 4 |
| [G-12] | Bytes constants are more efficient than String constants |  | 3 |
| [G-13] | Duplicated require()/if() checks should be refactored to a modifier or function |  | 2 |
| [G-14] | Use constants instead of type(uintx).max |  | 8 |
| [G-15] | Use nested if and, avoid multiple check combinations | | 4 |
| [G-16] | Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it) |  | 6 |

## Gas Optimizations  

## [G-1] Add unchecked {} for subtractions where the operands can’t underflow because of previous checks
Issue Description - The subtraction operation remainder = supplyCap - _totalSupply; does not have an unchecked block, even though underflow
is not possible due to the previous check normalizedTransferAmount > amount. Adding an unchecked block helps avoid unnecessary
checks.

Proposed Optimization - Add an unchecked block around the subtraction:

```solidity
    113    remainder = supplyCap - _totalSupply;
remainder = supplyCap - _totalSupply;
file:
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L113-L113
```

```solidity
101        uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
File: /governance/contracts/OLAS.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L101
```
```solidity:
   148         pv = mapUserPoints[account][lastPointNumber - 1];

file:/governance/contracts/veOLAS.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L148
```
```solidity:
     245       lastPoint.bias -= lastPoint.slope * int128(int64(tStep - lastCheckpoint));


file:governance/contracts/veOLAS.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L245-L245
```
```solidity:
656        dBlock = pointNext.blockNumber - point.blockNumber;
            dt = pointNext.ts - point.ts;
        } else {
            dBlock = block.number - point.blockNumber;
            dt = block.timestamp - point.ts;
        }
        blockTime = point.ts;
        if (dBlock > 0) {
            blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;
        }

file:/contracts/veOLAS.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L656-L665


```
## [G-2] Using assembly to revert with an error message
Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s some example:


```solidity
 76  if (dataLength > DEFAULT_DATA_LENGTH) {
file: /contracts/multisigs/GnosisSafeMultisig.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeMultisig.sol#L76-L78
```
```solidity:
     
file:


``````solidity
file:


``````solidity
file:


```
  

## [G-3] Use assembly to reuse memory space when making more than one external call
Issue Description - When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not
clear or reuse memory, so it expands memory each time.

Proposed Optimization - Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments
without expanding memory further.
Estimated Gas Savings - Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000
gas. With larger arguments or return data, the savings would be even greater

```solidity
  75   function mint(address account, uint256 amount) external {
        // Access control
        if (msg.sender != minter) {
            revert ManagerOnly(msg.sender, minter);
        }


       
        if (inflationControl(amount)) {
            _mint(account, amount);
        }
    }
file:/governance/contracts/OLAS.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L75-L85


```
```solidity:
  42  function _findPointByBlock(uint256 blockNumber, address account) internal view
        returns (PointVoting memory point, uint256 minPointNumber)
    {
        // Get the last available point number
        uint256 maxPointNumber;
   47     if (account == address(0)) {
   48        maxPointNumber = totalNumPoints;
        } else {
   50        maxPointNumber = mapUserPoints[account].length;
   51         if (maxPointNumber == 0) {
                return (point, minPointNumber);
            }
file:/governance/contracts/veOLAS.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L542-L553
```
```solidity:
    484    LockedBalance memory lockedBalance = mapLockedBalances[msg.sender];
    146    uint256 lastPointNumber = mapUserPoints[account].length;
    148      pv = mapUserPoints[account][lastPointNumber - 1];
    487        unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
    522           supplyAfter = supplyBefore - amount;
       
 file:/governance/contracts/veOLAS.sol
    https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L484-L484

 file:  /governance/contracts/veOLAS.sol 
    https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L146

 file:/governance/contracts/veOLAS.sol
   https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L148

file:/governance/contracts/veOLAS.sol
  https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L487

file:governance/contracts/veOLAS.sol
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L522
```
## [G-4] Don’t make variables public unless necessary
Issue Description - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.
Proposed Optimization - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private
or internal visibility.
Estimated Gas Savings - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up
over many transactions targeting a contract with public state variables that don’t need to be public.

```solidity
 28   uint256 public immutable timeLaunch;
 108  address public immutable token;
file:/governance/contracts/OLAS.so
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L28
   
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L108
```

## [G-5] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES.
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).


```solidity
file: /solidity/liquidity_lockbox.sol

///@audit ' totalLiquidity ' is state variable.

189        totalLiquidity += positionLiquidity;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189C1-L189C45




```solidity
file:

```


```solidity
file:

```


```solidity
file:

```


```solidity
file:

```



## [G-06] Require()/Revert() string longer than 32 bytes cost extra gas  

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


## [G-07] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

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


## [G-08] Should use arguments instead of state variable 

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


## [G-09] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /contracts/veOLAS.sol

534        IERC20(token).transfer(msg.sender, amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L534C1-L534C52


## [G-10] Use bitmap to save gas

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


## [G-11] State Variable can be packed into fewer storage slots    				

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

## [G-12] Bytes constants are more efficient than String constants

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


## [G-13] Duplicated require()/if() checks should be refactored to a modifier or function   

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


## [G-14]  Use constants instead of type(uintx).max

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


## [G-15] Use nested if and, avoid multiple check combinations

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


## [G-16] Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it)


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