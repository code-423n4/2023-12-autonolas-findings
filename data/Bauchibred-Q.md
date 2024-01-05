# QA Report

## Table Of Contents

| Issue ID                                                                                                                                   | Description                                                                                                                  |
| ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| [QA-01](#qa-01-missing-payable-in-tokenomicsproxy-contracts-fallback-function)                                                             | Missing `payable` in `TokenomicsProxy` Contract's Fallback Function                                                          |
| [QA-02](#qa-02-introduce-a-nonapprovable-event-and-other-subtle-fixes)                                                                     | Introduce a `NonApprovable` event and other subtle fixes                                                                     |
| [QA-03](#qa-03-incorrect-calculation-of-maximum-lock-time-in-veolassol)                                                                    | Incorrect calculation of maximum lock time in `veOLAS.sol`                                                                   |
| [QA-04](#qa-04-incomplete-hexadecimal-representation-in-tokenuri-function)                                                                 | Incomplete hexadecimal representation in `tokenURI` Function                                                                 |
| [QA-05](#qa-05-checkafterexecution-is-not-implemented-which-would-cause-no-guards-to-be-present-for-the-multisig-call-after-any-execution) | `checkAfterExecution` is not implemented which would cause no guards to be present for the multisig call after any execution |
| [QA-06](#qa-06-setters-should-always-have-equality-checkers)                                                                               | Setters should always have equality checkers                                                                                 |
| [QA-07](#qa-07-introduce-a-functionality-to-change-the-chainid-for-a-specific-chain)                                                       | Introduce a functionality to change the `chainId` for a specific chain                                                       |
| [QA-08](#qa-08-incorrect-tick-index-checks-in-liquiditylockboxsol)                                                                         | Incorrect tick index checks in `liquidity_lockbox.sol`                                                                       |
| [QA-09](#qa-09-rename-some-variables)                                                                                                      | Rename some variables                                                                                                        |
| [QA-10](#qa-10-checkmoduletransaction-is-not-present-in-codebase)                                                                          | `checkModuleTransaction()` is not present in codebase                                                                        |
| [QA-11](#qa-11-the-supply-cap-in-tokenomicsconstants-seems-to-wrongly-factor-the-array-indexing-logic)                                     | The Supply Cap in `TokenomicsConstants` seems to wrongly factor the array indexing logic                                     |
| [QA-12](#qa-12-improve-code-documentation)                                                                                                 | Improve code documentation                                                                                                   |

## QA-01 Missing `payable` in `TokenomicsProxy` Contract's Fallback Function

### Proof of Concept

Take a look at`TokenomicsProxy#fallback()`:

```solidity
    fallback() external {
        assembly {
            let tokenomics := sload(PROXY_TOKENOMICS)
            // Otherwise continue with the delegatecall to the tokenomics implementation
            calldatacopy(0, 0, calldatasize())
            let success := delegatecall(gas(), tokenomics, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            if eq(success, 0) {
                revert(0, returndatasize())
            }
            return(0, returndatasize())
        }
    }
```

- The `fallback` function is designed to delegate calls to the tokenomics implementation.
- However, it is not marked as `payable`, meaning it cannot receive native tokens.
- This limitation easily hinders certain functionalities, i.e causing a DOS to any functionality that's going to require sending ETH.

### Impact

The `TokenomicsProxy` contract's fallback function lacks the `payable` modifier. This omission restricts the contract from accepting or processing transactions that send native tokens. As a result, any logic within the tokenomics system that requires receiving native tokens cannot be executed through this proxy, limiting the functionality of the tokenomics implementation.

### Recommended Mitigation Steps

Modify the `fallback` function to include the `payable` modifier:

    ```diff
    -fallback() external {
    +fallback() external payable {
        // Delegatecall logic...
    }
    ```

## QA-02 Introduce a `NonApprovable` event and other subtle fixes

### Impact

Code documentation quality

### Proof of Concept

There are two instances of this, both in `wveOLAS.sol` and `veOLAS.sol`
Take a look at [approve()]()

```solidity
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }
```

This function is suposed to revert the approval of the token which it correctly does, issue is with the event it emits as a new user might not understand reverted message

### Also from `veOLAS.sol`

We can see a check for the lock expiry is implemented in the `depositFor()`:

```solidity
        if (lockedBalance.endTime < (block.timestamp + 1)) {
            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);
        }
```

Issue with this is that it seems unnecessary as protocol could just change the equality check to a `<=` and have it against `block.timestamp instead, i.e

```diff
-        if (lockedBalance.endTime < (block.timestamp + 1)) {
+        if (lockedBalance.endTime <= (block.timestamp )) {
            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);
        }
```

### Also from `Depository.sol`

```solidity
    // Tkenomics contract address
    address public tokenomics;
```

`Tkenomics` should be changed to `Tokenomics`

### Also from `Depository.sol`

Cache the productId array in the `close()` function to ease operations
`            uint256 productId = productIds[i];`

### Recommended Mitigation Steps

Introduce a `NonApprovable` event and emit this instead of `NonTransferable` in instances of approvals being queried.

## QA-03 Incorrect calculation of maximum lock time in `veOLAS.sol`

### Proof of Concept

The `veOLAS` contract, designed for locking tokens, incorrectly calculates the maximum lock time as exactly four years without accounting for leap years.

The issue is found in the definition of `MAXTIME` and `IMAXTIME`:

```solidity
// Maximum lock time (4 years)
uint256 internal constant MAXTIME = 4 * 365 * 86400; // 4 years in seconds
int128 internal constant IMAXTIME = 4 * 365 * 86400; // 4 years in seconds, as int128
```

#### Analysis:

- The contract defines `MAXTIME` as `4 * 365 * 86400`, which represents four years in seconds, assuming each year has 365 days.
- However, within four years, a leap year definitely occurs, adding an extra day to the calendar. This means that over a four-year period, the total count of days should be `365 * 4 + 1`.
- Failing to account for the leap year results in a shortfall of 86,400 seconds (1 day) in the calculation of `MAXTIME`.

### Impact

This discrepancy can lead to a minor but notable difference in the actual locking period, potentially impacting the users who lock their tokens for this maximum duration.

Note that protocol is making calculation all over codes hinting 1340 years, if looking at that long of a timeframe then it would only be fairs to take into accout leap years, otherwise protocol is looking at >300,000 seconds on the later years

### Recommended Mitigation Steps

Update the `MAXTIME` and `IMAXTIME` constants to accurately represent four years, including the leap year. The revised calculation should be:

```solidity
uint256 internal constant MAXTIME = (4 * 365 + 1) * 86400; // 4 years including a leap year
int128 internal constant IMAXTIME = (4 * 365 + 1) * 86400; // Same, as int128
```

## QA-04 Incomplete hexadecimal representation in `tokenURI` Function

### Proof of Concept

The `tokenURI` function in the provided smart contract contains a vulnerability due to an incomplete hexadecimal representation. This issue arises from the `_toHex16` function, which fails to prepend the '0x' prefix to the hexadecimal string, deviating from the copied hexadecimal format, i.e as referenced in the code snippet from the open sourced link: https://stackoverflow.com/questions/67893318/solidity-how-to-represent-bytes32-as-string

Take a look at the `_toHex16()` function:

```solidity
    function _toHex16(bytes16 data) internal pure returns (bytes32 result) {
        result = bytes32 (data) & 0xFFFFFFFFFFFFFFFF000000000000000000000000000000000000000000000000 |
        (bytes32 (data) & 0x0000000000000000FFFFFFFFFFFFFFFF00000000000000000000000000000000) >> 64;
        result = result & 0xFFFFFFFF000000000000000000000000FFFFFFFF000000000000000000000000 |
        (result & 0x00000000FFFFFFFF000000000000000000000000FFFFFFFF0000000000000000) >> 32;
        result = result & 0xFFFF000000000000FFFF000000000000FFFF000000000000FFFF000000000000 |
        (result & 0x0000FFFF000000000000FFFF000000000000FFFF000000000000FFFF00000000) >> 16;
        result = result & 0xFF000000FF000000FF000000FF000000FF000000FF000000FF000000FF000000 |
        (result & 0x00FF000000FF000000FF000000FF000000FF000000FF000000FF000000FF0000) >> 8;
        result = (result & 0xF000F000F000F000F000F000F000F000F000F000F000F000F000F000F000F000) >> 4 |
        (result & 0x0F000F000F000F000F000F000F000F000F000F000F000F000F000F000F000F00) >> 8;
        result = bytes32 (0x3030303030303030303030303030303030303030303030303030303030303030 +
        uint256 (result) +
            (uint256 (result) + 0x0606060606060606060606060606060606060606060606060606060606060606 >> 4 &
            0x0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F) * 39);
    }
```

Unlike the referenced [open source](https://stackoverflow.com/questions/67893318/solidity-how-to-represent-bytes32-as-string),

```solidity
function toHex (bytes32 data) public pure returns (string memory) {
    return string (abi.encodePacked ("0x", toHex16 (bytes16 (data)), toHex16 (bytes16 (data << 128))));
}

```

As seen, in [protcol's implementation](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L109-L124) there is no prepending `0x` here, note in the usage of `_toHex16` within `tokenURI`:

```solidity
function tokenURI(uint256 unitId) public view virtual override returns (string memory) {
    bytes32 unitHash = _getUnitHash(unitId);
    // Concatenate the base URI, a cid prefix, and the hex16 representation
    return string(abi.encodePacked(baseURI, CID_PREFIX, _toHex16(bytes16(unitHash)),
        _toHex16(bytes16(unitHash << 128))));
}
```

### Impact

Medium to low, these discrepancies lead to potential misinterpretation or malfunction in the system in regards to the token URIs, particularly since protocol expects to use a fully if fully formatted hexadecimal string.

### Recommended Mitigation Steps

Modify the `toHex16`'s logic to include the '0x' prefix in its output. This ensures that the hexadecimal representation adheres to the standard format.

## QA-05 `checkAfterExecution` is not implemented which would cause no guards to be present for the multisig call after any execution

### Proof of Concept

Take a look at [checkAfterExecution()]()

```solidity
    function checkAfterExecution(bytes32, bool) external {}

```

We can see that this function is set to guards the multisig call after its execution, but probably due to the developers forgetting this piece, it's not being implemented which would mean that there are no guardrails in place, note that based on the [Changelog](https://github.com/safe-global/safe-contracts/blob/f03dfae65fd1d085224b00a10755c509a4eaacfe/CHANGELOG.md?plain=1#L399) from Safe we can see that an important check that's to be implemented by guards is `checkAfterExecution`. This check is called at the very end of the execution and allows to perform checks on the final state of the Safe. The parameters passed to that check are the `safeTxHash` and a `success` boolean.

### Impact

Currently there are no checks in `checkAfterExecution()` to ensure that the signers cannot perform any illegal actions or for the Safe itself to be able to validate it's final state as required by the [Changelog](https://github.com/safe-global/safe-contracts/blob/f03dfae65fd1d085224b00a10755c509a4eaacfe/CHANGELOG.md?plain=1#L399)

whatever false logic they might want to implement to exert too much control over the safe, like adding more owners to the safe or what have you, implying that actors could coll

### Recommended Mitigation Steps

Fully implement the `checkAfterExecution()` function and have it validate the final state of the Safe after any execution

## QA-06 Setters should always have equality checkers

### Proof of Concept

This is the case for most `change...` prefixed functions
For example, take a look at `GenericRegistry.sol`

```solidity

    function changeOwner(address newOwner) external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    function changeManager(address newManager) external virtual {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newManager == address(0)) {
            revert ZeroAddress();
        }

        manager = newManager;
        emit ManagerUpdated(newManager);
    }

```

As seen, this function checks and changes some aspects of the protocol, issue is that the codes don't check if these values are what's been previously set, i.e if the new value is equal to the old value

### Impact

Bad code quality

### Recommended Mitigation Steps

Introduce an equality check.

## QA-07 Introduce a functionality to change the `chainId` for a specific chain

### Proof of Concept

Currenly, protocol harcodes chainId in multiple logics for different chains, for example which
Take a look at this section from [GuardCM.\_processBridgeData()]()

```solidity
        if (chainId == 137 || chainId == 80001) {
            // Check the L1 initial selector
            bytes4 functionSig = bytes4(data);
            if (functionSig != SEND_MESSAGE_TO_CHILD) {
                revert WrongSelector(functionSig, chainId);
            }
```

Evudently, there is a hradcode of the chain Id and depending on this the specific logic of the polyygon chain is applied, issue is that whereever a hardcode is applied and the underlying chain undergoes a hardfork, Olas can no longer rightly integrate with the protocol, and there is also no current functionality to like introduce a new chainID

### Impact

As previously stated in the case an integrating chain undergoes a hardfork functionalities to this chain would be DOS'd since protocol currently hardcodes the chainId's without taking into account that a fork could happen.

### Recommended Mitigation Steps

Implement a functionality that can help protocol in the case an integrating chain undergoes a hardfork.

## QA-08 Incorrect tick index checks in `liquidity_lockbox.sol`

### Proof of Concept

The issue is present in the `_getPositionData` function:

```solidity
function _getPositionData(AccountInfo position, address positionMint) internal view returns (Position positionData) {
    // ... other checks ...

    // Check tick values
    if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex) {
        revert("Wrong ticks");
    }

    // ... other checks ...
}
```

Evidently:

- The function checks if the `tickLowerIndex` is not equal to `minTickLowerIndex` and if the `tickUpperIndex` is not equal to `maxTickLowerIndex`.
- The intended design of the protocol as documented is that the `tickLowerIndex` should not be less than `minTickLowerIndex` and the `tickUpperIndex` should not be greater than `maxTickLowerIndex`.
- The current implementation however would mistakenly revert transactions even when the tick indices are within the valid range but do not exactly match the min and max tick lower index constants.

### Impact

Submitting this as QA, as there is a potential valid logic for protocol to implement it this way, i.e if they only want to engage with full liquidity positions, otherwise this seems of a higher severity since the `liquidity_lockbox.sol` contains a flawed tick index validation logic, due to its use of strict equality checks (`!=`) instead of range checks (`</>`), which contradicts the protocol's intended design and this design flaw could lead to scenarios where legitimate attempts to `deposit()` liquidity are unjustly reverted, thus impeding the protocol's functionality and user experience.

### Recommended Mitigation Steps

If this is intended functionality then correctly document this, otherwise modify the tick index checks to ensure they validate the indices are within the intended range. The checks should be updated to something like:

```solidity
if (positionData.tickLowerIndex < minTickLowerIndex || positionData.tickUpperIndex > maxTickLowerIndex) {
    revert("Tick indices out of range");
}
```

This way, valid tick ranges wouldn't revert at an attempt to `deposit()` or `_getPositionData()`.

## QA-09 Rename some variables

### Proof of Concept

Take a look at [liquidity_lockbox.sol]()

```solidity
    int32 public constant minTickLowerIndex = -443632;
    int32 public constant maxTickLowerIndex = 443632;
```

These are tick values for the accepted ranges, but unlike the `minTickLowerIndex` that relays to the right value it's supposed to be, i.e the minimum, the latter `maxTickLowerIndex` has a confusing name as it's supposed to be the upper limit value.

### Impact

Confusing codes

### Recommended Mitigation Steps

Rename `maxTickLowerIndex` to `maxTickUpperIndex`

## QA-10 `checkModuleTransaction()` is not present in codebase

### Proof of Concept

Take a look at [GuardCM.sol]()

One can see that the `checkAfterExecution()` function exists, but there is no implementation of `checkModuleTransaction` which somewhat counters the idea of safely integrating with Gnosis.

Note that The `checkModuleTransaction()`

### Impact

Unsafe integration with Gnosis

### Recommended Mitigation Steps

Implement all safe integration checks in regards to Gnosis

## QA-11 The Supply Cap in `TokenomicsConstants` seems to wrongly factor the array indexing logic

### Proof of Concept

The issue is present in the `getSupplyCapForYear()` function, i.e:

```solidity
function getSupplyCapForYear(uint256 numYears) public pure returns (uint256 supplyCap) {
    if (numYears < 10) {
        uint96[10] memory supplyCaps = [
            // ... array elements ...
        ];
        supplyCap = supplyCaps[numYears];
    } else {
        // ... logic for years after 10 ...
    }
}
```

#### Issue Analysis:

- For the first 10 years, the function uses a pre-defined array `supplyCaps` to return the supply cap.
- The function retrieves the supply cap using `supplyCaps[numYears]`. However, since arrays in Solidity are zero-indexed, `supplyCaps[1]` actually refers to the second element in the array, not the first.
- This means that in the first year, the function incorrectly returns the second year's supply cap, and so on for subsequent years.

### Impact

The `getSupplyCapForYear` function in the `TokenomicsConstants` contract incorrectly calculates the supply cap due to a misunderstanding of array indexing. This issue arises because the function assumes that the `numYears` parameter will match the actual number of years that have passed, but it does not account for the zero-based indexing of arrays. As a result, the function returns the wrong supply cap, potentially leading to user confusion and frustration when incorrect data is provided, and also affecting other instances where the supply cap could be used for the protocol

### Recommended Mitigation Steps

Where as one can understand coming from the `Tokenomics.sol` that `numYears==0` should be 1 year past the logic is not documented with the `getSupplyCapForYear()` and this should be modified.

## QA-12 Improve code documentation

### Proof of Concept

Take a look at [UnitRegistry.sol#L26)](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L26)

```solidity
// Type of the unit: component or unit
    UnitType public immutable unitType;
```

`component or unit` is supposed to be `component or AGENT`

### Impact

Confusion in code

### Recommended Mitigation Steps

Improve documentation quality and implement fixes.
