# Audit Gas Savings Report

## File: governance/contracts/multisigs/GuardCM.sol
***Suggested modifications are marked with //@audit***
### `setTargetSelectorChainIds` Function

```solidity
function setTargetSelectorChainIds(
    address[] memory targets,
    bytes4[] memory selectors,
    uint256[] memory chainIds,
    bool[] memory statuses
) external {
    // Check for the ownership
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    
    //@audit cache the lengths of all arrays used more than once;
    uint256 targetLength = targets.length;
    uint256 selectorLength = selectors.length;
    uint256 statusesLength = statuses.length;
    uint256 chainIdsLength = chainIds.length;
    
    // Check array length
    //@audit replace all array.length with their chached length
    if (targetLength != selectorLength || targetLength != statusesLength || targetLength != chainIdsLength) {
        revert WrongArrayLength(targetLength, selectorLength, statusesLength, chainIdsLength);
    }

    // Traverse all the targets and selectors to build their paired values
    for (uint256 i = 0; i < targetLength; ++i) {
        address targetAddress = targets[i];
        bytes4 selector = selectors[i];
        uint256 chainId = chainIds[i];

        // Check for zero address targets
        //@audit replaced targets[i] with targetAddress
        if (targetAddress == address(0)) { 
            revert ZeroAddress();
        }

        // Check selector for zero selector value
        //@audit replaced selectors[i] with selector
        if (selector == bytes4(0)) { 
            revert ZeroValue();
        }

        // Check chain Ids to be greater than zero
        // @audit replaced chainIds[i] with chainId
        if (chainId == 0) { 
            revert ZeroValue();
        }

        // Push a pair of key defining variables into one key
        // target occupies first 160 bits
        uint256 targetSelectorChainId = uint256(uint160(targetAddress));
        // selector occupies next 32 bits
        targetSelectorChainId |= uint256(uint32(selector)) << 160;
        // chainId occupies next 64 bits
        targetSelectorChainId |= chainId << 192;

        // Set the status of the target and selector combination
        mapAllowedTargetSelectorChainIds[targetSelectorChainId] = statuses[i];
    }

    emit SetTargetSelectors(targets, selectors, chainIds, statuses);
}
```
### `setBridgeMediatorChainIds` Function
```solidity
function setBridgeMediatorChainIds(
    address[] memory bridgeMediatorL1s,
    address[] memory bridgeMediatorL2s,
    uint256[] memory chainIds
) external {
    // Check for the ownership
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }

    //@audit cache length for used arrays
    uint256 bridgeMediatorL1sLength = bridgeMediatorL1s.length;
    uint256 bridgeMediatorL2sLength = bridgeMediatorL2s.length;   
    uint256 chainIdsLength = chainIds.length; 

    // Check for array correctness
    if (bridgeMediatorL1sLength != bridgeMediatorL2sLength || bridgeMediatorL1sLength != chainIdsLength) {
        revert WrongArrayLength(bridgeMediatorL1sLength, bridgeMediatorL2sLength, chainIdsLength, chainIdsLength);
    }

    // Link L1 and L2 bridge mediators, set L2 chain Ids
    for (uint256 i = 0; i < chainIdsLength; ++i) {

        //@audit cache addresses read from the array more than once
        address bridgeMediatorL1TargetAddress = bridgeMediatorL1s[i];
        address bridgeMediatorL2TargetAddress = bridgeMediatorL2s[i];

        // Check for zero addresses
        if (bridgeMediatorL1TargetAddress == address(0) || bridgeMediatorL2TargetAddress == address(0)) {
            revert ZeroAddress();
        }

        // Check supported chain Ids on L2
        uint256 chainId = chainIds[i];
        if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001) {
            revert L2ChainIdNotSupported(chainId);
        }

        // Push a pair of key defining variables into one key
        // bridgeMediatorL2 occupies first 160 bits
        uint256 bridgeMediatorL2ChainId = uint256(uint160(bridgeMediatorL2TargetAddress));
        // L2 chain Id occupies next 64 bits
        bridgeMediatorL2ChainId |= chainId << 160;
        mapBridgeMediatorL1L2ChainIds[bridgeMediatorL1TargetAddress] = bridgeMediatorL2ChainId;
    }

    emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);
}
```

| Solc version | Optimizer enabled | Runs     | Block limit           |
|--------------|-------------------|----------|-----------------------|
| 0.8.23       | true              | 1000000  | 30000000 gas          |

| Contract  | Min       | Max       | Avg       | % of limit | When        | Gas savings |
|-----------|-----------|-----------|-----------|------------|-------------|------|
| GuardCM   | 2708197   | 2708221   | 2708217   | 9%         | No edits    | - |
| GuardCM   | 2696370   | 2696394   | 2696390   | 9%         | setTargetSelectorChainIds edited  | 11827 gas saved|
| GuardCM   | 2675617   | 2675641   | 2675637   | 8.9%       | Both functions edited | 32580 gas saved|

By making tweaks to the setTargetSelectorChainIds function, we managed to save around 11827 units of gas. After working on the optimizations for the setBridgeMediatorChainIds function as well, the overall gas savings increased to 32580 units.
