# [G-10] Use `constants` instead of `type(uintX).max`

### Description:
Using constants instead of `type(uintX).max` saves gas in Solidity. This is because the `type(uintX).max` function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. 
`Constants`, on the other hand, are stored in the `bytecode` of your contract, so they do not have to be recalculated every time you need them. 
Saves 13 GAS

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L131