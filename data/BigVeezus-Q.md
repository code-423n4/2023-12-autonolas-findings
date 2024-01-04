# ERC20 Race-condition vulnerability

ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard. To account for this, either use OpenZeppelin's SafeERC20 library or wrap each operation in a require statement.
Additionally, ERC20's approve functions have a known race-condition vulnerability. To account for this, use OpenZeppelin's SafeERC20 library's `safeIncrease` or `safeDecrease` Allowance functions.

### Depository.sol:390
```
 IToken(olas).transfer(msg.sender, payout);
```
can be written as
```
IToken(olas).safeTransfer(msg.sender, payout);
or
//check for success
bool success = IERC20(token).transfer(msg.sender, address(this), amount);
require(success, "ERC20 transfer failed");
```