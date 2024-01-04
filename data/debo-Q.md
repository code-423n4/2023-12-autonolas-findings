# Title
[L-01] Potential use of "block.number" as source of randonmness
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