# Codebase Optimization Report

## Table of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Table of Contents](#table-of-contents)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Codebase impressions](#codebase-impressions)
  - [Pack variables(Save 1 SLOT: 2100 Gas)](#pack-variablessave-1-slot-2100-gas)
  - [Wrong packing order(Pack variables that are accessed together) save 2100 Gas](#wrong-packing-orderpack-variables-that-are-accessed-together-save-2100-gas)
  - [Wrong order of caching(save 1 SLOAD: 100 Gas)](#wrong-order-of-cachingsave-1-sload-100-gas)
  - [Refactor the code to only make state reads when necessary](#refactor-the-code-to-only-make-state-reads-when-necessary)
  - [Using unchecked blocks to save gas(Not found by bot)](#using-unchecked-blocks-to-save-gasnot-found-by-bot)
  - [Cache variables in stack  rather than re-reading them from storage(Not reported by bot)](#cache-variables-in-stack--rather-than-re-reading-them-from-storagenot-reported-by-bot)
  - [Optimizing check order for cost efficient function execution](#optimizing-check-order-for-cost-efficient-function-execution)
    - [Parameter validation should be prioritized](#parameter-validation-should-be-prioritized)
    - [Do not read state variables if we might revert on a parameter check validation](#do-not-read-state-variables-if-we-might-revert-on-a-parameter-check-validation)
    - [Validate `amount` before loading any state variable](#validate-amount-before-loading-any-state-variable)
    - [Parameters should be validated first before writing to state](#parameters-should-be-validated-first-before-writing-to-state)
    - [Do not read any state variables before validating all non state variables](#do-not-read-any-state-variables-before-validating-all-non-state-variables)
    - [Cheaper checks should be prioritized](#cheaper-checks-should-be-prioritized)
    - [Revert early and cheaply](#revert-early-and-cheaply)
    - [Validate function parameters first before reading from storage](#validate-function-parameters-first-before-reading-from-storage)
    - [Cheap checks should be done first](#cheap-checks-should-be-done-first)
    - [Validate `unitHash` first before doing any other operations](#validate-unithash-first-before-doing-any-other-operations)
    - [Avoid reading any state variables if we might revert on a cheap check](#avoid-reading-any-state-variables-if-we-might-revert-on-a-cheap-check)
    - [Cheaper to validate immutable variables first](#cheaper-to-validate-immutable-variables-first)
    - [Validate the lengths before reading the state variable `dispenser`](#validate-the-lengths-before-reading-the-state-variable-dispenser)
  - [Conclusion](#conclusion)


## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Codebase impressions

The developers have done an excellent job of identifying and implementing some of the most evident optimizations. They have done an excellent job with storage packing while providing the math(slot count) when dealing with structs which has been helpful

However, we have identified additional areas where optimization is possible, some of which may not be immediately apparent.

## Pack variables(Save 1 SLOT: 2100 Gas)

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L17-L23
```solidity
File: /registries/contracts/GenericRegistry.sol
17:    address public manager;
18:    // Base URI
19:    string public baseURI;
20:    // Unit counter
21:    uint256 public totalSupply;
22:    // Reentrancy lock
23:    uint256 internal _locked = 1;
```

For the `reentrancy lock`, there are only 2 possibilities ie either the value is 1 or 2 thus we can reduce the type of the variable from `uint256` to `uint8` and pack with the variable `address manager`

The reason we pack with manager is because the two are accessed in the same transaction on the contract `UnitRegistry` 
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L53-L61
```solidity
53:        if (_locked > 1) {
54:            revert ReentrancyGuard();
55:        }
56:        _locked = 2;

58:        // Check for the manager privilege for a unit creation
59:        if (manager != msg.sender) {
60:            revert ManagerOnly(msg.sender, manager);
61:        }
```

We can refactor the code as follows to ensure the variables are packed 

```diff
diff --git a/registries/contracts/GenericRegistry.sol b/registries/contracts/Gen
ericRegistry.sol
index 6dedf90..33ec5d9 100644
--- a/registries/contracts/GenericRegistry.sol
+++ b/registries/contracts/GenericRegistry.sol
@@ -15,12 +15,13 @@ abstract contract GenericRegistry is IErrorsRegistries, ERC721 {
     address public owner;
     // Unit manager
     address public manager;
+       // Reentrancy lock
+    uint8 internal _locked = 1;
     // Base URI
     string public baseURI;
     // Unit counter
     uint256 public totalSupply;
-    // Reentrancy lock
-    uint256 internal _locked = 1;
+
```

## Wrong packing order(Pack variables that are accessed together) save 2100 Gas

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L80-L95
```solidity
File: /tokenomics/contracts/Depository.sol
80:    address public owner;

83:    uint32 public bondCounter;

86:    uint32 public productCounter;

89:    address public immutable olas;
90:    // Tkenomics contract address
91:    address public tokenomics;
92:    // Treasury contract address
93:    address public treasury;
94:    // Bond Calculator contract address
95:    address public bondCalculator;
```

Currently the variables `owner,bondCounter,productCounter` are packed together. However, only `owner,productCounter` are accessed together, meaning that the benefits of packing are only available to this two variables.
When packing, we should pack variables that are accessed in the same transaction. We can therefore move the `bondCounter` variable and pack it with another variable eg `bondCalculator` which would ensure we benefit from packing

```diff
diff --git a/tokenomics/contracts/Depository.sol b/tokenomics/contracts/Depository.sol
index f9c7d64..ff11fef 100644
--- a/tokenomics/contracts/Depository.sol
+++ b/tokenomics/contracts/Depository.sol
@@ -78,9 +78,6 @@ contract Depository is IErrorsTokenomics {

     // Owner address
     address public owner;
-    // Individual bond counter
-    // We assume that the number of bonds will not be bigger than the number of seconds
-    uint32 public bondCounter;
     // Bond product counter
     // We assume that the number of products will not be bigger than the number of seconds
     uint32 public productCounter;
@@ -93,6 +90,9 @@ contract Depository is IErrorsTokenomics {
     address public treasury;
     // Bond Calculator contract address
     address public bondCalculator;
+        // Individual bond counter
+    // We assume that the number of bonds will not be bigger than the number of seconds
+    uint32 public bondCounter;

     // Mapping of bond Id => account bond instance
     mapping(uint256 => Bond) public mapUserBonds;
```

## Wrong order of caching(save 1 SLOAD: 100 Gas)

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L897-L906
```solidity
File: /tokenomics/contracts/Tokenomics.sol
897:        uint256 prevEpochTime = mapEpochTokenomics[epochCounter - 1].epochPoint.endTime;
898:        uint256 diffNumSeconds = block.timestamp - prevEpochTime;
899:        uint256 curEpochLen = epochLen;

902:        if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) {
903:            return false;
904:        }

906:        uint256 eCounter = epochCounter;
```

On Line 897, we read the state variable `epochCounter` and then we read it again when caching it on line 906.
If we start by caching, we can save 1 SLOAD(100 Gas)

```diff
+        uint256 eCounter = epochCounter;
+        uint256 prevEpochTime = mapEpochTokenomics[eCounter - 1].epochPoint.endTime;
         uint256 diffNumSeconds = block.timestamp - prevEpochTime;
         uint256 curEpochLen = epochLen;
         // Check if the time passed since the last epoch end time is bigger than the specified epoch length,
@@ -903,7 +904,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
             return false;
         }

-        uint256 eCounter = epochCounter;
+
         TokenomicsPoint storage tp = mapEpochTokenomics[eCounter];
```

## Refactor the code to only make state reads when necessary

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L400-L410
```solidity
File: /tokenomics/contracts/Treasury.sol
400:        uint256 amountETHFromServices = ETHFromServices;
401:        // Send ETH rewards, if any
402:        if (accountRewards > 0 && amountETHFromServices >= accountRewards) {
403:            amountETHFromServices -= accountRewards;
404:            ETHFromServices = uint96(amountETHFromServices);
405:            emit Withdraw(ETH_TOKEN_ADDRESS, account, accountRewards);
406:            (success, ) = account.call{value: accountRewards}("");
407:            if (!success) {
408:                revert TransferFailed(address(0), address(this), account, accountRewards);
409:            }
410:        }
```


```diff
-        uint256 amountETHFromServices = ETHFromServices;
         // Send ETH rewards, if any
-        if (accountRewards > 0 && amountETHFromServices >= accountRewards) {
-            amountETHFromServices -= accountRewards;
-            ETHFromServices = uint96(amountETHFromServices);
-            emit Withdraw(ETH_TOKEN_ADDRESS, account, accountRewards);
-            (success, ) = account.call{value: accountRewards}("");
-            if (!success) {
-                revert TransferFailed(address(0), address(this), account, accountRewards);
+        if (accountRewards > 0 ){
+            uint256 amountETHFromServices = ETHFromServices;
+            if (amountETHFromServices >= accountRewards) {
+                amountETHFromServices -= accountRewards;
+                ETHFromServices = uint96(amountETHFromServices);
+                emit Withdraw(ETH_TOKEN_ADDRESS, account, accountRewards);
+                (success, ) = account.call{value: accountRewards}("");
+                if (!success) {
+                    revert TransferFailed(address(0), address(this), account, accountRewards);
+                }
             }
         }

```

## Using unchecked blocks to save gas(Not found by bot)

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L147-L149
```solidity
File: /governance/contracts/veOLAS.sol
147:        if (lastPointNumber > 0) {
148:            pv = mapUserPoints[account][lastPointNumber - 1];
149:        }
```
The operation `lastPointNumber - 1` cannot overflow since we have a check that ensures that `lastPointNumber` is greater than `0`. We can therefore wrap the whole thing in unchecked blocks

```diff
@@ -145,7 +145,10 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
     function getLastUserPoint(address account) external view returns (PointVoting memory pv) {
         uint256 lastPointNumber = mapUserPoints[account].length;
         if (lastPointNumber > 0) {
-            pv = mapUserPoints[account][lastPointNumber - 1];
+            unchecked{
+                pv = mapUserPoints[account][lastPointNumber - 1];
+            }
+
         }
     }
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L335
```solidity
File: /tokenomics/contracts/Treasury.sol
335:                amountOwned -= tokenAmount;
```

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L355
```solidity
File: /tokenomics/contracts/Treasury.sol
355:                reserves -= tokenAmount;
```

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L403
```solidity
File: /tokenomics/contracts/Treasury.sol
403:            amountETHFromServices -= accountRewards;
```
The operation `amountETHFromServices -= accountRewards` cannot overflow as we perform a check `amountETHFromServices >= accountRewards` that has to be true before we do our arithmetic

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L445
```solidity
File: /tokenomics/contracts/Treasury.sol
445:                amountETHFromServices -= treasuryRewards;
```
The operation `amountETHFromServices -= treasuryRewards` cannot overflow as we are only performing it if `amountETHFromServices >= treasuryRewards`

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L328
```solidity
File: /tokenomics/contracts/Depository.sol
328:        supply -= payout;
```
we have a condition check `if (payout > supply) {` which ensures that our arithmetic operation would only be performed if supply is greater than payout. This can therefore never overflow and can be wrapped in unchecked blocks


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L59-L61
```solidity
File: /governance/contracts/OLAS.sol
59:        if (msg.sender != owner) {
60:            revert ManagerOnly(msg.sender, owner);
61:        }
```

##  Cache variables in stack  rather than re-reading them from storage(Not reported by bot)

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L228-L230
```solidity
File: /tokenomics/contracts/Treasury.sol
228:        if (IToken(token).allowance(account, address(this)) < tokenAmount) {
229:            revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);
230:        }
```

Cache the call `IToken(token).allowance(account, address(this))`. As this is an external call, in case of a revert, we would make the external call twice which would be very expensive
If we cache and don't revert we would only waste a small amount compared to the alternative

## Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Parameter validation should be prioritized 

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L376-L381
```solidity
File: /governance/contracts/veOLAS.sol
376:    function depositFor(address account, uint256 amount) external {
377:        LockedBalance memory lockedBalance = mapLockedBalances[account];
378:        // Check if the amount is zero
379:        if (amount == 0) {
380:            revert ZeroValue();
381:        }
```

It's cheaper to read the function parameter `amount` as compared to reading a storage variable `mapLockedBalances[account]`. We should therefore do the parameter validation first before making any state reads

```diff
@@ -374,11 +374,12 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
     /// @param account Account address.
     /// @param amount Amount to add.
     function depositFor(address account, uint256 amount) external {
-        LockedBalance memory lockedBalance = mapLockedBalances[account];
         // Check if the amount is zero
         if (amount == 0) {
             revert ZeroValue();
         }
+        LockedBalance memory lockedBalance = mapLockedBalances[account];
+
         // The locked balance must already exist
         if (lockedBalance.amount == 0) {
             revert NoValueLocked(account);
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L425-L451

### Do not read state variables if we might revert on a parameter check validation

```solidity
File: /governance/contracts/veOLAS.sol
425:    function _createLockFor(address account, uint256 amount, uint256 unlockTime) private {
426:        // Check if the amount is zero
427:        if (amount == 0) {
428:            revert ZeroValue();
429:        }
430:        // Lock time is rounded down to weeks
431:        // Cannot practically overflow because block.timestamp + unlockTime (max 4 years) << 2^64-1
432:        unchecked {
433:            unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
434:        }
435:        LockedBalance memory lockedBalance = mapLockedBalances[account];
436:        // The locked balance must be zero in order to start the lock
437:        if (lockedBalance.amount > 0) {
438:            revert LockedValueNotZero(account, uint256(lockedBalance.amount));
439:        }
440:        // Check for the lock time correctness
441:        if (unlockTime < (block.timestamp + 1)) {
442:            revert UnlockTimeIncorrect(account, block.timestamp, unlockTime);
443:        }
444:        // Check for the lock time not to exceed the MAXTIME
445:        if (unlockTime > block.timestamp + MAXTIME) {
446:            revert MaxUnlockTimeReached(account, block.timestamp + MAXTIME, unlockTime);
447:        }
448:        // After 10 years, the inflation rate is 2% per year. It would take 220+ years to reach 2^96 - 1 total supply
449:        if (amount > type(uint96).max) {
450:            revert Overflow(amount, type(uint96).max);
451:        }
```

```diff
     // Total number of economical checkpoints (starting from zero)
     uint256 public totalNumPoints;
@@ -432,11 +432,6 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
         unchecked {
             unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
         }
-        LockedBalance memory lockedBalance = mapLockedBalances[account];
-        // The locked balance must be zero in order to start the lock
-        if (lockedBalance.amount > 0) {
-            revert LockedValueNotZero(account, uint256(lockedBalance.amount));
-        }
         // Check for the lock time correctness
         if (unlockTime < (block.timestamp + 1)) {
             revert UnlockTimeIncorrect(account, block.timestamp, unlockTime);
@@ -449,6 +444,12 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
         if (amount > type(uint96).max) {
             revert Overflow(amount, type(uint96).max);
         }
+        LockedBalance memory lockedBalance = mapLockedBalances[account];
+        // The locked balance must be zero in order to start the lock
+        if (lockedBalance.amount > 0) {
+            revert LockedValueNotZero(account, uint256(lockedBalance.amount));
+        }
+

         _depositFor(account, amount, unlockTime, lockedBalance, DepositType.CREATE_LOCK_TYPE);
     }

```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L459-L463

### Validate `amount` before loading any state variable

```solidity
File: /governance/contracts/veOLAS.sol
459:        LockedBalance memory lockedBalance = mapLockedBalances[msg.sender];
460:        // Check if the amount is zero
461:        if (amount == 0) {
462:            revert ZeroValue();
463:        }
```

```diff
     function increaseAmount(uint256 amount) external {
+         // Check if the amount is zero
+         if (amount == 0) {
+             revert ZeroValue();
+         }
         LockedBalance memory lockedBalance = mapLockedBalances[msg.sender];
-        // Check if the amount is zero
-        if (amount == 0) {
-            revert ZeroValue();
-        }
+
         // The locked balance must already exist
         if (lockedBalance.amount == 0) {
             revert NoValueLocked(msg.sender);
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L95-L102

### Parameters should be validated first before writing to state

```solidity
File: /tokenomics/contracts/Treasury.sol
95:    constructor(address _olas, address _tokenomics, address _depository, address _dispenser) payable {
96:        owner = msg.sender;
97:        _locked = 1;

100:        if (_olas == address(0) || _tokenomics == address(0) || _depository == address(0) || _dispenser == address(0)) {
101:            revert ZeroAddress();
102:        }
```


```diff
diff --git a/tokenomics/contracts/Treasury.sol b/tokenomics/contracts/Treasury.sol
index 516c631..def6e78 100644
--- a/tokenomics/contracts/Treasury.sol
+++ b/tokenomics/contracts/Treasury.sol
@@ -93,13 +93,14 @@ contract Treasury is IErrorsTokenomics {
     /// @param _depository Depository address.
     /// @param _dispenser Dispenser address.
     constructor(address _olas, address _tokenomics, address _depository, address _dispenser) payable {
-        owner = msg.sender;
-        _locked = 1;
-
         // Check for at least one zero contract address
         if (_olas == address(0) || _tokenomics == address(0) || _depository == address(0) || _dispenser == address(0)) {
             revert ZeroAddress();
-        }
+        }//@audit move this check to the top
+
+        owner = msg.sender;
+        _locked = 1;
+

         olas = _olas;
         tokenomics = _tokenomics;
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L139-L146

### Do not read any state variables before validating all non state variables

```solidity
File: /tokenomics/contracts/Treasury.sol
139:        if (msg.sender != owner) {
140:            revert OwnerOnly(msg.sender, owner);
141:        }

143:        // Check for the zero address
144:        if (newOwner == address(0)) {
145:            revert ZeroAddress();
146:        }
```


```diff
     function changeOwner(address newOwner) external {
-        // Check for the contract ownership
-        if (msg.sender != owner) {
-            revert OwnerOnly(msg.sender, owner);
-        }
-
         // Check for the zero address
         if (newOwner == address(0)) {
             revert ZeroAddress();
         }
+        // Check for the contract ownership
+        if (msg.sender != owner) {
+            revert OwnerOnly(msg.sender, owner);
+        }
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L184-L191

### Cheaper checks should be prioritized 

```solidity
File: /tokenomics/contracts/Treasury.sol
184:        if (msg.sender != owner) {
185:            revert OwnerOnly(msg.sender, owner);
186:        }

188:        // Check for the zero value
189:        if (_minAcceptedETH == 0) {
190:            revert ZeroValue();
191:        }

193:        // Check for the overflow value
194:        if (_minAcceptedETH > type(uint96).max) {
195:            revert Overflow(_minAcceptedETH, type(uint96).max);
196:        }
```

```diff
     function changeMinAcceptedETH(uint256 _minAcceptedETH) external {
-        // Check for the contract ownership
-        if (msg.sender != owner) {
-            revert OwnerOnly(msg.sender, owner);
-        }
-
-        // Check for the zero value
+                // Check for the zero value
         if (_minAcceptedETH == 0) {
             revert ZeroValue();
         }
@@ -194,6 +189,10 @@ contract Treasury is IErrorsTokenomics {
         if (_minAcceptedETH > type(uint96).max) {
             revert Overflow(_minAcceptedETH, type(uint96).max);
         }
+        // Check for the contract ownership
+        if (msg.sender != owner) {
+            revert OwnerOnly(msg.sender, owner);
+        }

         minAcceptedETH = uint96(_minAcceptedETH);
         emit MinAcceptedETHUpdated(_minAcceptedETH);q
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L259-L273

### Revert early and cheaply

```solidity
File: /tokenomics/contracts/Treasury.sol
259:        if (_locked > 1) {
260:            revert ReentrancyGuard();
261:        }
262:        _locked = 2;

265:        if (msg.value < minAcceptedETH) {
266:            revert LowerThan(msg.value, minAcceptedETH);
267:        }

270:        uint256 numServices = serviceIds.length;
271:        if (amounts.length != numServices) {
272:            revert WrongArrayLength(numServices, amounts.length);
273:        }
```

```diff
     function depositServiceDonationsETH(uint256[] memory serviceIds, uint256[] memory amounts) external payable {
+                // Check for the same length of arrays
+        uint256 numServices = serviceIds.length;
+        if (amounts.length != numServices) {
+            revert WrongArrayLength(numServices, amounts.length);
+        }
         // Reentrancy guard
         if (_locked > 1) {
             revert ReentrancyGuard();
@@ -266,12 +271,6 @@ contract Treasury is IErrorsTokenomics {
             revert LowerThan(msg.value, minAcceptedETH);
         }

-        // Check for the same length of arrays
-        uint256 numServices = serviceIds.length;
-        if (amounts.length != numServices) {
-            revert WrongArrayLength(numServices, amounts.length);
-        }

```

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L315-L327

### Validate function parameters first before reading from storage

```solidity
File: /tokenomics/contracts/Treasury.sol
315:        if (msg.sender != owner) {
316:            revert OwnerOnly(msg.sender, owner);
317:        }

320:        if (to == address(this)) {
321:            revert TransferFailed(token, address(this), to, tokenAmount);
322:        }

325:        if (tokenAmount == 0) {
326:            revert ZeroValue();
327:        }
```

```diff
     function withdraw(address to, uint256 tokenAmount, address token) external returns (bool success) {
-        // Check for the contract ownership
-        if (msg.sender != owner) {
-            revert OwnerOnly(msg.sender, owner);
-        }
-
-        // Check that the withdraw address is not treasury itself
+                // Check that the withdraw address is not treasury itself
         if (to == address(this)) {
             revert TransferFailed(token, address(this), to, tokenAmount);
         }
@@ -325,6 +320,10 @@ contract Treasury is IErrorsTokenomics {
         if (tokenAmount == 0) {
             revert ZeroValue();
         }
+        // Check for the contract ownership
+        if (msg.sender != owner) {
+            revert OwnerOnly(msg.sender, owner);
+        }

```

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L489-L496

### Cheap checks should be done first

```solidity
File: /tokenomics/contracts/Treasury.sol
489:        if (msg.sender != owner) {
490:            revert OwnerOnly(msg.sender, owner);
491:        }

493:        // Check for the zero address token
494:        if (token == address(0)) {
495:            revert ZeroAddress();
496:        }
```

```diff
     function enableToken(address token) external {
+         // Check for the zero address token
+         if (token == address(0)) {
+             revert ZeroAddress();
+         }
         // Check for the contract ownership
         if (msg.sender != owner) {
             revert OwnerOnly(msg.sender, owner);
         }

-        // Check for the zero address token
-        if (token == address(0)) {
-            revert ZeroAddress();
-        }
-
         // Authorize the token
         if (!mapEnabledTokens[token]) {
             mapEnabledTokens[token] = true;
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L125-L141

### Validate `unitHash` first before doing any other operations

```solidity
File: /registries/contracts/UnitRegistry.sol
125:        if (manager != msg.sender) {
126:            revert ManagerOnly(msg.sender, manager);
127:        }

130:        if (ownerOf(unitId) != unitOwner) {
131:            if (unitType == UnitType.Component) {
132:                revert ComponentNotFound(unitId);
133:            } else {
134:                revert AgentNotFound(unitId);
135:            }
136:        }

139:        if (unitHash == 0) {
140:            revert ZeroValue();
141:        }
```


```diff
     function updateHash(address unitOwner, uint256 unitId, bytes32 unitHash) external virtual
         returns (bool success)
-    {
-        // Check the manager privilege for a unit modification
+
+        // Check for the hash value
+        if (unitHash == 0) {
+            revert ZeroValue();
+        }
+        // Che the manager privilege for a unit modification
         if (manager != msg.sender) {
             revert ManagerOnly(msg.sender, manager);
         }
@@ -135,11 +139,6 @@ abstract contract UnitRegistry is GenericRegistry {
             }
         }

-        // Check for the hash value
-        if (unitHash == 0) {
-            revert ZeroValue();
-        }
-
         mapUnitIdHashes[unitId].push(unitHash);
         success = true;

```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L571-L583

### Avoid reading any state variables if we might revert on a cheap check

```solidity
File: /tokenomics/contracts/Tokenomics.sol
571:        if (msg.sender != owner) {
572:            revert OwnerOnly(msg.sender, owner);
573:        }

576:        if (_rewardComponentFraction + _rewardAgentFraction > 100) {
577:            revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);
578:        }

581:        if (_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction > 100) {
582:            revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);
583:        }
```

```diff
@@ -567,20 +567,19 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
         uint256 _topUpAgentFraction
     ) external
     {
-        // Check for the contract ownership
-        if (msg.sender != owner) {
-            revert OwnerOnly(msg.sender, owner);
-        }
-
         // Check that the sum of fractions is 100%
         if (_rewardComponentFraction + _rewardAgentFraction > 100) {
             revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);
-        }
-
+        }
         // Same check for top-up fractions
         if (_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction > 100) {
             revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);
         }
+        // Check for the contract ownership
+        if (msg.sender != owner) {
+            revert OwnerOnly(msg.sender, owner);
+        }
+
```


https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L384-L393

### Cheaper to validate immutable variables first

```solidity
File: /tokenomics/contracts/Tokenomics.sol
384:    function changeTokenomicsImplementation(address implementation) external {
385:        // Check for the contract ownership
386:        if (msg.sender != owner) {
387:            revert OwnerOnly(msg.sender, owner);
388:        }

390:        // Check for the zero address
391:        if (implementation == address(0)) {
392:            revert ZeroAddress();
393:        }
```

```diff
     function changeTokenomicsImplementation(address implementation) external {
+         // Check for the zero address
+         if (implementation == address(0)) {
+             revert ZeroAddress();
+         }
         // Check for the contract ownership
         if (msg.sender != owner) {
             revert OwnerOnly(msg.sender, owner);
         }

-        // Check for the zero address
-        if (implementation == address(0)) {
-            revert ZeroAddress();
-        }
-
         // Store the implementation address under the designated storage slot
         assembly {
             sstore(PROXY_TOKENOMICS, implementation)
```

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L1089-L1096

### Validate the lengths before reading the state variable `dispenser`

```solidity
File: /tokenomics/contracts/Tokenomics.sol
1089:        if (dispenser != msg.sender) {
1090:            revert ManagerOnly(msg.sender, dispenser);
1091:        }

1093:        // Check array lengths
1094:        if (unitTypes.length != unitIds.length) {
1095:            revert WrongArrayLength(unitTypes.length, unitIds.length);
1096:        }
```


```diff
     function accountOwnerIncentives(address account, uint256[] memory unitTypes, uint256[] memory unitIds) external
         returns (uint256 reward, uint256 topUp)
     {
+       // Check array lengths
+       if (unitTypes.length != unitIds.length) {
+           revert WrongArrayLength(unitTypes.length, unitIds.length);
+       }
         // Check for the dispenser access
         if (dispenser != msg.sender) {
             revert ManagerOnly(msg.sender, dispenser);
         }

-        // Check array lengths
-        if (unitTypes.length != unitIds.length) {
-            revert WrongArrayLength(unitTypes.length, unitIds.length);
-        }
-
         // Component / agent registry addresses
         address[] memory registries = new address[](2);
         (registries[0], registries[1]) = (componentRegistry, agentRegistry);
```


## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
