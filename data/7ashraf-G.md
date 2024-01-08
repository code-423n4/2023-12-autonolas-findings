# Gas Report
## Summary

| **Gas**   | **Title**                              | **Number of Instances** |
|-----------|----------------------------------------|--------------------------|
| *G-01*   | Don't Cache Variables If Used Once     | 4                        |
| *G-02*   | Reduce Code Caching                   | 1                        |
| *G-03*   | Shorthand Operation Usage             | 2                        |
| *G-04*   | Avoid On-chain Calculations           | 2                        |
| *G-05*   | No Need to Zero Value Check            | 1                        |
| *G-06*   | Unused Boolean Return Value           | 1                        |

## G-01 Don't cache variables if gonna be used only once
### Instances
* [OLAS.sol #92](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L92)
```solidity
        uint256 remainder = inflationRemainder();
        return (amount <= remainder);
```
* [OLAS.sol #99](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L99)
```solidity
        uint256 _totalSupply = totalSupply;
```
* [UnitRegistry.sol #164](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L164)
```solidity
function getDependencies(uint256 unitId) external view virtual
        returns (uint256 numDependencies, uint32[] memory dependencies)
    {
>>        Unit memory unit = mapUnits[unitId];
>>        return (unit.dependencies.length, unit.dependencies);
    }
```
* [UnitRegistry.sol #174](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L174)
```solidity
        unitHashes = mapUnitIdHashes[unitId];
        return (unitHashes.length, unitHashes);
```

### Mitigation
`        return (amount <= inflationRemainder(););
`

## G-02 Reduce code caching to save gas
the following mathematical operations code be done in one operation 

### Instances
veOlas.sol #510
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

## G-03 Using shorthand operation costs more gas 
using the `-=` opertaion costs more gase than `foo = foo - bar`
### Instances
veOlas.sol #556
```solidity

```
* [veOLAS.sol #597](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L597)
```solidity
            uPoint.bias -= uPoint.slope * int128(int64(ts) - int64(uPoint.ts));

```
* [veOLAS.sol #680](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L680)
```solidity
        uPoint.bias -= uPoint.slope * int128(int64(uint64(blockTime)) - int64(uPoint.ts));

```


## G-04 Avoid on-chain calculations if results are known
### Instances
* [veOLAS.sol #101](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L101)
```solidity
    uint256 internal constant MAXTIME = 4 * 365 * 86400;

```
* [veOLAS.sol #103](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L103)
```solidity
    int128 internal constant IMAXTIME = 4 * 365 * 86400;

```

## G-05 No need to zero value check
The variable is already set to cache the value `pay` which was already checked for zero value.
 
### Instances
* [Despository.sol #384](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L384)
```solidity
 function redeem(uint256[] memory bondIds) external returns (uint256 payout) {
        for (uint256 i = 0; i < bondIds.length; ++i) {
            // Get the amount to pay and the maturity status
            uint256 pay = mapUserBonds[bondIds[i]].payout;
            bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;

            // Revert if the bond does not exist or is not matured yet
            @audit check is done here 
            if (pay == 0 || !matured) {
                revert BondNotRedeemable(bondIds[i]);
            }

            // Check that the msg.sender is the owner of the bond
            if (mapUserBonds[bondIds[i]].account != msg.sender) {
                revert OwnerOnly(msg.sender, mapUserBonds[bondIds[i]].account);
            }

            // Increase the payout
            payout += pay;

            // Get the productId
            uint256 productId = mapUserBonds[bondIds[i]].productId;

            // Delete the Bond struct and release the gas
            delete mapUserBonds[bondIds[i]];
            emit RedeemBond(productId, msg.sender, bondIds[i]);
        }

        // Check for the non-zero payout
        if (payout == 0) {
            revert ZeroValue();
        }

        // No reentrancy risk here since it's the last operation, and originated from the OLAS token
        // No need to check for the return value, since it either reverts or returns true, see the ERC20 implementation
        IToken(olas).transfer(msg.sender, payout);
    }
```
## G-06 The boolean return value has no use
### Description
The boolean return value `success` indicating the function successful behaviour has no use and can be neglected as the function reverts on failure, and there is no case where the function does not revert and `success` is set to false.
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
