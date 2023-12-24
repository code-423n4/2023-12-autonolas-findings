# [01] Violation of Constant Variable Naming Conventions

### Description:

In Solidity, constant variables should be named using all capital letters with underscores separating words. This is a widely accepted convention for naming constants in Solidity and many other programming languages. Examples of such constant variable names could be `MAX_BLOCKS`

```Solidity 
uint8 constant public MAX_BLOCKS = 100;

```
### Instances:
This is violated in the following contexts.
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L22-L26

# [02] Use a two-step transfer function for the Owner and Minter roles in OLAS contract. 

### Description:
The `OLAS` contract has a very important `Minter` role which can be changed via a `changeMinter()` function and `Owner` role that can be changed via `changeOwner()` function.

In order to prevent the role from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), require that the recipient of the new `minter` or `owner` role actively accepts via a `contract call` of its own.

### Instances:

- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L43-L69


# [03] Potential Misleading Information Due to Unconditional Return Value in `DecreaseAllowance()` Function

### Description:
The `decreaseAllowance()` function  doesn't inherently check whether the transaction is successful. It performs the operation of reducing the allowance of a spender and returns `true` regardless of whether the operation was successful or not.

### Impact:
As for the impact on listeners that depend on the emitted `Approval` event, this can potentially lead to misleading information. Listeners that rely on the `Approval` event might interpret the event as a confirmation that the decrease in allowance was successful, even if it wasn't. This could lead to incorrect assumptions and behaviors in the rest of the application or in other parts of the smart contract.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L128-L138

# [04] Contradictory Use of `Override` and `Virtual` Keywords in Function Specifier

### Description:
In Solidity, a function cannot have both virtual and override specifiers at the same time. They serve different purposes and are used in different scenarios:

The `virtual` keyword is used in the base contract to indicate that a function can be overridden by derived contracts. If a function is marked as virtual, it means that the function's behavior can be changed in derived contracts.

The `override` keyword is used in the derived contract to indicate that the function is intended to override a function in the base contract. 

### Impact:
If a function has both `override` and `virtual` specifiers, it implies a contradiction: the function is both overriding a function in a base contract and allowing itself to be overridden by derived contracts. This contradictory behavior is not allowed in Solidity, and the compiler will throw an error.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L761
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L767
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L772