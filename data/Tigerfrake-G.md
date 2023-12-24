# [G-01] Use `constants` instead of `type(uintX).max`

### Description:
Using constants instead of `type(uintX).max` saves gas in Solidity. This is because the `type(uintX).max` function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. 
`Constants`, on the other hand, are stored in the `bytecode` of your contract, so they do not have to be recalculated every time you need them. 
Saves 13 GAS

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L131

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

