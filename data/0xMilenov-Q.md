# OLAS QA Report 

### [L-01] Arrays can grow in size without a way to shrink them  

As these arrays cannot shrink, if the array has a maximum size, it won't be possible to change its elements once it reaches that size. Otherwise, it can grow indefinitely in size, which can increase the likelihood of out-of-gas errors.

[veOLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L315)
```solidity File: 
 mapUserPoints[account].push(uNew);
```

[UnitRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L143)
```
 mapUnitIdHashes[unitId].push(unitHash);`
```


### [L-02] Solidity 0.8.20's reliance on PUSH0 could pose compatibility issues on alternate chains

Using Solidity 0.8.20's new PUSH0 opcode can cause deployment issues on some L2s due to incompatibility. To work around this issue, use an earlier version.
### [L-03] Known vulnerabilities in current @openzeppelin/contracts version

Given the presence of known vulnerabilities in the current @openzeppelin/contracts version, it is advisable to update to at least `@openzeppelin/contracts@5.0.1` to address these issues and enhance the contract's security.

```solidity
File: package.json 
}28: "@openzeppelin/contracts": "=4.8.3",`
```

[package.json](https://github.com/code-423n4/2023-12-autonolas/blob/main/package.json#L28)

### [L-04] A year is not always equivalent to 365 days

During leap years, the number of days in a year is 366, which means that calculations performed during these years may yield incorrect results if not adjusted for the additional day.

[OLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L22)
```
solidity
uint256 public constant oneYear = 1 days * 365;
```
### [L-05] Some functions do not work correctly with fee-on-transfer tokens

Some tokens take a transfer fee (e.g. `STA`, `PAXG`), some do not currently charge a fee but may do so in the future (e.g. `USDT`, `USDC`). Should a fee-on-transfer token be added as an asset and deposited, it could be abused, as the accounting is wrong. In the current implementation the following function calls do not work well with fee-on-transfer tokens as the amount variable is the pre-fee amount, including the fee, whereas the final balance do not include the fee anymore.

[FxERC20ChildTunnel.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102)
```solidity
bool success = IERC20(childToken).transferFrom(
```

[FxERC20RootTunnel.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98)
```solidity
bool success = IERC20(rootToken).transferFrom(
```

[Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L234)
```solidity
bool success = IToken(token).transferFrom(
```
### [L-06] Unreturned surplus from msg.value submissions

The subsequent code permits the sender to supply Ether, yet neglects to give back the surplus beyond the requisite amount, effectively locking funds within the contract.

The condition ought to be adjusted for exact matching, or the code should be designed to reimburse the additional amount.

[Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L146-L351)
```solidity

}146: if (msg.value < minAcceptedETH) {

}147: revert LowerThan(msg.value, minAcceptedETH);

}151: amount += msg.value;

}314: if (msg.value < minAcceptedETH) {

}315: revert LowerThan(msg.value, minAcceptedETH);

}333: if (msg.value != totalAmount) {

}334: revert WrongAmount(msg.value, totalAmount);

}338: uint256 donationETH = ETHFromServices + msg.value;

}351: msg.value

```

### [L-07] Incompatibility of increaseAllowance/decreaseAllowance with USDT on mainnet

On the mainnet, the remedy to ensure compatibility with increaseAllowance/decreaseAllowance isn't implemented as evidenced [here](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code). This results in a reversal when establishing a non-zero & non-maximum allowance, unless the set allowance is currently zero.

[OLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L128-L148)
```solidity
 function decreaseAllowance(

 function increaseAllowance(
```

### [L-08] Non-compliant `IERC20` tokens may revert with `transfer`

Some `IERC20` tokens (e.g. BNB, OMG, USDT) do not implement the standard properly, but they are still accepted by most code that accepts `ERC20` tokens. For example, USDT `transfer` functions on L1 do not return booleans: when casted to `IERC20`, their function signatures do not match, and therefore the calls made will revert.

[FxERC20ChildTunnel.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79-L102)
```solidity
bool success = IERC20(childToken).transfer(to, amount);

bool success = IERC20(childToken).transferFrom(
```

[FxERC20RootTunnel.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98)
```solidity
bool success = IERC20(rootToken).transferFrom(
```
### [L-10] Risks with direct `supportsInterface()` invocations

When invoking `supportsInterface()` on a contract not compliant with the ERC-165 standard, there's a risk of the call reverting.

Beyond that, even in cases where the contract does recognize the function, a malicious contract might be designed to consume all available gas from the transaction.

To prevent this, use a low-level [staticcall()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f959d7e4e6ee0b022b41e5b644c79369869d8411/contracts/utils/introspection/ERC165Checker.sol#L119) and assign a fixed gas quantity for the operation, examining the return code afterwards.

Alternatively, leverage OpenZeppelin's [ERC165Checker.supportsInterface()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f959d7e4e6ee0b022b41e5b644c79369869d8411/contracts/utils/introspection/ERC165Checker.sol#L36-L39) for this purpose.

  [GovernorOLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L108)
```solidity
return super.supportsInterface(interfaceId);
```

[wveOLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L293)
```solidity
return IVEOLAS(ve).supportsInterface(interfaceId);
```
 