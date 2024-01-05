Context: [Treasury.sol#L212](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L212)
```solidity
    function depositTokenForOLAS(address account, uint256 tokenAmount, address token, uint256 olasMintAmount)
```

The bot report states the "Missing zero address check in functions with address parameters" but not that the tokenAmount and olasMintAmount can be zero.

The `uint256 tokenAmount` and the `uint256 olasMintAmount` parameters can be 0. This can lead to issues.

```diff
    function depositTokenForOLAS(address account, uint256 tokenAmount, address token, uint256 olasMintAmount) external {
+       require(tokenAmount > 0, "Token amount must be greater than zero");
+       require(olasMintAmount > 0, "OLAS mint amount must be greater than zero");

        // Check for the depository access
        if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }
        ...
```