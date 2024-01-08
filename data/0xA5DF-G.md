
| ID  | Title                                                       | Gas saved |
|-----|-------------------------------------------------------------|-----------|
| G1  | A more efficient way to assert that this is a delegate call | 2100      |
| G2  | Token1 isn’t used in some case                              | 2400      |
| G3  | Use assembly to trim arrays                                 | 10800     |
| G4  | Add the unit ID at `_calculateSubComponents()`              | 6000      |
| G5  | Assign only the required field from storage to memory       | 2100      |
| SUM |                                                             | 23,400    |

## [G1] A more efficient way to assert that this is a delegate call
Gas saved: 2.1K per tx

Current check to ensure a delegate-call-only uses an sload which costs ~2.1K per tx.
There's a much simpler and cheaper way to do that, see [OZ's docs](https://docs.openzeppelin.com/upgrades-plugins/1.x/faq#delegatecall-selfdestruct)


[Current code](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L883-L889)

```solidity
        assembly {
            implementation := sload(PROXY_TOKENOMICS)
        }
        // Check if there is any address in the PROXY_TOKENOMICS address slot
        if (implementation == address(0)) {
            revert DelegatecallOnly();
        }
```

OZ's code:
```solidity
abstract contract OnlyDelegateCall {
    /// @custom:oz-upgrades-unsafe-allow state-variable-immutable
    address private immutable self = address(this);

    function checkDelegateCall() private view {
        require(address(this) != self);
    }

    modifier onlyDelegateCall() {
        checkDelegateCall();
        _;
    }
}
```



## [G2] Token1 isn’t used in some case
Gas saved: 2.4 per tx (sload + call)

In `getCurrentPriceLP()` `token1` isn't needed when `token0 == olas`.
Not reading that can save us an `sload` to a cold key inside the call + the call itself.

[Code](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L74C1-L91C10)
```solidity
        if (totalSupply > 0) {
            address token0 = pair.token0();
            address token1 = pair.token1();
            uint256 reserve0;
            uint256 reserve1;
            // requires low gas
            (reserve0, reserve1, ) = pair.getReserves();
            // token0 != olas && token1 != olas, this should never happen
            if (token0 == olas || token1 == olas) {
                // If OLAS is in token0, assign its reserve to reserve1, otherwise the reserve1 is already correct
                if (token0 == olas) {
                    reserve1 = reserve0;
                }
                // Calculate the LP price based on reserves and totalSupply ratio multiplied by 1e18
                // Inspired by: https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy#L262
                priceLP = (reserve1 * 1e18) / totalSupply;
            }
        }
```

## [G3] Use assembly to trim arrays
Gas saved: 10.8K = ~300 units per element * a sum 36 elements on avg 

See [PoC](https://gist.github.com/0xA5DF/08f8d8e0d91d7cfd95cdbcca9911c200)
I tested the PoC with 20 elements, it saved about 6K gas units.

### GuardCM

There are a few cases in `GuardCM` where we trim an array by copying it to a new array.
However, this can be done by assembly, given that the old array isn't used anymore in the code.

Estimated avg # of elements: 12 (min is 8 according to the code)




[GuardCM.sol#L273-L275](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L273-L275)
```solidiy
            for (uint256 i = 0; i < payload.length; ++i) {
                payload[i] = data[i + 4];
            }
```

[GuardCM.sol#L291-L294](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L291-L294)
```solidity
            bytes memory bridgePayload = new bytes(mediatorPayload.length - SELECTOR_DATA_LENGTH);
            for (uint256 i = 0; i < bridgePayload.length; ++i) {
                bridgePayload[i] = mediatorPayload[i + SELECTOR_DATA_LENGTH];
            }
```

[GuardCM.sol#L317-L320](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L317-L320)

```solidity
            bytes memory payload = new bytes(data.length - SELECTOR_DATA_LENGTH);
            for (uint256 i = 0; i < payload.length; ++i) {
                payload[i] = data[i + SELECTOR_DATA_LENGTH];
            }
```

[GuardCM.sol#L339-L342](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L339-L342)

```solidity
        bytes memory payload = new bytes(data.length - SELECTOR_DATA_LENGTH);
        for (uint256 i = 0; i < payload.length; ++i) {
            payload[i] = data[i + 4];
        }
```

### GnosisSafeMultisig

This one has one instance
Estimated average # of elements: 4


[GnosisSafeMultisig.sol#L79-L81](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/multisigs/GnosisSafeMultisig.sol#L79-L81)

```solidity
                for (uint256 i = 0; i < payloadLength; ++i) {
                    payload[i] = data[i + DEFAULT_DATA_LENGTH];
                }
```

### UnitRegistry

Here we're not trimming the beginning of the array but the end of it, but the same optimization can be applied.
Estimated average # of elements: 20 

[UnitRegistry.sol#L261-L264](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L261-L264)
```solidity
        subComponentIds = new uint32[](counter);
        for (uint32 i = 0; i < counter; ++i) {
            subComponentIds[i] = allComponents[i];
        }
```

About this instance I'm not 100% sure it's safe since it's adding an element, so I won't count that (see an alternative way to save the gas here in the next section)

[UnitRegistry.sol#L95-L104](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L95-L104)

```solidity
        if (unitType == UnitType.Component) {
            uint256 numSubComponents = subComponentIds.length;
            uint32[] memory addSubComponentIds = new uint32[](numSubComponents + 1);
            for (uint256 i = 0; i < numSubComponents; ++i) {
                addSubComponentIds[i] = subComponentIds[i];
            }
            // Adding self component Id
            addSubComponentIds[numSubComponents] = uint32(unitId);
            subComponentIds = addSubComponentIds;
        }
```

## [G4] Add the unit ID at `_calculateSubComponents()`
Gas saved: 6K (300 per element * 20 elements on avg)

Instead of copying the array after it was created by `_calculateSubComponents()` it'd be much more efficient to refactor the code so that `_calculateSubComponents()` would add it on it's own when needed.


[UnitRegistry.sol#L95-L104](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L95-L104)

```solidity
        if (unitType == UnitType.Component) {
            uint256 numSubComponents = subComponentIds.length;
            uint32[] memory addSubComponentIds = new uint32[](numSubComponents + 1);
            for (uint256 i = 0; i < numSubComponents; ++i) {
                addSubComponentIds[i] = subComponentIds[i];
            }
            // Adding self component Id
            addSubComponentIds[numSubComponents] = uint32(unitId);
            subComponentIds = addSubComponentIds;
        }
```


## [G5] Assign only the required field from storage to memory
Gas saved: 2.1K (sload to a cold slot)

This one is similar to G40 in the bot report, but it wasn't caught by it.
The Unit struct contains also the `unitHash` field, but this one isn't used here.
Therefore, it'd be cheaper to make `unit` a storage variable, this way we aren't reading the `unitHash` field from storage.

[UnitRegistry.sol#L163](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/UnitRegistry.sol#L163)

```solidity
    function getDependencies(uint256 unitId) external view virtual
        returns (uint256 numDependencies, uint32[] memory dependencies)
    {
        Unit memory unit = mapUnits[unitId];
        return (unit.dependencies.length, unit.dependencies);
    }
```