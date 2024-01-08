# Quality Assurance Report
##Summary
The code review reveals key issues in the provided smart contract repository. These include the necessity to verify the existence of `unitId` before updating its hash and placing emits within if-statements to prevent false emissions. Other recommendations involve reverting on non-existent bonds, avoiding duplicate token names and symbols, and checking for zero addresses in constructors. The review suggests using modifiers for cleaner code, fixing comment typos, and streamlining redundant mathematical operations for efficiency. These insights offer a concise guide for enhancing the security and clarity of the smart contract implementation.

| Issue    | Title                                                                   | Number of Instances |
|----------|-------------------------------------------------------------------------|---------------------|
| L-01     | Check existence of `unitId` in `mapUnitIdHashes` before updating hash   | 5                   |
| L-02     | Move `Depost` emit inside if-statement to avoid false emits              | 1                   |
| L-03     | Revert on non-existent bonds                                           | 1                   |
| L-04     | Avoid duplicate token names and symbols in constructor                  | 2                   |
| L-05      | Funds owned by the contract cannot be sweeped or transferred           | 1                   |
| L-06     | Inaccurate error message                                               | 1                   |
| L-07     | Non-owner can burn tokens by transferring to address 0                 | 1                   |
| L-08     | Assert `to` and `amount` greater than zero when minting                | 1                   |
| L-09     | Add return values for `mint` and `burn`                                | 2                   |
| L-10     | Check if already un/paused before un/pausing                          | 2                   |
| N-01     | Utilize the use of modifiers                                           | -                   |
| N-02     | Fix typo in comments                                                   | 1                   |
| N-03     | Consider adding remaining time in revert message                      | 1                   |
| N-04     | Reduce redundant code                                                  | 1                   |
| N-05     | Inaccurate variable naming                                             | 2                   |
| N-06     | Function `create` should emit an event                                 | 1                   |
| N-07     | Boolean return value has no use                                        | 1                   |
| N-08     | Utilize the use of the already defined error `ZeroAddress`             | 2                   |
| N-09     | Utilize the use of Non-reentrant modifier for cleaner code             | 1                   |
| N-10     | Check if new implementation address does not equal current address    | 1                   |
| N-11     | Emit an event for increasing unlock time                               | 2                   |


These findings provide actionable insights for enhancing the overall quality and robustness of the smart contract codebase.





## [L-01] Check if `unitId` exists inside the  `mapUnitIdHashes[unitId]` before updating its hash
### Instances
* [UnitRegistry.sol #143](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L143)
```solidity
        mapUnitIdHashes[unitId].push(unitHash);

```
* [UnitRegistry.sol #163](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L163)
```solidity
        Unit memory unit = mapUnits[unitId];
```
* [UnitRegistry.sol #174](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L174)
```solidity
        unitHashes = mapUnitIdHashes[unitId];
```
* [UnitRegistry.sol #185](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L185)
```solidity
        subComponentIds = mapSubComponents[uint256(unitId)];
```
* [RegistriesManager.sol #50](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L50)
```solidity
function updateHash(IRegistry.UnitType unitType, uint256 unitId, bytes32 unitHash) external returns (bool success) {
        if (unitType == IRegistry.UnitType.Component) {
            success = IRegistry(componentRegistry).updateHash(msg.sender, unitId, unitHash);
        } else {
            success = IRegistry(agentRegistry).updateHash(msg.sender, unitId, unitHash);
        }
    }
```
## [L-02] Move Depost emit inside if-statment to avoid false emits

### Instances
* [veOLAS.sol #367](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367)
```solidity
if (amount > 0) {
            // OLAS is a solmate-based ERC20 token with optimized transferFrom() that either returns true or reverts
            IERC20(token).transferFrom(msg.sender, address(this), amount);
+           emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
+           emit Supply(supplyBefore, supplyAfter);
        }

-        emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
-        emit Supply(supplyBefore, supplyAfter);
```



## [L-03] should revert with an error on non-existent bonds
### Instances
* [Depository.sol #480](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L480)
```solidity
480    function getBondStatus(uint256 bondId) external view returns (uint256 payout, bool matured) {
481        payout = mapUserBonds[bondId].payout;
482        // If payout is zero, the bond has been redeemed or never existed
483        if (payout > 0) {
484            matured = block.timestamp >= mapUserBonds[bondId].maturity;
485        }
486    }
```

## [L-04] Dont allow duplicate token names and symbols inside constructor
### Instances
* [ComponentRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol)
* [AgentRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol)


## [L-05] Funds owned by the contract cannot be sweeped or transferred
The scenario happens if a user decides to create lock funds using the contract address, at the end no one will be able to withdraw the funds, and they will be locked.
Consider adding a sweep tokens function to the contract
### Instances
* [veOLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol)





## [L-06] Inaccurate error message
* Error message is product closed, which will be emitted for never existing prodcuts.
this gives in accurate message about the product
### Instances
* [Depository.sol #304](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L304)
```solidity
        if (supply == 0) {
            revert ProductClosed(productId);
        }
```
### Mitigation
* Check for prodcuct == 0 for prodcut does not exist



## [L-07] Non owner can burn tokens by transferring to address 0
### Instances
* [BridgedERC20.sol #59](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L59)
### Mitigation
* Override the transfer method to revert 

## [L-08] Assert `to` and `amount` greater than zero when minting
### Instances
* [FxERC20RootTunnel.sol #77 ](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L77)
```solidity
        IERC20(rootToken).mint(to, amount);
```

## [L-09] Add return values for `mint` and `burn`
### Instances
* [IERC20.sol #43](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L43)
* [IERC20.sol #47](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L47)
```solidity
 /// @dev Mints tokens.
    /// @param account Account address.
    /// @param amount Token amount.
    function mint(address account, uint256 amount) external;

    /// @dev Burns tokens.
    /// @param amount Token amount to burn.
    function burn(uint256 amount) external;
```


## [L-10] Check if already un/paused before un/pausing
Check the state of contract if already pasued or not before pausing or not pausing 
### Instance
* [GenericManager.sol #36](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L36)

* [GenericManager.sol #47](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L47)
```solidity
    /// @dev Pauses the contract.

 function pause() external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        paused = true;
        emit Pause(msg.sender);
    }

    /// @dev Unpauses the contract.
    function unpause() external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        paused = false;
        emit Unpause(msg.sender);
    }
```

## [N-01] Utilize the use of modifiers 
Protocol can achieve cleaner code by following separation of concerns and encapsulation priniciple instead of applying the same checks on msg.sneder each methdod, we can use a modifier like `onlyOwner` or `onlyManager` and appply the needed checks inside these modifiers.

## [N-02] Fix typo in comments
### Instances
* [OLAS.sol #96](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L96)
```
- reminder
+ remainder
```

## [N-03] Consider adding remaining time in revert message
### Instances
* [veOLAS.sol #513](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L513)
## [N-04] Reduce redundunt code
the following mathematical operations code be done in one operation 
### Instances
* [veOLAS.sol #510](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L510)

```js
 uint256 supplyBefore = supply;
        uint256 supplyAfter;
        // The amount cannot be less than the total supply
        unchecked {
            supplyAfter = supplyBefore - amount;
            supply = supplyAfter;
        }
```

to
```js
unchecked{
    supply = supply - amount;
}

...
        emit Supply(supply + amount, supply - amount);

```

## [N-05] InAccurate variable naming 
### Instances
* [IErros.sol #32](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IErrors.sol#L32) should be `token` instead of `account`
```solidity
 /// @dev Token is non-transferable.
    /// @param account Token address.
    error NonTransferable(address account);

```
* [IErros.sol #36](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IErrors.sol#L36) should be `token` instead of `account`
```solidity
/// @dev Token is non-delegatable.
    /// @param account Token address.
    error NonDelegatable(address account);
```



## [N-06] Function `create` should emit an event
### Instances
* [GnosisSafeSameAddressMultisig.sol #85]()
## [N-07] The boolean return value has no use
### Description
The boolean return value `success` indicating the function successful behaviour has no use and can be neglected as the function reverts on failure, and there is no cas where the function does not revert and `success` is set to false.
### Instances
* [UnitRegistry.sol #121](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L121)
```solidity
function updateHash(address unitOwner, uint256 unitId, bytes32 unitHash) external virtual
        returns (bool success)
    {
        ...
        success = true;

        emit UpdateUnitHash(unitId, unitType, unitHash);
    }
```
## [N-08]Utilize the use of the already defined error `ZeroAddress()`
The contract defines an error which is called `ZeroAddress()` in the following lines
```js
IErrorsTokenomics.sol #17    error ZeroAddress();

```
but when comes to checking for zero address, it uses exipicit checking for zero addresses
### Instances
* [Depository.sol #143](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L143)
* [Depository.sol #163](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L163)

### Mitigation
```js
- if (_bondCalculator != address(0)) {
-            bondCalculator = _bondCalculator;
-            emit BondCalculatorUpdated(_bondCalculator);
-        }

+ if (_bondCalculator == address(0)) {
+            revert ZeroAddress();
+        }
+        emit BondCalculatorUpdated(_bondCalculator);

```



## [N-09]Utilize the use of Non-reentrant modifire for a cleaner code
### Instances
* [Dispenser.sol #93](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L93)


## [N-010]Check if new implementation address does not equal current address
### Instances
* [Tokenomics.sol #384](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L384)


## [N-11]Consider emiting an event for this function
On increasing unlock time, function should emit an event with the parameters including account, previous lock time, and new lock time 
### Instances
* [veOLAS.sol #483](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L483)
* [veOLAS.sol #458](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L458)

