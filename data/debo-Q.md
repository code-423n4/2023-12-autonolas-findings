# Title
# [L-01] Potential use of "block.number" as source of randonmness
# Impact
The environment variable "block.number" looks like it might be used as a source of randomness. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
# Location
```url
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L173-L322
```
# POC
The vulnerable functions of checkpoint are depicted below.
```sol
    function _checkpoint(
        address account,
        LockedBalance memory oldLocked,
        LockedBalance memory newLocked,
        uint128 curSupply
    ) internal {
        PointVoting memory uOld;
        PointVoting memory uNew;
        int128 oldDSlope;
        int128 newDSlope;
        uint256 curNumPoint = totalNumPoints;


        if (account != address(0)) {
            // Calculate slopes and biases
            // Kept at zero when they have to
            if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) {
                uOld.slope = int128(oldLocked.amount) / IMAXTIME;
                uOld.bias = uOld.slope * int128(uint128(oldLocked.endTime - uint64(block.timestamp)));
            }
            if (newLocked.endTime > block.timestamp && newLocked.amount > 0) {
                uNew.slope = int128(newLocked.amount) / IMAXTIME;
                uNew.bias = uNew.slope * int128(uint128(newLocked.endTime - uint64(block.timestamp)));
            }


            // Reads values of scheduled changes in the slope
            // oldLocked.endTime can be in the past and in the future
            // newLocked.endTime can ONLY be in the FUTURE unless everything is expired: then zeros
            oldDSlope = mapSlopeChanges[oldLocked.endTime];
            if (newLocked.endTime > 0) {
                if (newLocked.endTime == oldLocked.endTime) {
                    newDSlope = oldDSlope;
                } else {
                    newDSlope = mapSlopeChanges[newLocked.endTime];
                }
            }
        }


        PointVoting memory lastPoint;
        if (curNumPoint > 0) {
            lastPoint = mapSupplyPoints[curNumPoint];
        } else {
            // If no point is created yet, we take the actual time and block parameters
            lastPoint = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);
        }
        uint64 lastCheckpoint = lastPoint.ts;
        // initialPoint is used for extrapolation to calculate the block number and save them
        // as we cannot figure that out in exact values from inside of the contract
        PointVoting memory initialPoint = lastPoint;
        uint256 block_slope; // dblock/dt
        if (block.timestamp > lastPoint.ts) {
            // This 1e18 multiplier is needed for the numerator to be bigger than the denominator
            // We need to calculate this in > uint64 size (1e18 is > 2^59 multiplied by 2^64).
            block_slope = (1e18 * uint256(block.number - lastPoint.blockNumber)) / uint256(block.timestamp - lastPoint.ts);
        }
        // If last point is already recorded in this block, slope == 0, but we know the block already in this case
        // Go over weeks to fill in the history and (or) calculate what the current point is
        {
            // The timestamp is rounded by a week and < 2^64-1
            uint64 tStep = (lastCheckpoint / WEEK) * WEEK;
            for (uint256 i = 0; i < 255; ++i) {
                // Hopefully it won't happen that this won't get used in 5 years!
                // If it does, users will be able to withdraw but vote weight will be broken
                // This is always practically < 2^64-1
                unchecked {
                    tStep += WEEK;
                }
                int128 dSlope;
                if (tStep > block.timestamp) {
                    tStep = uint64(block.timestamp);
                } else {
                    dSlope = mapSlopeChanges[tStep];
                }
                lastPoint.bias -= lastPoint.slope * int128(int64(tStep - lastCheckpoint));
                lastPoint.slope += dSlope;
                if (lastPoint.bias < 0) {
                    // This could potentially happen, but fuzzer didn't find available "real" combinations
                    lastPoint.bias = 0;
                }
                if (lastPoint.slope < 0) {
                    // This cannot happen - just in case. Again, fuzzer didn't reach this
                    lastPoint.slope = 0;
                }
                lastCheckpoint = tStep;
                lastPoint.ts = tStep;
                // After division by 1e18 the uint64 size can be reclaimed
                lastPoint.blockNumber = initialPoint.blockNumber + uint64((block_slope * uint256(tStep - initialPoint.ts)) / 1e18);
                lastPoint.balance = initialPoint.balance;
                // In order for the overflow of total number of economical checkpoints (starting from zero)
                // The _checkpoint() call must happen n >(2^256 -1)/255 or n > ~1e77/255 > ~1e74 times
                unchecked {
                    curNumPoint += 1;    
                }
                if (tStep == block.timestamp) {
                    lastPoint.blockNumber = uint64(block.number);
                    lastPoint.balance = curSupply;
                    break;
                } else {
                    mapSupplyPoints[curNumPoint] = lastPoint;
                }
            }
        }


        totalNumPoints = curNumPoint;


        // Now mapSupplyPoints is filled until current time
        if (account != address(0)) {
            // If last point was in this block, the slope change has been already applied. In such case we have 0 slope(s)
            lastPoint.slope += (uNew.slope - uOld.slope);
            lastPoint.bias += (uNew.bias - uOld.bias);
            if (lastPoint.slope < 0) {
                lastPoint.slope = 0;
            }

```
# Remediation
Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
# [L-02] Constant/View/Pure functions
# Impact
Functions can be declared view in which case they promise not to modify the state.
If the compiler’s EVM target is Byzantium or newer (default) the opcode STATICCALL is used when view functions are called, which enforces the state to stay unmodified as part of the EVM execution. For library view functions DELEGATECALL is used, because there is no combined DELEGATECALL and STATICCALL. This means library view functions do not have run-time checks that prevent state modifications. This should not impact security negatively because library code is usually known at compile-time and the static checker performs compile-time checks.

The following statements are considered modifying the state:

Writing to state variables.

Emitting events.

Creating other contracts.

Using selfdestruct.

Sending Ether via calls.

Calling any function not marked view or pure.

Using low-level calls.

Using inline assembly that contains certain opcodes.

Pure Functions
Functions can be declared pure in which case they promise not to read from or modify the state. In particular, it should be possible to evaluate a pure function at compile-time given only its inputs and msg.data, but without any knowledge of the current blockchain state. This means that reading from immutable variables can be a non-pure operation.

In addition to the list of state modifying statements explained above, the following are considered reading from the state:

Reading from state variables.

Accessing address(this).balance or <address>.balance.

Accessing any of the members of block, tx, msg (with the exception of msg.sig and msg.data).

Calling any function not marked pure.

Using inline assembly that contains certain opcodes.

Pure functions are able to use the revert() and require() functions to revert potential state changes when an error occurs.

Reverting a state change is not considered a “state modification”, as only changes to the state made previously in code that did not have the view or pure restriction are reverted and that code has the option to catch the revert and not pass it on.

This behavior is also in line with the STATICCALL opcode.

# POC
The locations of the vulnerable functions are as follows.

GenericRegistry.tokenByIndex(uint256) : Is constant but potentially should not be.
Pos: 97:4:

GenericRegistry.tokenURI(uint256) : Is constant but potentially should not be.
Pos: 135:4:

UnitRegistry._checkDependencies(uint32[],uint32) : Potentially should be constant/view/pure but is not.
Pos: 42:4:

UnitRegistry._calculateSubComponents(enum UnitRegistry.UnitType,uint32[]) : Is constant but potentially should not be.
Pos: 200:4

GnosisSafeMultisig._parseData(bytes) : Is constant but potentially should not be.
more
Pos: 45:4

Depository.getProducts(bool) : Is constant but potentially should not be.
more
Pos: 396:4

Depository.getBonds(address,bool) : Is constant but potentially should not be.
more
Pos: 435:4

FxBaseChildTunnel._processMessageFromRoot(uint256,address,bytes) : Potentially should be constant/view/pure but is not. Note: Modifiers are currently not considered by this static analysis.
Pos: 63:4.

# Remediation
It is not possible to prevent functions from reading the state at the level of the EVM, it is only possible to prevent them from writing to the state (i.e. only view can be enforced at the EVM level, pure can not).
