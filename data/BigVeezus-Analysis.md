## Approach
Though this is my first time submitting an audit in C4,
First thing I did to evaluate the codebase was run the files with static analysis tools and then tried to understand the code and what each of the folders containing contracts does, down to the functions.

## Review
It was a well detailed and very secure project, but I noticed a couple things/

### 1. Repetitive codes
For example, when writing a function that can only be called by the owner of the contract.
at **Depository.sol** line 125-127
```
 if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
```
This particular line of code was repeated several times in the codebase which is not best pratice.

Its best to use modifiers and call modifier just before the code runs.
```
  modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }

// then you pass it as a middleware with thr function as
 function changeManagers(address _tokenomics, address _treasury) external onlyOwner {
// do something here
}
```

It helps code re-usability.
It helps save gas in the long run.
It helps readability and clarity.

### 2. Unprotected owner functions.
functions that are protected to be accessed by owners are not protected from re-entrancy and other attacks probably because they can only be called by the owner of the contract, but this is still not the best practice.

All functions should be protected in anticipation of any attack both owner functions, admin etc.

Overall, the codebase is a B rating, quite secure and structured.

### Time spent:
5 hours