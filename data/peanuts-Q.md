### [L-01] MIN_VESTING should be adjustable in Depository.sol

MIN_VESTING is currently set as 1 days in Depository.sol.

```
    uint256 public constant MIN_VESTING = 1 days;
```

This value cannot be changed, but ideally it should be flexible because it will give the protocol more opportunities to grow their system. For example, setting min vesting to less than a day can incentivise more people to bond their LP tokens for OLAS tokens. Changing the amount will incur more centralization risks, but since the protocol is heavily reliant on centralization already, it will be better to add more convenience for a slight increase in centralization risk.

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L209-L212

### [L-02] `getBonds()` in Depository.sol may face out of gas error since bondCounter only increases for every bond created through `deposit()`

In `getBonds()`, the function attempts to get all the bonds from one account using a loop.

```
  uint256 numBonds = bondCounter;
        bool[] memory positions = new bool[](numBonds);
        // Record the bond number if it belongs to the account address and was not yet redeemed
        for (uint256 i = 0; i < numBonds; ++i) {
```

`bondCounter` increases everytime a user calls `deposit()` and will never decrease

```
 // Create and add a new bond, update the bond counter
        bondId = bondCounter;
        mapUserBonds[bondId] = Bond(msg.sender, uint96(payout), uint32(maturity), uint32(productId));
        bondCounter = uint32(bondId + 1);
```

This means that as more users start to call `deposit()` to create a bond, the value of `bondCounter` will increase. If it gets too large, the `getBonds()` function will run out of gas as it attempts to loop through every bond created just to find the bond that a particular account has created.

To give specific numbers, if Alice creates the first bond, `bondCounter = 1`, and there are 10000 bonds created after that, and Alice creates another bond `bondCounter = 10001`, then the function has to loop 10001 times just to find that Alice has created the first bond and the latest bond.

This function is extremely gas intensive. A better way is to create a mapping for the particular user every time a bond is created through `deposit()`, and push the new value into the mapping.

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L435-L447

### [L-03] Users using the `deposit()` function in Depository.sol can be DoS'd by ensuring that payout > supply everytime

In `deposit()`, a user deposits a certain amount of LP tokens to get a certain amount of OLAS tokens. This calculation is done through `IGenericBondCalculator(bondCalculator).calculatePayoutOLAS`, and if the payout is greater than the supply, revert the whole function

```
// Check for the sufficient supply
        if (payout > supply) {
            revert ProductSupplyLow(token, productId, payout, supply);
        }
```

To calculate the exact amount such that payout == supply, the user has to do his own calculation and call `calculatePayoutOLAS()` directly.

```
  function calculatePayoutOLAS(uint256 tokenAmount, uint256 priceLP) external view
        returns (uint256 amountOLAS)
    {
```

Let's say that the remaining supply of OLAS token is 10000, and it takes 100 LP tokens to get 1000 OLAS tokens. Alice calls `deposit()` and deposit 100 LP tokens, with the intention of getting the last 1000 OLAS tokens. Another user sees this, and frontruns the transaction by depositing 1 wei of LP tokens and get 10 wei of OLAS tokens. Alice transaction then gets executed, but reverts because the payout is now greater than the supply. Alice has to calculate again and deposit 100-1wei of LP tokens to get 1000-10wei of OLAS tokens.

Instead of reverting when the payout > supply, the function should recalculate the payout so that payout == supply

```
   if (payout > supply) {
            // Have another calculation in here such that payout == supply
            // This way, users who want to deposit a large amount of LP tokens to get the remaining OLAS token supply will not get DoS'd easily
            // Another way is to have a minimum LP deposit, so users cannot DoS with 1 wei, but this will incur dust tokens
            // Dust tokens isn't an issue as the bond can be closed with the remaining supply returning back to the effectiveBond in Tokenomics contract
        }
```

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L319-L326

### [L-04] `payout` can still be non-zero when depositing LP tokens in Depository.sol

In `deposit()`, the comments state that payout cannot be zero since the price LP is non-zero

```
   // Calculate the payout in OLAS tokens based on the LP pair with the discount factor (DF) calculation
        // Note that payout cannot be zero since the price LP is non-zero, otherwise the product would not be created
        payout = IGenericBondCalculator(bondCalculator).calculatePayoutOLAS(tokenAmount, product.priceLP);
```

This is the calculation:

```
        uint256 totalTokenValue = mulDiv(priceLP, tokenAmount, 1);
        // Check for the cumulative LP tokens value limit
        if (totalTokenValue > type(uint192).max) {
            revert Overflow(totalTokenValue, type(uint192).max);
        }
        // Amount with the discount factor is IDF * priceLP * tokenAmount / 1e36
        // At this point of time IDF is bound by the max of uint64, and totalTokenValue is no bigger than the max of uint192
        amountOLAS = ITokenomics(tokenomics).getLastIDF() * totalTokenValue / 1e36;
```

If the price of the LP is below 1e18, and the tokenAmount is extremely small (in wei), the payout can be truncated to zero assuming that getLastIDF() returns 1e18.

totalTokenValue = 0.9e17 \* 1 / 1 = 0.9e17

amountOLAS = 1e18 \* 0.9e17 / 1e36 = 0

It is possible for the payout to return a non-zero number.

Check that payout is not zero before continuing the function.

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L320C77-L320C88

### [L-05] veOLAS allows for contracts to vote, but not veCRV

In veCRV, there is a check when creating a lock,

```
    self.assert_not_contract(msg.sender)
```

which means that smart contracts are not allowed to vote.

```
@internal
def assert_not_contract(addr: address):
    """
    @notice Check if the call is from a whitelisted smart contract, revert if not
    @param addr Address to be checked
    """
    if addr != tx.origin:
        checker: address = self.smart_wallet_checker
        if checker != ZERO_ADDRESS:
            if SmartWalletChecker(checker).check(addr):
                return
        raise "Smart contract depositors not allowed"
```

There is no such check in veOLAS.sol.

One possible reason to prevent smart contracts from participating is to reduce the attack vectors such as flashloan attacks.

To improve consistency of fork code, check for the contract existence size in veOLAS.sol as well.

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L425-L434

### [L-06] Users can create lock for others in veOLAS.sol, which opens up griefing capabilities

Users can call the `createLockFor()` function to create a lock for another account.

```
function createLockFor(address account, uint256 amount, uint256 unlockTime) external {
        // Check if the account address is zero
        if (account == address(0)) {
            revert ZeroAddress();
        }

        _createLockFor(account, amount, unlockTime);
    }
```

Griefers can create a lock for random users with 1 wei as the amount and 4 years as the unlock time to purposely grief users. Some genuine users of the OLAS protocol may not want to lock their tokens for 4 years, but a malicious griefer can frontrun their createLock function and create a lock for them with 4 years as the `unlockTime`. Since the unlockTime cannot be reduced after creation, the genuine users will be affected and have to create another account just to create a lock.

In the original veCRV implementation, there is no function that allows a user to create a lock for other users.

```
@external
@nonreentrant('lock')
def create_lock(_value: uint256, _unlock_time: uint256):
    """
    @notice Deposit `_value` tokens for `msg.sender` and lock until `_unlock_time`
    @param _value Amount to deposit
    @param _unlock_time Epoch time when tokens unlock, rounded down to whole weeks
    """
```

The only functionality in the veCRV implementation that allows user to help other users is the depositfor function.

It is best to remove the createLockFor function in veOLAS.sol to prevent any griefing capabilities. Just allow the msg.sender to create a lock for himself.

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L411-L418
