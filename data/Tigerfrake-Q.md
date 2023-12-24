# [01] Violation of Constant Variable Naming Conventions

### Description:

In Solidity, constant variables should be named using all capital letters with underscores separating words. This is a widely accepted convention for naming constants in Solidity and many other programming languages. Examples of such constant variable names could be `MAX_BLOCKS`

```Solidity 
uint8 constant public MAX_BLOCKS = 100;

```
### Contexts:
This is violated in the following contexts.
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L22-L26

# [02] Use a two-step transfer function for the Owner and Minter roles in OLAS contract. 

### Description:
The `OLAS` contract has a very important `Minter` role which can be changed via a `changeMinter()` function and `Owner` role that can be changed via `changeOwner()` function.

In order to prevent the role from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), require that the recipient of the new `minter` or `owner` role actively accepts via a `contract call` of its own.

### Contexts:

- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L43-L69

