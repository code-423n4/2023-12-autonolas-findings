# GAS OPTIMIZATIONS

##

## [G-1] ``manager`` and ``_locked`` state variables can be packed within same slot : Saves ``2000 GAS`` , ``1 SLOT``

_locked can be a uint8 instead of a uint256. It only stores the values 1 and 2 because the _locked variable is used solely for the Reentrancy lock. Therefore, using uint8 is perfectly safe and will not affect the protocol.

```diff
FILE: 2023-12-autonolas/registries/contracts/GenericRegistry.sol

// Owner address
    address public owner;
    // Unit manager
    address public manager;
+    // Reentrancy lock
+    uint8 internal _locked = 1;
    // Base URI
    string public baseURI;
    // Unit counter
    uint256 public totalSupply;
    // Reentrancy lock
-    uint256 internal _locked = 1;

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L14-L23

##

## [G-2] 

