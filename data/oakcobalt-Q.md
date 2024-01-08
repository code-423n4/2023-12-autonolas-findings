### Low-01: `decreaseAllowance()` returns misleading values when `type(uint256).max` is current spender allowance.
In OLAS.sol, `decreaseAllowance()` allows a caller to reduce the current allowance for a spender. When allowance is successfully reduced, the function return true to indicate success.

However, when the existing allowance for the spender is `type(uint256).max`, the function will do nothing but also return true. This is misleading behavior that might cause the caller or the parent flow to mistakenly assume the allowance has been reduced when it is not. The caller might not be aware that max allowance has been set for the spender. 

```solidity
//contracts/OLAS.sol
    function decreaseAllowance(address spender, uint256 amount) external returns (bool) {
        uint256 spenderAllowance = allowance[msg.sender][spender];
        //@audit when spenderAllowance is type(uint256).max, if statement will be skipped, but function will still return true, which is misleading because the allowance is not reduced and the caller might also not be aware that max allowance has been set for the spender.
|>      if (spenderAllowance != type(uint256).max) {
            spenderAllowance -= amount;
            allowance[msg.sender][spender] = spenderAllowance;
            emit Approval(msg.sender, spender, spenderAllowance);
        }
        return true;
    }
```
(https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L131)

**Recommendation:**
Consider revert with an error message to alert user when spenderAllowance is type(uint256).max.

### Low-02: Incorrect comment. Ethereum already has fixed block time since proof-of-stake upgrades.
In veOLAS.sol, there are comments explaining time-weighted vote count logic. However, there is incorrect comment when it comes to block number and block time.

It says `We cannot really do block numbers per se because slope is per time, not per block, and per block could be fairly bad because Ethereum changes its block times. What we can do is to extrapolate ***At functions.` 

The above comment is incorrect, because Ethereum already has fixed block time as of now. Although this doesn't affect the overall code logic, it's good practice and reduces efforts in audits to have correct code comments.

**Recommendation:**
Remove the comment regarding Ethereum block times.

### Low-03: Incorrect and misleading comments on function arguments
In veOLAS.sol, there are multiple functions that takes `uint256 unlockTime` as one of the arguments. However, the code comment for `@param unlockTime` is incorrect and misleading.

When passed as a parameter `uint256 unlockTime` will be the duration of the lock in seconds. And inside function body, `unlockTime` will be re-written to the end timestamp of the lock based on the lock duration passed. `unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;`

The comment for `@param unlockTime` is `unlockTime Time when tokens unlock, rounded down to a whole week.`, which indicates the parameter should be the lock end timestamp when the parameter is used as lock duration in function body.

**Instances(2):**
(1)
```solidity
//contracts/veOLAS.sol
     //@audit incorrect and misleading comments, based on implementation below, unlockTime is supposed to be the duration of the lock, however, comment suggest it's the end lock timestamp
|>  /// @param unlockTime Time when tokens unlock, rounded down to a whole week.
    function _createLockFor(address account, uint256 amount, uint256 unlockTime) private {
...
unchecked {
            unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
        }

```
(https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L424)
(2)
```solidity
//contracts/veOLAS.sol
    /// @dev Extends the unlock time.
    //@audit incorrect and misleading comments, based on implementation below, unlockTime is supposed to be the duration of the lock, however, comment suggest it's the end lock timestamp
|>  /// @param unlockTime New tokens unlock time.
    function increaseUnlockTime(uint256 unlockTime) external {
...
        unchecked {
            unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
        }
```
(https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L482)

**Recommendation:**
Change `@param unlockTime` comment to say the lock duration.

### Low-04:`balanceOfAt()` in veOLAS.sol will return inaccurate value when the balance of future `blockNumber` is queried
In veOLAS.sol, `balanceOfAt()` the lock balance for at a given blockNumber will be returned. However, the return value will be inaccurate when the `blockNumber` is a future block because the lock balance might change.

In `balanceOfAt()`, first the closest block number to provided `blockNumber` will be search in `_findPointByBlock()`. When the input `blockNumber` is in-between the earliest checkpoint blockNumber of the account and the last checkpoint of the account, `_findPointByBlock()` will return the block number in record that is right before `blockNumber` parameter. 

But when the input `blockNumber` is after the last checkpoint of the account, `_findPointByBlock()` will return the last checkpoint without checking whether the input `blockNumber` is earlier or later than current `block.timestamp`. This means, if `blockNumber` is a future block (> `block.timestamp`), the function will still pass and return the balance at the last checkpoint, this is inaccurate and misleading since we don't know what will happen in the future. Instead, the function should check whether `blockNumber` is greater than current `block.timestamp`, if so revert with an alert message to avoid giving misleading answers.

```solidity
//contracts/veOLAS.sol
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance) {
       //@audit  future blockNumber input will likely result in inaccurate return value since account balance might change. Consider disable when query account balance at a future block.
        // Find point with the closest block number to the provided one
|>      (PointVoting memory uPoint, ) = _findPointByBlock(blockNumber, account);
        // If the block number at the point index is bigger than the specified block number, the balance was zero
        //@audit this will return balance even if blockNumber is a future block
        if (uPoint.blockNumber < (blockNumber + 1)) {
|>          balance = uint256(uPoint.balance);
        }
    }

    function _findPointByBlock(uint256 blockNumber, address account) internal view
        returns (PointVoting memory point, uint256 minPointNumber)
    {
...
        // Binary search that will be always enough for 128-bit numbers
        for (uint256 i = 0; i < 128; ++i) {
        //@audit-info note: this will ensure that minPointNumber found will be right before maxPointNumber
            if ((minPointNumber + 1) > maxPointNumber) {
                break;
            }
...
        }
        // Get the found point
        if (account == address(0)) {
            point = mapSupplyPoints[minPointNumber];
        } else {
       //@audit-info note: this returns minPointNumber that is before blockNumber
            point = mapUserPoints[account][minPointNumber];
        }
...
```
(https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L626-L627)

**Recommendation:**
Consider check `blockNumber` with `block.timestamp` and revert when `blockNumber` is greater than `block.timestamp`


