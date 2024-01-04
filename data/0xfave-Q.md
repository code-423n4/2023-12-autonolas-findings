The core function mint is used Mints bridged tokens. Address(0) check is missing in both this function and the internal function _mint, which is triggered to mint the tokens to the to address. Consider applying a check in the function to ensure tokens aren’t minted to the zero address.

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/BridgedERC20.sol#L48-L55

```solidity
function mint(address account, uint256 amount) external {
        // Only the contract owner is allowed to mint
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        
        _mint(account, amount);
    }
```