### Missing zero address check
###### Github lines:
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L579 for target
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L48 for 

---

### State variable that could be defined immutable

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L123C2-L126C26

```solidity
  string public name;
  string public symbol;
```

---


### Invalid type casting

uPoint.ts should be convert in `int128` 

```solidity
int128(int64(ts) - int64(uPoint.ts))

where
struct PointVoting {
    int128 bias;
    int128 slope;
    uint64 ts;
    uint64 blockNumber;
    uint128 balance;
}
```
###### Github lines:
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L245
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L597
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L680
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L704

---

### WEEK / WEEK always give 1 as a value
that’s, so it’s useless to divide then multiple by `WEEK` 
```solidity
uint64 tStep = (lastPoint.ts / WEEK) * WEEK;

unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;

```

###### Github lines:
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L231
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L433
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L487
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L692

---

### Missing duplicate array check
###### Github lines:
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol#L92
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L86