# [01] Violation of Constant Variable Naming Conventions

### Description:

In Solidity, constant variables should be named using all capital letters with underscores separating words. This is a widely accepted convention for naming constants in Solidity and many other programming languages. Examples of such constant variable names could be `MAX_BLOCKS`

```Solidity 
uint8 constant public MAX_BLOCKS = 100;

```
### Contexts:
This is violated in the following contexts.
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L22-L26
