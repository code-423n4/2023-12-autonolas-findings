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

# [G-03] Using `mappings` instead of `arrays` saves gas; 

### Description:
Not only will arrays consume a lot of gas, but if gas prices rise too much, it can even prevent your contract from being carried out beyond the block gas limit.

```Solidity
address[] memory targets;
```

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2Fmultisigs%2FGuardCM.sol#L345

### Recommendation:
‍Instead of looping over an array until you locate the key you need, use mappings, which are hash tables that enable you to retrieve any value using its key in a single action.

# [G-04] Explicit zero_check for `amount` parameter in deposit function saves gas.

### Description:
If a user calls the function with a `zero-amount` parameter, the function will still execute, consuming gas. However, since no action is performed `(since the amount is zero)`, this execution is unnecessary and could lead to wasted gas. This is evident in the line `IERC20(token).transferFrom(msg.sender, address(this), amount)`; where the function attempts to transfer tokens even if the `amount is zero`.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L330-L369

### Recommendation:
> It's generally a good practice to include a require statement at the beginning of your function to ensure that the amount sent is not zero

# [G-05] Caching Array lengths in for-loops saves GAS.

### Description:
Caching the length of an array in a for loop can save gas because reading the array length at each iteration of the loop requires additional computational resources.

For memory arrays, an extra mload operation is performed, adding 3 extra gas for each iteration therfore caching it will save 3 GAS.

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics%2Fcontracts%2FDepository.sol#L356-L357

# [G-06] Declare variables outside the loop
Per iteration, saves 26 gas.

### Description:
Declaring variables outside a for loop helps save on gas costs because the variable does not need to be reallocated in each iteration of the loop, which can save computational resources and therefore reduce the gas cost.

Example:
```Solidity
contract Test {
   function testOutsideLoop() public {
       uint myNumber;
       for (uint i=0; i<10; i++) {
           myNumber = i*i;
       }
   }
}
```
### Instance:
This is violated in this context:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L561-L565

Where `uint256` mid is defined within the for-loop.


# [G-07] Use `! = 0` instead of `>0` for unsigned integer comparison.

###Description:
In Solidity, using `!= 0` instead of `> 0` in integer comparisons can be more gas-efficient. This is because the `!=` operator checks if a value is non-zero by simply checking the zero flag, which is more efficient in terms of gas cost compared to checking if a value is greater than zero using `>` 

When a comparison operator is used on `uint256` values, it requires the `Ethereum Virtual Machine (EVM)` to perform a subtraction operation between the two values, which takes up more gas than a simple check for equality. 
For example, if you want to check if a number is `greater than zero`, the `EVM` must subtract one from the number and compare the result to zero. This operation takes more computational resources and therefore consumes more gas than simply checking if the number is not equal to zero. 

### Instances:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance%2Fcontracts%2FveOLAS.sol#L663

### Recommendation:
Change the condition in this manner:
```Solidity
if(dBlock ! =0) {
```

