# [01] Violation of Constant Variable Naming Conventions

### Description:

In Solidity, constant variables should be named using all capital letters with underscores separating words. This is a widely accepted convention for naming constants in Solidity and many other programming languages. Examples of such constant variable names could be `MAX_BLOCKS`

```Solidity 
uint8 constant public MAX_BLOCKS = 100;

```
### Instances:
This is violated in the following contexts.
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L22-L26


# [02] Potential Misleading Information Due to Unconditional Return Value in `DecreaseAllowance()` Function

### Description:
The `decreaseAllowance()` function  doesn't inherently check whether the transaction is successful as per the comment by @auther. 
```Solidity
/// @return True if operation succeeded. 
```

It performs the operation of reducing the allowance of a spender and returns `true` regardless of whether the operation was successful or not.

### Impact:
As for the impact on listeners that depend on the emitted `Approval` event, this can potentially lead to misleading information. Listeners that rely on the `Approval` event might interpret the event as a confirmation that the decrease in allowance was successful, even if it wasn't. This could lead to incorrect assumptions and behaviors in the rest of the application or in other parts of the smart contract.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L128-L138

# [03] Contradictory Use of `Override` and `Virtual` Keywords in Function Specifier

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

# [04] Missing Zero- address Validation

### Description:
Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L155-L157

### Recommendation
> Consider adding explicit zero-address validation on input parameters of address type.

# [05] Possibility of long-term suspension in `GuardCM` contract.

### Description:
If the pause feature is not designed with a mechanism to resume normal operation, it could potentially remain paused indefinitely.
```Solidity
function pause() external {
  // ...
}

function unpause() external {
  // ...
}
```
Here, once the contract is `paused`, there's no guarantee it will ever be `unpaused`. If the `unpause()` function is not called, the contract remains `paused`.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2Fmultisigs%2FGuardCM.sol#L539-L569

### Recommendation:
> It's recommended to use established patterns or libraries, such as OpenZeppelin's Pausable contract, which have been vetted by the community

# [06] Floating pragma

#### Description:
The disadvantage of using a floating pragma is that it can lead to inconsistencies and potential bugs. When a contract is compiled with a different compiler version, it might produce different bytecode due to changes in the compiler, even if the source code looks the same. This can lead to unexpected behavior if the contract is deployed on a network with a different compiler version than the one it was tested with.

```Solidity
pragma solidity ^0.4.0;

contract PragmaNotLocked {
   uint public x = 1;
}
```
Moreover, if the contract uses features that were introduced in a newer compiler version, and it gets compiled with an older compiler version, it could fail or behave differently than expected.

#### Instances:
- All files

#### Recommendation:
> It's generally recommended to lock the compiler version to avoid these issues.
```Solidity
pragma solidity 0.4.25;

contract PragmaFixed {
   uint public x = 1;
}
```

# [07] For same condition checks, use modifiers.

#### Description:
The main advantage of using modifiers for the same condition checks in different functions is code reusability and readability. 
Instead of repeating the same condition check in every function that requires it, you can define a modifier once and then apply it to any function that needs it. This makes your code cleaner and easier to maintain.

#### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L43-L46
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L58-L61
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FOLAS.sol#L75-L79

#### Recommendation:
An `onlyOwner()` modifier would have been defined for `changeOwner()`, `changeMinter()` & `mint()` fucntions in `OLAS` contract. 

```Solidity
modifier onlyOwner() {
   require(msg.sender == owner);
   _;
}
```

# [08] Potential Out of Gas Exception Due to Long `accounts` Array

#### Description:
The contract has an issue in its `setDonatorsStatuses()` function where it does not
limit the length of the `accounts` array when it is initialized. 
This could potentially
lead to an `Out of Gas (OOG)` exception if the `accounts` array becomes excessively long. Therefore, a `Denial of Service` for all the functionalities of the protocol.

#### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics%2Fcontracts%2FDonatorBlacklist.sol#L56-L77

#### Recommendation:
> To mitigate this issue, it is recommended to add a check in the `setDonatorsStatuses()` 
functions to ensure that the length of the `accounts` array does not exceed a certain limit. This limit should be set to a reasonable value to prevent the array from becoming excessively
long. If the length of the `accounts` array exceeds this limit, the function should `revert` with
an appropriate error message. This will prevent potential `Out of Gas (OOG)`


# [09] Precision Loss When Dividing Odd Integers by Two

#### Description:
The contract has a flaw where it may lose precision when dividing `odd integers` by `two`. This
is because in Solidity, `integer division is floor division`, meaning that the result of the division operation will be the `largest integer less than or equal to the exact result`. Therefore,
when an `odd integer` is divided by `two`, the result will be `rounded down`, leading to a `loss of
precision`.

#### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L565

#### Recommendation:
When dividing an amount by `two`, consider taking the first amount as the division result by
two, and the second one to be the total amount minus the first one.