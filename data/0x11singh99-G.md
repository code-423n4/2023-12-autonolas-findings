**Note:**  I've removed all findings instances appearing in the bot and 4naly3er reports and have only kept the ones that they missed.

# Gas-Optimization for Olas Protocol

## Gas Optimizations

| Number                                                                                                      | Issue                                                                                          | Instances |
| ----------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------- | :-------: |
| [[G-01](#g-01-change-the-order-of-declaration-of-state-variables-to-save-storage-slots-by-packing-tightly)] | Change the order of declaration of state variables to save storage slots by packing tightly    |     1     |
| [[G-02](#g-02-state-variables-can-be-packed-into-fewer-storage-slot-by-reducing-their-size)]                | State variables can be packed into fewer storage slot by reducing their size                   |     1     |
| [[G-03](#g-03-state-variables-only-set-in-the-constructor-should-be-declared-immutablemissed-by-bot)]       | `State` variables only set in the `constructor` should be declared `immutable` (Missed by bot) |     1     |
| [[G-04](#g-04-state-variables-can-be-cached-instead-of-re-reading-them-from-storage)]                       | `State` variables can be cached instead of re-reading them from storage                        |     2     |
| [[G-05](#g-05-here-no-need-to-check-for-address0-in-constructor)]                                           | Here no need to check for `address(0)` in `constructor`                                        |     3     |
| [[G-06](#g-06-cache-value-in-a-local-variable-instead-of-re-calculate)]                                     | `Cache` value in a local variable instead of `re-calculate`                                    |     1     |
| [[G-07](#g-07-use-calldata-instead-of-memory)]                                                              | Use `calldata` instead of `memory`                                                             |    15     |
| [[G-08](#g-08-dont-cache-state-variable-if-it-is-used-only-once)]                                           | Don't cache `state variable` if it is used only once                                           |     1     |
| [[G-09](#g-09-cache-array-length-if-it-is-used-many-times)]                                                 | Cache `Array length` if it is used many times                                                  |     2     |
| [[G-10](#g-10-remove-unused-error)]                                                                         | Remove unused error                                                                            |     1     |
| [[G-11](#g-11-leverage-dot-notation-method-for-struct-assignment)]                                          | Leverage dot notation method for struct assignment                                             |     5     |
| [[G-12](#g-12-do-not-initialize-variables-with-their-default-value)]                                        | Do not initialize variables with their `default value`                                         |    11     |
| [[G-13](#g-13-first-check-the-if-statement-that-are-not-reading-the-state-variables)]                       | First check the `if()` statement that are not reading the `state variables`                    |    40     |
| [[G-14](#g-14-if-statement-is-already-check-before)]                                                        | `If()` statement is already check before                                                       |     1     |
| [[G-15](#g-15-use-cached-value)]                                                                            | Use `cached` value                                                                             |     3     |

## [G-01] Change the order of declaration of state variables to save storage slots by packing tightly

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot.

_1 Instance_

### Pack `numPositionAccounts` and `firstAvailablePositionAccountIndex` with `address pdaProgram` and `totalLiquidity` with `address bridgedTokenMint`: `SAVES 2000 GAS, 1 SLOT`

By re ordering the declaration of state variables can save 1 SLOT here. `numPositionAccounts` and `firstAvailablePositionAccountIndex` both are size of `uint32` so both can be packed with `address pdaProgram` in one SLOT. And `totalLiquidity` is of size `uint64` so it can also be packed with `address bridgedTokenMint` in one SLOT. By packing them like this will save **1 Storage Slot** which was occupied by `numPositionAccounts`, `firstAvailablePositionAccountIndex` and `totalLiquidity`.

```solidity
File : lockbox-solana/solidity/liquidity_lockbox.sol

27:    // Current program owned PDA account address
28:    address public pdaProgram;
29:    // Bridged token mint address
30:    address public bridgedTokenMint;
31:    // PDA bridged token account address
32:    address public pdaBridgedTokenAccount;
33:    // PDA header for position account
34:    uint64 public pdaHeader = 0xd0f7407ae48fbcaa;
35:    // Program PDA seed
36:    bytes public constant pdaProgramSeed = "pdaProgram";
37:    // Program PDA bump
38:    bytes1 public pdaBump;
39:    int32 public constant minTickLowerIndex = -443632;
40:    int32 public constant maxTickLowerIndex = 443632;
41:
42:    // Total number of token accounts (even those that hold no positions anymore)
43:    uint32 public numPositionAccounts;//@audit
44:    // First available account index in the set of accounts;
45:    uint32 public firstAvailablePositionAccountIndex;//@audit
46:    // Total liquidity in a lockbox
47:    uint64 public totalLiquidity;//@audit

```

[27-47](https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L27C1-L47C34)

### Recommendation :

New order of declaration

```diff

    // Current program owned PDA account address
    address public pdaProgram;
+    // Total number of token accounts (even those that hold no positions anymore)
+    uint32 public numPositionAccounts;
+    // First available account index in the set of accounts;
+    uint32 public firstAvailablePositionAccountIndex;

    // Bridged token mint address
    address public bridgedTokenMint;
+    // Total liquidity in a lockbox
+    uint64 public totalLiquidity;

    // PDA bridged token account address
    address public pdaBridgedTokenAccount;
    // PDA header for position account
    uint64 public pdaHeader = 0xd0f7407ae48fbcaa;
    // Program PDA seed
    bytes public constant pdaProgramSeed = "pdaProgram";
    // Program PDA bump
    bytes1 public pdaBump;
    int32 public constant minTickLowerIndex = -443632;
    int32 public constant maxTickLowerIndex = 443632;

-    // Total number of token accounts (even those that hold no positions anymore)
-    uint32 public numPositionAccounts;
-    // First available account index in the set of accounts;
-    uint32 public firstAvailablePositionAccountIndex;
-    // Total liquidity in a lockbox
-    uint64 public totalLiquidity;

```

## [G-02] State variables can be packed into fewer storage slot by reducing their size

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this
will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the
variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent
SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset
(20000 gas). Reads of the variables can also be cheaper

_1 Instance_

### After reducing size of `_locked` from `uint256` to `uint8`, it can be packed with `address manager` in single SLOT :`SAVES 2000 GAS`, `1 SLOT`

Reduce uint type of `_locked` from `uint256` to `uint8` and pack with `address manager` because `_locked` can have only two values `1` and `2` assigned in `UnitRegistry::create` function at the [line-56](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L56) and at [line-113](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L113) . Since `GenericRegistry.sol` is inherited by `UnitRegistry.sol`. At the time of declaration also 1 is assigned in `_locked`.

### So `uint8` is more than enough to hold these values

```solidity
File : registries/contracts/GenericRegistry.sol

17:  address public manager;

23:  uint256 internal _locked = 1;
```

[17-23](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L17C1-L23C34)

```diff
File : registries/contracts/GenericRegistry.sol

    address public manager;
+   uint8 internal _locked = 1;
    // Base URI
    string public baseURI;
    // Unit counter
    uint256 public totalSupply;
    // Reentrancy lock
-   uint256 internal _locked = 1;

```

## [G-03] `State` variables only set in the `constructor` should be declared `immutable`(Missed by bot)

Accessing state variables within a function involves an SLOAD operation, but `immutable` variables can be accessed directly without the need of it, thus reducing gas costs. As these state variables are assigned only in the constructor, consider declaring them `immutable`.

**Note: Missed by bot-report**

_1 Instance_

```diff
File : tokenomics/contracts/Treasury.sol

-     address public olas;
+     address public immutable olas;

```

[65](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L65C5-L65C25)

## [G-04] `State` variables can be cached instead of re-reading them from storage

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

**Note: Missed by bot-report**

_2 Instances_

<details><summary>see instance</summary>

```solidity
File : tokenomics/contracts/Tokenomics.sol

1010:        numYears = (block.timestamp + curEpochLen - timeLaunch) / ONE_YEAR;
1011:        // Account for the year change to adjust the max bond
1012:        if (numYears > currentYear) {
1013:            // Calculate the inflation remainder for the passing year
1014:            // End of the year timestamp
1015:            uint256 yearEndTime = timeLaunch + numYears * ONE_YEAR;

```

[1010-1015](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1010C9-L1015C68)

```diff
File : tokenomics/contracts/Tokenomics.sol

+        uint32 _timeLaunch = timeLaunch;
-        numYears = (block.timestamp + curEpochLen - timeLaunch) / ONE_YEAR;
+        numYears = (block.timestamp + curEpochLen - _timeLaunch) / ONE_YEAR;
        // Account for the year change to adjust the max bond
        if (numYears > currentYear) {
            // Calculate the inflation remainder for the passing year
            // End of the year timestamp
-           uint256 yearEndTime = timeLaunch + numYears * ONE_YEAR;
+           uint256 yearEndTime = _timeLaunch + numYears * ONE_YEAR;

```

```solidity
File : tokenomics/contracts/Tokenomics.sol

973:    if (tokenomicsParametersUpdated & 0x02 == 0x02) {
...
1034:            tokenomicsParametersUpdated = 0;
1035:        }

```

[973-1035](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L973C9-L1035C10)

```diff
File : tokenomics/contracts/Tokenomics.sol

+       bytes1 _tokenomicsParametersUpdated = tokenomicsParametersUpdated;
-        if (tokenomicsParametersUpdated & 0x02 == 0x02) {
+        if (_tokenomicsParametersUpdated & 0x02 == 0x02) {
            // Confirm the change of incentive fractions
            emit IncentiveFractionsUpdated(eCounter + 1);
        } else {
            // Copy current tokenomics point into the next one such that it has necessary tokenomics parameters
            for (uint256 i = 0; i < 2; ++i) {
                nextEpochPoint.unitPoints[i].topUpUnitFraction = tp.unitPoints[i].topUpUnitFraction;
                nextEpochPoint.unitPoints[i].rewardUnitFraction = tp.unitPoints[i].rewardUnitFraction;
            }
            nextEpochPoint.epochPoint.rewardTreasuryFraction = tp.epochPoint.rewardTreasuryFraction;
            nextEpochPoint.epochPoint.maxBondFraction = tp.epochPoint.maxBondFraction;
        }
        // Update parameters for the next epoch, if changes were requested by the changeTokenomicsParameters() function
        // Check if the second bit is set to one
-        if (tokenomicsParametersUpdated & 0x01 == 0x01) {
+        if (_tokenomicsParametersUpdated & 0x01 == 0x01) {
            // Update epoch length and set the next value back to zero
            if (nextEpochLen > 0) {
                curEpochLen = nextEpochLen;
                epochLen = uint32(curEpochLen);
                nextEpochLen = 0;
            }

            // Update veOLAS threshold and set the next value back to zero
            if (nextVeOLASThreshold > 0) {
                veOLASThreshold = nextVeOLASThreshold;
                nextVeOLASThreshold = 0;
            }

            // Confirm the change of tokenomics parameters
            emit TokenomicsParametersUpdated(eCounter + 1);
        }
        // Record settled epoch timestamp
        tp.epochPoint.endTime = uint32(block.timestamp);

        // Adjust max bond value if the next epoch is going to be the year change epoch
        // Note that this computation happens before the epoch that is triggered in the next epoch (the code above) when
        // the actual year changes
        numYears = (block.timestamp + curEpochLen - timeLaunch) / ONE_YEAR;
        // Account for the year change to adjust the max bond
        if (numYears > currentYear) {
            // Calculate the inflation remainder for the passing year
            // End of the year timestamp
            uint256 yearEndTime = timeLaunch + numYears * ONE_YEAR;
            // Calculate the inflation per epoch value until the end of the year
            inflationPerEpoch = (yearEndTime - block.timestamp) * curInflationPerSecond;
            // Recalculate the inflation per second based on the new inflation for the current year
            curInflationPerSecond = getInflationForYear(numYears) / ONE_YEAR;
            // Add the remainder of the inflation for the next epoch based on a new inflation per second ratio
            inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;
            // Calculate the max bond value
            curMaxBond = (inflationPerEpoch * nextEpochPoint.epochPoint.maxBondFraction) / 100;
            // Update state maxBond value
            maxBond = uint96(curMaxBond);
            // Reset the tokenomics parameters update flag
            tokenomicsParametersUpdated = 0;
-        } else if (tokenomicsParametersUpdated > 0) {
+        } else if (_tokenomicsParametersUpdated > 0) {
            // Since tokenomics parameters have been updated, maxBond has to be recalculated
            curMaxBond = (curEpochLen * curInflationPerSecond * nextEpochPoint.epochPoint.maxBondFraction) / 100;
            // Update state maxBond value
            maxBond = uint96(curMaxBond);
            // Reset the tokenomics parameters update flag
            tokenomicsParametersUpdated = 0;
        }

```

</details>

## [G-05] Here no need to check for `address(0)` in `constructor`

_3 Instances_

Here no need to check `_fxChild` for address(0) because `_fxChild` is not assigning to any `state`/`immutable` variable.

```solidity
File : governance/contracts/bridges/FxERC20ChildTunnel.sol

37:   constructor(address _fxChild, address _childToken, address _rootToken) FxBaseChildTunnel(_fxChild) {
38:        // Check for zero addresses
           //@audit _fxChild
39:        if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0)) {
40:            revert ZeroAddress();
41:        }
42:
43:        childToken = _childToken;
44:        rootToken = _rootToken;
45:     }

```

[37-45](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L37C3-L45C6)

Here no need to check `_checkpointManager` and `_fxRoot` for address(0) because `_checkpointManager` and `_fxRoot` are not assigning to any `state`/`immutable` variable.

```solidity
File : governance/contracts/bridges/FxERC20RootTunnel.sol

38:  constructor(address _checkpointManager, address _fxRoot, address _childToken, address _rootToken)
39:        FxBaseRootTunnel(_checkpointManager, _fxRoot)
40:    {
41:        // Check for zero addresses
           //@audit _checkpointManager and _fxRoot
42:        if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
43:            _rootToken == address(0)) {
44:            revert ZeroAddress();
45:        }
46:
47:        childToken = _childToken;
48:        rootToken = _rootToken;
49:    }

```

[38-49](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L38C3-L49C6)

## [G-06] `Cache` value in a local variable instead of `re-calculate`

_1 Instance_

```solidity
File : tokenomics/contracts/Tokenomics.sol

319:        // Check that the tokenomics contract is initialized no later than one year after the OLAS token is deployed
320:        if (block.timestamp >= (_timeLaunch + ONE_YEAR)) {
321:            revert Overflow(_timeLaunch + ONE_YEAR, block.timestamp);
322:        }
323:        // Seconds left in the deployment year for the zero year inflation schedule
324:        // This value is necessary since it is different from a precise one year time, as the OLAS contract started earlier
325:        uint256 zeroYearSecondsLeft = uint32(_timeLaunch + ONE_YEAR - block.timestamp);

```

[319-325](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L319C1-L325C88)

```diff
File : tokenomics/contracts/Tokenomics.sol

+       uint256 _timeLaunchPlusOneYear = _timeLaunch + ONE_YEAR;
        // Check that the tokenomics contract is initialized no later than one year after the OLAS token is deployed
-       if (block.timestamp >= (_timeLaunch + ONE_YEAR)) {
+       if (block.timestamp >= _timeLaunchPlusOneYear) {
-            revert Overflow(_timeLaunch + ONE_YEAR, block.timestamp);
+            revert Overflow(_timeLaunchPlusOneYear, block.timestamp);
        }
        // Seconds left in the deployment year for the zero year inflation schedule
        // This value is necessary since it is different from a precise one year time, as the OLAS contract started earlier
-       uint256 zeroYearSecondsLeft = uint32(_timeLaunch + ONE_YEAR - block.timestamp);
+       uint256 zeroYearSecondsLeft = uint32(_timeLaunchPlusOneYear - block.timestamp);

```

## [G-07] Use `calldata` instead of `memory`

**Note: Missed by bot-report**

_15 Instances_

<details><summary>see instance</summary>

```solidity
File : governance/contracts/multisigs/GuardCM.sol

387:    function checkTransaction(
388:        address to,
389:        uint256,
390:        bytes memory data, //@audit use calldata
391:        Enum.Operation operation,
392:        uint256,
393:        uint256,
394:        uint256,
395:        address,
396:        address payable,
397:        bytes memory,
398:        address
399:    ) external {

441:    function setTargetSelectorChainIds(
442:        address[] memory targets, //@audit use calldata
443:        bytes4[] memory selectors, //@audit use calldata
444:        uint256[] memory chainIds, //@audit use calldata
445:        bool[] memory statuses //@audit use calldata
446:     ) external {

495:    function setBridgeMediatorChainIds(
496:        address[] memory bridgeMediatorL1s,
497:        address[] memory bridgeMediatorL2s, //@audit use calldata
498:        uint256[] memory chainIds //@audit use calldata
499:     ) external {

```

[387-399](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L387C1-L399C17), [441-446](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L441C1-L446C17), [495-499](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L495C1-L499C17)

```solidity
File : governance/contracts/GovernorOLAS.so

45:     function propose(
46:        address[] memory targets, //@audit use calldata
47:        uint256[] memory values, //@audit use calldata
48:        bytes[] memory calldatas, //@audit use calldata
49:        string memory description //@audit use calldata
50:    ) public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)
51:    {

```

[45-51](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L45C1-L51C6)

```solidity
File : tokenomics/contracts/Dispenser.sol

        //@audit use calldata for `unitTypes` and `unitIds`
89:     function claimOwnerIncentives(uint256[] memory unitTypes, uint256[] memory unitIds) external

```

[89](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L89C5-L89C97)

```solidity
File : tokenomics/contracts/Tokenomics.sol

788:    function trackServiceDonations(
789:        address donator,
790:        uint256[] memory serviceIds, //@audit use calldata
791:        uint256[] memory amounts, //@audit use calldata
792:        uint256 donationETH
793:    ) external {

```

[788-793](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L788C1-L793C17)

</details>

## [G-08] Don't cache `state variable` if it is used only once

**Not: Missed by bot-report**

_1 Instance_

```solidity
File : governance/contracts/OLAS.sol

99:         uint256 _totalSupply = totalSupply;
100:        // Current year
101:        uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
102:        // Calculate maximum mint amount to date
103:        uint256 supplyCap = tenYearSupplyCap;
104:        // After 10 years, adjust supplyCap according to the yearly inflation % set in maxMintCapFraction
105:        if (numYears > 9) {
106:            // Number of years after ten years have passed (including ongoing ones)
107:            numYears -= 9;
108:            for (uint256 i = 0; i < numYears; ++i) {
109:                supplyCap += (supplyCap * maxMintCapFraction) / 100;
110:            }
111:        }
112:        // Check for the requested mint overflow
113:        remainder = supplyCap - _totalSupply;
114:    }

```

[99-114](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L99C9-L114C6)

```diff
File : governance/contracts/OLAS.sol

-       uint256 _totalSupply = totalSupply;
        // Current year
        uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
        // Calculate maximum mint amount to date
        uint256 supplyCap = tenYearSupplyCap;
        // After 10 years, adjust supplyCap according to the yearly inflation % set in maxMintCapFraction
        if (numYears > 9) {
            // Number of years after ten years have passed (including ongoing ones)
            numYears -= 9;
            for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
        }
        // Check for the requested mint overflow
-       remainder = supplyCap - _totalSupply;
+       remainder = supplyCap - totalSupply;
    }

```

## [G-09] Cache `Array length` if it is used many times

**Note: Missed by bot-report**

_2 Instances_

```solidity
File : governance/contracts/multisigs/GuardCM.sol

410:                if (data.length < SELECTOR_DATA_LENGTH) {
411:                    revert IncorrectDataLength(data.length, SELECTOR_DATA_LENGTH);
412:                }
413:
414:                // Get the function signature
415:                bytes4 functionSig = bytes4(data);
416:                // Check the schedule or scheduleBatch function authorized parameters
417:                // All other functions are not checked for
418:                if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) {
419:                    // Data length is too short: need to have enough bytes for the schedule() function
420:                    // with one selector extracted from the payload
421:                    if (data.length < MIN_SCHEDULE_DATA_LENGTH) {
422:                        revert IncorrectDataLength(data.length, MIN_SCHEDULE_DATA_LENGTH);
423:                    }

```

[410-423](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L410C1-L423C22)

```diff
File : governance/contracts/multisigs/GuardCM.sol

+                uint256 _dataLength = data.length;
-                if (data.length < SELECTOR_DATA_LENGTH) {
+                if (_dataLength < SELECTOR_DATA_LENGTH) {
-                    revert IncorrectDataLength(data.length, SELECTOR_DATA_LENGTH);
+                    revert IncorrectDataLength(_dataLength, SELECTOR_DATA_LENGTH);
                }

                // Get the function signature
                bytes4 functionSig = bytes4(data);
                // Check the schedule or scheduleBatch function authorized parameters
                // All other functions are not checked for
                if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) {
                    // Data length is too short: need to have enough bytes for the schedule() function
                    // with one selector extracted from the payload
-                   if (data.length < MIN_SCHEDULE_DATA_LENGTH) {
+                   if (_dataLength < MIN_SCHEDULE_DATA_LENGTH) {
-                        revert IncorrectDataLength(data.length, MIN_SCHEDULE_DATA_LENGTH);
+                        revert IncorrectDataLength(_dataLength, MIN_SCHEDULE_DATA_LENGTH);
                    }

```

```solidity
File : governance/contracts/multisigs/GuardCM.sol

506:       if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) {
507:            revert WrongArrayLength(bridgeMediatorL1s.length, bridgeMediatorL2s.length, chainIds.length, chainIds.length);
508:       }

```

[506-508](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506C8-L508C10)

```diff
File : governance/contracts/multisigs/GuardCM.sol

+       uint256 _bridgeMediatorL1sLength = bridgeMediatorL1s.length;
-        if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) {
-             revert WrongArrayLength(bridgeMediatorL1s.length, bridgeMediatorL2s.length, chainIds.length, chainIds.length);
+        if (_bridgeMediatorL1sLength != bridgeMediatorL2s.length || _bridgeMediatorL1sLength != chainIds.length) {
+             revert WrongArrayLength(_bridgeMediatorL1sLength, bridgeMediatorL2s.length, chainIds.length, chainIds.length);
        }

```

## [G-10] Remove unused error

**Note: Missed by bot-report**

_1 Instance_

```solidity
File : registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

19:    error ZeroAddress();

```

[19](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L19C1-L19C21)

## [G-11] Leverage dot notation method for struct assignment

_5 Instances_

<details><summary>see instance</summary>

We have a few methods we can use when assigning values to struct

- Type({field1: value1, field2: value2});
- Type(value1,value2);
- dot notation(structObject.field = value1);

```solidity
struct Example{
    uint a;
    uint b;
  }
  Example public example;

  function sample1(uint value1, uint value2) external {
    example.a = value1;
    example.b = value2;
  }
  function sample2(uint value1,uint value2) external{
      example = Example(value1, value2);
    }

  function sample3(uint value1, uint value2) external{
    example = Example({
      a : value1,
      b : value2
    });
  }

```

When we use the first two examples(sample1,sample2), the values are directly written to storage variable(example), however using the last method(sample3)
the compiler will allocate some memory to store the struct instance first before writing it to storage
Example({a:value1,b:value2}) will create new Example in memory to store the values a and b. Then once we have this in memory we need to write it to the storage var example.
That said, we can see where the gas difference would come from, the first two writes the values directly to storage while the last one needs to copy to memory first.

```solidity
File : lockbox-solana/solidity/liquidity_lockbox.sol

        positionData = Position({
            whirlpool: position.data.readAddress(8),
            positionMint: position.data.readAddress(40),
            liquidity: position.data.readUint128LE(72),
            tickLowerIndex: position.data.readInt32LE(88),
            tickUpperIndex: position.data.readInt32LE(92)
        });

```

[86-92](https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L86C1-L92C12)

```diff
File : lockbox-solana/solidity/liquidity_lockbox.sol

-        positionData = Position({
-            whirlpool: position.data.readAddress(8),
-            positionMint: position.data.readAddress(40),
-            liquidity: position.data.readUint128LE(72),
-            tickLowerIndex: position.data.readInt32LE(88),
-            tickUpperIndex: position.data.readInt32LE(92)
-        });
+          positionData.whirlpool = position.data.readAddress(8),
+          positionData.positionMint = position.data.readAddress(40),
+          positionData.liquidity = position.data.readUint128LE(72),
+          positionData.tickLowerIndex = position.data.readInt32LE(88),
+          positionData.tickUpperIndex = position.data.readInt32LE(92)
+        });

```

</details>

## [G-12] Do not initialize variables with their `default value`

Every variable assignment in Solidity costs gas. When initializing variables, we often waste gas by assigning default values that will never be used.

**Note: Missed by bot-report**

_11 Instances_

<details><summary>see instance</summary>

```solidity
File : governance/contracts/bridges/FxGovernorTunnel.sol

125:   for (uint256 i = 0; i < dataLength;) {

153:   for (uint256 j = 0; j < payloadLength; ++j) {

```

[125](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L125C9-L125C47), [153](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L153C13-L153C58)

```solidity
File : governance/contracts/multisigs/GuardCM.sol

212:    for (uint256 i = 0; i < data.length;) {

237:    for (uint256 j = 0; j < payloadLength; ++j) {

273:    for (uint256 i = 0; i < payload.length; ++i) {

292:    for (uint256 i = 0; i < bridgePayload.length; ++i) {

318:    for (uint256 i = 0; i < payload.length; ++i) {

340:    for (uint256 i = 0; i < payload.length; ++i) {

360:    for (uint i = 0; i < targets.length; ++i) {

458:    for (uint256 i = 0; i < targets.length; ++i) {

511:    for (uint256 i = 0; i < chainIds.length; ++i) {

```

[212](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L212C9-L212C48), [237](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L237C13-L237C58), [273](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L273C12-L273C59), [292](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L292C12-L292C65), [318](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L318C13-L318C59), [340](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L340C9-L340C55), [360](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L360C8-L360C52), [458](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L458C9-L458C55), [511](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L511C9-L511C56)

</details>

## [G-13] First check the `if()` statement that are not reading the `state variables`

**Change Order of the `if()` statements**

_40 Instances_

<details><summary>see instance</summary>

```solidity
File : governance/contracts/bridges/BridgedERC20.sol

           //@audit change order - 2nd
32:        if (msg.sender != owner) {
33:            revert OwnerOnly(msg.sender, owner);
34:        }
35:
36:        // Zero address check
           //@audit change order - 1st
37:        if (newOwner == address(0)) {
38:            revert ZeroAddress();
39:        }

```

[32-39](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32C1-L39C10)

```solidity
File : governance/contracts/OLAS.sol

           //@audit change order - 2nd
44:        if (msg.sender != owner) {
45:            revert ManagerOnly(msg.sender, owner);
46:        }
47:
           //@audit change order - 1st
48:        if (newOwner == address(0)) {
49:            revert ZeroAddress();
50:        }



           //@audit change order - 2nd
59:        if (msg.sender != owner) {
60:            revert ManagerOnly(msg.sender, owner);
61:        }
62:
           //@audit change order - 1st
63:        if (newMinter == address(0)) {
64:            revert ZeroAddress();
65:        }

```

[44-50](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44C1-L50C10), [59-65](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59C1-L65C10)

```solidity
File : governance/contracts/bridges/FxGovernorTunnel.sol

           //@audit change order - 2nd
84:        if (msg.sender != address(this)) {
85:            revert SelfCallOnly(msg.sender, address(this));
86:        }
87:
88:        // Check for the zero address
           //@audit change order - 1st
89:        if (newRootGovernor == address(0)) {
90:            revert ZeroAddress();
91:        }

```

[84-91](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84C1-L91C10)

```solidity
File : governance/contracts/bridges/HomeMediator.sol

           //@audit change order - 2nd
84:        if (msg.sender != address(this)) {
85:            revert SelfCallOnly(msg.sender, address(this));
86:        }
87:
88:        // Check for the zero address
           //@audit change order - 1st
89:        if (newForeignGovernor == address(0)) {
90:            revert ZeroAddress();
91:        }

```

[84-91](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84C1-L91C10)

```solidity
File : governance/contracts/multisigs/GuardCM.sol

            //@audit change order - 2nd
155:        if (msg.sender != owner) {
156:            revert OwnerOnly(msg.sender, owner);
157:        }
158:
159:        // Check for the zero address
            //@audit change order - 1st
160:        if (newGovernor == address(0)) {
161:            revert ZeroAddress();
162:        }



            //@audit change order - 2nd
171:        if (msg.sender != owner) {
172:            revert OwnerOnly(msg.sender, owner);
173:        }
174:
175:        // Check for the zero value
            //@audit change order - 1st
176:        if (proposalId == 0) {
177:            revert ZeroValue();
178:        }

```

[155-162](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155C1-L162C10), [171-178](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171C8-L178C10)

```solidity
File : registries/contracts/GenericManager.sol

           //@audit change order - 2nd
22:        if (msg.sender != owner) {
23:            revert OwnerOnly(msg.sender, owner);
24:        }
25:
26:        // Check for the zero address
           //@audit change order - 1st
27:        if (newOwner == address(0)) {
28:            revert ZeroAddress();
29:        }

```

[22-29](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L22C1-L29C10)

```solidity
File : registries/contracts/GenericRegistry.sol

        //@audit change order - 2nd
39:     if (msg.sender != owner) {
40:            revert OwnerOnly(msg.sender, owner);
41:     }
42:
43:     // Check for the zero address
        //@audit change order - 1st
44:     if (newOwner == address(0)) {
45:            revert ZeroAddress();
46:     }



        //@audit change order - 2nd
55:     if (msg.sender != owner) {
56:            revert OwnerOnly(msg.sender, owner);
57:     }
58:
59:     // Check for the zero address
        //@audit change order - 1st
60:     if (newManager == address(0)) {
61:            revert ZeroAddress();
62:     }
```

[39-46](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L39C8-L46C10), [55-62](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L55C8-L62C10)

```solidity
File: tokenomics/contracts/Depository.sol

            //@audit change order - 2nd
125:        if (msg.sender != owner) {
126:            revert OwnerOnly(msg.sender, owner);
127:        }
128:
129:        // Check for the zero address
            //@audit change order - 1st
130:        if (newOwner == address(0)) {
131:            revert ZeroAddress();
132:        }

```

[125-132](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L125C1-L132C10)

```solidity
File : tokenomics/contracts/Dispenser.sol

           //@audit change order - 2nd
48:        if (msg.sender != owner) {
49:            revert OwnerOnly(msg.sender, owner);
50:        }
51:
52:        // Check for the zero address
           //@audit change order - 1st
53:        if (newOwner == address(0)) {
54:            revert ZeroAddress();
55:        }

```

[48-55](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L48C1-L55C10)

```solidity
File : tokenomics/contracts/DonatorBlacklist.sol

           //@audit change order - 2nd
38:        if (msg.sender != owner) {
39:            revert OwnerOnly(msg.sender, owner);
40:        }
41:
42:        // Check for the zero address
           //@audit change order - 1st
43:        if (newOwner == address(0)) {
44:            revert ZeroAddress();
45:        }

```

[38-45](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L38C8-L45C10)

```solidity
File : tokenomics/contracts/Tokenomics.sol

            //@audit change order - 2nd
406:        if (msg.sender != owner) {
407:            revert OwnerOnly(msg.sender, owner);
408:        }
409:
410:        // Check for the zero address
            //@audit change order - 1st
411:        if (newOwner == address(0)) {
412:            revert ZeroAddress();
413:        }



            //@audit change order - 2nd
386:        if (msg.sender != owner) {
387:            revert OwnerOnly(msg.sender, owner);
388:        }
389:
390:        // Check for the zero address
            //@audit change order - 1st
391:        if (implementation == address(0)) {
392:            revert ZeroAddress();
393:        }

```

[406-413](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L406C1-L413C10), [386-393](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L386C1-L393C10)

```solidity
File : tokenomics/contracts/Treasury.sol

            //@audit change order - 2nd
139:        if (msg.sender != owner) {
140:            revert OwnerOnly(msg.sender, owner);
141:        }
142:
143:        // Check for the zero address
            //@audit change order - 1st
144:        if (newOwner == address(0)) {
145:            revert ZeroAddress();
146:        }



            //@audit change order - 3rd
184:        if (msg.sender != owner) {
185:            revert OwnerOnly(msg.sender, owner);
186:        }
187:
188:        // Check for the zero value
            //@audit change order - 1st
189:        if (_minAcceptedETH == 0) {
190:            revert ZeroValue();
191:        }
192:
193:        // Check for the overflow value
            //@audit change order - 2nd
194:        if (_minAcceptedETH > type(uint96).max) {
195:            revert Overflow(_minAcceptedETH, type(uint96).max);
196:        }



            //@audit change order - 3rd
315:        if (msg.sender != owner) {
316:            revert OwnerOnly(msg.sender, owner);
317:        }
318:
319:        // Check that the withdraw address is not treasury itself
            //@audit change order - 2nd
320:        if (to == address(this)) {
321:            revert TransferFailed(token, address(this), to, tokenAmount);
322:        }
323:
324:        // Check for the zero withdraw amount
            //@audit change order - 1st
325:        if (tokenAmount == 0) {
326:            revert ZeroValue();
327:        }



            //@audit change order - 2nd
489:        if (msg.sender != owner) {
490:            revert OwnerOnly(msg.sender, owner);
491:        }
492:
493:        // Check for the zero address token
            //@audit change order - 1st
494:        if (token == address(0)) {
495:            revert ZeroAddress();
496:        }

```

[139-146](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L139C1-L146C10), [184-196](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L184C1-L196C10), [315-327](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L315C9-L327C10), [489-496](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L489C1-L496C10)

</details>

## [G-14] `If()` statement is already check before

_1 Instance_

In this function this `if (account != address(0))` statement is being checked twice while the amount is not updated so do not check it twice, the work will be done with just one check.

```solidity
File : governance/contracts/veOLAS.sol

278:        if (account != address(0)) {
279:            // If last point was in this block, the slope change has been already applied. In such case we have 0 slope(s)
280:            lastPoint.slope += (uNew.slope - uOld.slope);
281:            lastPoint.bias += (uNew.bias - uOld.bias);
282:            if (lastPoint.slope < 0) {
283:                lastPoint.slope = 0;
284:            }
285:            if (lastPoint.bias < 0) {
286:                lastPoint.bias = 0;
287:            }
288:        }
289:
290:        // Record the last updated point
291:        mapSupplyPoints[curNumPoint] = lastPoint;
292:
293:        if (account != address(0)) { //@audit This statement is already checked before
294:            // Schedule the slope changes (slope is going down)
295:            // We subtract new_user_slope from [newLocked.endTime]
296:            // and add old_user_slope to [oldLocked.endTime]
297:            if (oldLocked.endTime > block.timestamp) {
298:                // oldDSlope was <something> - uOld.slope, so we cancel that
299:                oldDSlope += uOld.slope;
300:                if (newLocked.endTime == oldLocked.endTime) {
301:                    oldDSlope -= uNew.slope; // It was a new deposit, not extension
302:                }
303:                mapSlopeChanges[oldLocked.endTime] = oldDSlope;
304:            }
305:
306:            if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) {
307:                newDSlope -= uNew.slope; // old slope disappeared at this point
308:                mapSlopeChanges[newLocked.endTime] = newDSlope;
309:                // else: we recorded it already in oldDSlope
310:            }
311:            // Now handle user history
312:            uNew.ts = uint64(block.timestamp);
313:            uNew.blockNumber = uint64(block.number);
314:            uNew.balance = newLocked.amount;
315:            mapUserPoints[account].push(uNew);
316:        }

```

[278-316](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278C9-L316C10)

## [G-15] Use `cached` value

_3 Instances_

<details><summary>see instance</summary>

Instead of calculating the `data.length` again and again, use the `dataLength` that is already cached.

```diff
File : governance/contracts/bridges/FxGovernorTunnel.sol

        uint256 dataLength = data.length;
        if (dataLength < DEFAULT_DATA_LENGTH) {
-            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);
+            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, dataLength);
        }

```

[119-122](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L119C8-L122C10)

```diff
File : governance/contracts/bridges/HomeMediator.sol

       uint256 dataLength = data.length;
       if (dataLength < DEFAULT_DATA_LENGTH) {
-            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);
+            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, dataLength);
        }

```

[119-122](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L119C2-L122C10)

Here `lastPoint.ts` is cached in `lastCheckpoint` so use `lastCheckpoint` instead of `lastPoint.ts`

```solidity
File : governance/contracts/veOLAS.sol

217:        uint64 lastCheckpoint = lastPoint.ts;
218:        // initialPoint is used for extrapolation to calculate the block number and save them
219:        // as we cannot figure that out in exact values from inside of the contract
220:        PointVoting memory initialPoint = lastPoint;
221:        uint256 block_slope; // dblock/dt
222:        if (block.timestamp > lastPoint.ts) {
223:            // This 1e18 multiplier is needed for the numerator to be bigger than the denominator
224:            // We need to calculate this in > uint64 size (1e18 is > 2^59 multiplied by 2^64).
225:            block_slope = (1e18 * uint256(block.number - lastPoint.blockNumber)) / uint256(block.timestamp - lastPoint.ts);

```

[217-225](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L217C1-L225C124)

```diff
File : governance/contracts/veOLAS.sol

        uint64 lastCheckpoint = lastPoint.ts;
        // initialPoint is used for extrapolation to calculate the block number and save them
        // as we cannot figure that out in exact values from inside of the contract
        PointVoting memory initialPoint = lastPoint;
        uint256 block_slope; // dblock/dt
-        if (block.timestamp > lastPoint.ts) {
+        if (block.timestamp > lastCheckpoint) {
            // This 1e18 multiplier is needed for the numerator to be bigger than the denominator
            // We need to calculate this in > uint64 size (1e18 is > 2^59 multiplied by 2^64).
-            block_slope = (1e18 * uint256(block.number - lastPoint.blockNumber)) / uint256(block.timestamp - lastPoint.ts);
+            block_slope = (1e18 * uint256(block.number - lastPoint.blockNumber)) / uint256(block.timestamp - lastCheckpoint);

```

</details>
