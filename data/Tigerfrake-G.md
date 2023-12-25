# [G-01] Use `constants` instead of `type(uintX).max`

### Description:
Using constants instead of `type(uintX).max` saves gas in Solidity. This is because the `type(uintX).max` function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. 
`Constants`, on the other hand, are stored in the `bytecode` of your contract, so they do not have to be recalculated every time you need them. 
Saves 13 GAS

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L131
- https://github.com/code-423n4/2023-12-autonolas/blob/main/registries%2Fcontracts%2FUnitRegistry.sol#L232

# [G-02] Splitting `require()/if()` statements that use && saves gas. 

### Description:
This [issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128)  describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 13 gas.

### Instance:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L188

### Recommendation:
> Using double `if/require` checks can save more gas

# [G-03] Avoid looping through lengthy arrays; 

### Description:
Not only will it consume a lot of gas, but if gas prices rise too much, it can even prevent your contract from being carried out beyond the block gas limit.

```Solidity
address[] memory targets;

for (uint i = 0; i < targets.length; ++i) {
```

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2Fmultisigs%2FGuardCM.sol#L345
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2Fmultisigs%2FGuardCM.sol#L360

### Recommendation:
‍Instead of looping over an array until you locate the key you need, use mappings, which are hash tables that enable you to retrieve any value using its key in a single action.

# [G-04] Explicit zero_check for `amount` parameter in deposit function saves gas.

### Description:
If a user calls the function with a `zero-amount` parameter, the function will still execute, consuming gas. However, since no action is performed `(since the amount is zero)`, this execution is unnecessary and could lead to wasted gas. This is evident in the line `IERC20(token).transferFrom(msg.sender, address(this), amount)`; where the function attempts to transfer tokens even if the `amount is zero`.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L330-L369

### Recommendation:
> It's generally a good practice to include a require statement at the beginning of your function to ensure that the amount sent is not zero