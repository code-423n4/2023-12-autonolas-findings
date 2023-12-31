# veOLAS
1) ## Missing ``assert_not_contract`` Check in ``veOLAS`` contract
The ``veOLAS`` contract contains a low-level vulnerability due to the omission of the ``assert_not_contract`` check in three critical ``functions—increaseAmount``, ``createLockFor``, and ``increaseUnlockTime``.
These functions, evidently influenced by their counterparts in the [veCRV](https://etherscan.io/token/0x5f3b5DfEb7B28CDbD7FAba78963EE202a494e2A2#code) contract, lack the essential verification step to ensure that the caller is not a contract.
In contrast, the [veCRV](https://etherscan.io/token/0x5f3b5DfEb7B28CDbD7FAba78963EE202a494e2A2#code) contract includes the ``assert_not_contract`` check in similar functions as a security measure against potential exploits by malicious contracts.The absence of this check in ``veOLAS`` opens up the possibility of improper exploitation, posing a risk to the contract's security and integrity.
A recommended solution involves promptly integrating the ``assert_not_contract`` check into the specified functions in the ``veOLAS`` contract to mitigate this vulnerability.
example of this problem:
### veOLAS.sol
```
    function increaseAmount(uint256 amount) external {
        LockedBalance memory lockedBalance = mapLockedBalances[msg.sender];
        // Check if the amount is zero
        if (amount == 0) {
            revert ZeroValue();
        }
        // The locked balance must already exist
        if (lockedBalance.amount == 0) {
            revert NoValueLocked(msg.sender);
        }
        // Check the lock expiry
        if (lockedBalance.endTime < (block.timestamp + 1)) {
            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);
        }
        // Check the max possible amount to add, that must be less than the total supply
        // After 10 years, the inflation rate is 2% per year. It would take 220+ years to reach 2^96 - 1 total supply
        if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);
        }

        _depositFor(msg.sender, amount, 0, lockedBalance, DepositType.INCREASE_LOCK_AMOUNT);
    }
```
### veCRV.vy
```
def increase_amount(_value: uint256):
    """
    @notice Deposit `_value` additional tokens for `msg.sender`
            without modifying the unlock time
    @param _value Amount of tokens to deposit and add to the lock
    """
    self.assert_not_contract(msg.sender)
    _locked: LockedBalance = self.locked[msg.sender]

    assert _value > 0  # dev: need non-zero value
    assert _locked.amount > 0, "No existing lock found"
    assert _locked.end > block.timestamp, "Cannot add to expired lock. Withdraw"

    self._deposit_for(msg.sender, _value, 0, _locked, INCREASE_LOCK_AMOUNT)
```
2) ## Lack of ``blockNumber`` value check on ``veOLAS._findPointByBlock``
The ``veOLAS`` contract exhibits a vulnerability in its [_findPointByBlock](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L542-L587) function, characterized by the absence of a essential check present in the ``veCRV`` contract. Specifically, the check on the ``blockNumber`` parameter, where in ``veCRV``, it is compared to ``block.number``, and if it is greater, the function call reverts.
This essential control is missing in ``veOLAS``, allowing the function to proceed without reverting. This absence of verification poses a risk to calling functions such as ``balanceOfAt``, ``getPastVotes``, and ``getPastTotalSupply``, potentially leading to functional issues.
It is advised to address this vulnerability promptly by incorporating the necessary block number check within the ``_findPointByBlock`` function in the ``veOLAS`` contract to ensure consistency and prevent potential complications.
### veOLAS
```
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt) {
        // Find point with the closest block number to the provided one
        (PointVoting memory sPoint, ) = _findPointByBlock(blockNumber, address(0));
        // If the block number at the point index is bigger than the specified block number, the balance was zero
        if (sPoint.blockNumber < (blockNumber + 1)) {
            supplyAt = uint256(sPoint.balance);
        }
    }

function _findPointByBlock(uint256 blockNumber, address account) internal view
        returns (PointVoting memory point, uint256 minPointNumber)
    {
        /////////////////////////////////////////////////////
        ///@audit -- here should be added blockNumber control
        /////////////////////////////////////////////////////
        // Get the last available point number
        uint256 maxPointNumber;
        if (account == address(0)) {
            maxPointNumber = totalNumPoints;
        } else {
            maxPointNumber = mapUserPoints[account].length;
            if (maxPointNumber == 0) {
                return (point, minPointNumber);
            }
            // Already checked for > 0 in this case
            unchecked {
                maxPointNumber -= 1;
            }
        }

        // Binary search that will be always enough for 128-bit numbers
        for (uint256 i = 0; i < 128; ++i) {
            if ((minPointNumber + 1) > maxPointNumber) {
                break;
            }
            uint256 mid = (minPointNumber + maxPointNumber + 1) / 2;

            // Choose the source of points
            if (account == address(0)) {
                point = mapSupplyPoints[mid];
            } else {
                point = mapUserPoints[account][mid];
            }

            if (point.blockNumber < (blockNumber + 1)) {
                minPointNumber = mid;
            } else {
                maxPointNumber = mid - 1;
            }
        }

        // Get the found point
        if (account == address(0)) {
            point = mapSupplyPoints[minPointNumber];
        } else {
            point = mapUserPoints[account][minPointNumber];
        }
    }
```
### veCRV
```
@external
@view
def totalSupplyAt(_block: uint256) -> uint256:
    """
    @notice Calculate total voting power at some point in the past
    @param _block Block to calculate the total voting power at
    @return Total voting power at `_block`
    """
    assert _block <= block.number
    _epoch: uint256 = self.epoch
    target_epoch: uint256 = self.find_block_epoch(_block, _epoch)

    point: Point = self.point_history[target_epoch]
    dt: uint256 = 0
    if target_epoch < _epoch:
        point_next: Point = self.point_history[target_epoch + 1]
        if point.blk != point_next.blk:
            dt = (_block - point.blk) * (point_next.ts - point.ts) / (point_next.blk - point.blk)
    else:
        if point.blk != block.number:
            dt = (_block - point.blk) * (block.timestamp - point.ts) / (block.number - point.blk)
    # Now dt contains info on how far are we beyond point

    return self.supply_at(point, point.ts + dt)
```
3) ## ``veOLAS.increaseUnlockTime`` will overincrease the unlockTime
code affected: https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L487
In the ``veOLAS`` contract, specifically in the ``increaseUnlockTime`` function, a discrepancy exists in the time calculation compared to its counterpart in the ``veCRV`` contract. In ``veOLAS``, the calculation is expressed as follows:
```
unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
```
However, in ``veCRV``, a simpler and seemingly correct calculation is used:
```
(_unlock_time / WEEK) * WEEK
```
Given that the comment above the function reads "@param unlockTime New tokens unlock time," it suggests that ``unlockTime`` represents the new unlock time directly. To ensure consistency and prevent user errors, the ``veOLAS`` contract should adopt the same calculation as ``veCRV``:
```
unlockTime = (unlockTime / WEEK) * WEEK;
```
# RegistriesManager
1) ## Uniqueness Check Missing in ``RegistriesManager`` for Unit Hashes
The ``UnitRegistry`` contract is afflicted by a low-vulnerability bug pertaining to the absence of uniqueness checks for unit IPFS hashes. Currently, there is a deficiency in verifying that each hash remains unique for every unit stored within the registry. This deficiency introduces a potential inconsistency where two units may inadvertently share the same IPFS hash, resulting in data integrity concerns.
To rectify this low-vulnerability bug:
first, Introduce a mapping mechanism, uniqueHashes, to track the uniqueness of unit hashes:
```
mapping(bytes32 => bool) private uniqueHashes;
```
second, Augment the create function to include a uniqueness check for unit hashes:
```
if (uniqueHashes[unitHash]) {
    revert DuplicateUnitHash(unitType, unitHash);
}
uniqueHashes[unitHash] = true;
```
third, Enhance the updateHash function to include a uniqueness check for updated hashes:
```
if (uniqueHashes[unitHash]) {
    revert DuplicateUnitHash(unitType, unitHash);
}
uniqueHashes[unitHash] = true;
```
By implementing these changes, the bug can be addressed, mitigating the risk of unintentional hash duplication and bolstering the overall consistency and integrity of unit data within the registry.
# Tokenomics
1) ## Incorrect Calculation of ``Tokenomics`` in ``_trackServiceDonations`` Function
The bug occurs in the ``_trackServiceDonations`` function within the ``Tokenomics`` contract. Specifically, in the block of code where incentives are processed for units within a service, there is an error in calculating the ``pendingRelativeTopUp`` and ``sumUnitTopUpsOLAS`` values. The issue arises from adding the amount (ETH) to these variables, whereas it should be added in ``OLAS``, leading to incorrect tokenomics calculations.
```
if (topUpEligible && incentiveFlags[unitType + 2]) {
    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;  // Bug here
    mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;  // Bug here
}
```
### Impact
The bug affects the accuracy of the tokenomics calculations related to top-ups within the ``Tokenomics`` contract. As amount is added to ``pendingRelativeTopUp`` and ``sumUnitTopUpsOLAS`` without converting it to ``OLAS``, the resulting values are incorrect, leading to potential discrepancies in the overall tokenomics of the system.
### Expected Behavior
The amount should be converted to ``OLAS`` before being added to ``pendingRelativeTopUp`` and ``sumUnitTopUpsOLAS``. This conversion is crucial to maintain consistency with the tokenomics structure, where these variables are denominated in ``OLAS``.
### Proposed Fix
get the amountInOLAS from the ETH amount and then add it up to those two variables