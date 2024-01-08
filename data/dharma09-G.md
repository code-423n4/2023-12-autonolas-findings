## [G-01] Computations should be memoized rather than having to re-compute them

In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls to pure functions and returning the cached result when the same inputs occur again.

### 2 Instances

1. We can memoize the computations of `(supplyCap * maxMintCapFraction) / 100`.

We can reduce the gas usage of the `inflationRemainder()` function by memoizing the computations of `(supplyCap * maxMintCapFraction) / 100`. i.e cache the results of their computations so that for subsequent computations we can use the cached values rather than having to recompute their values. The diff below shows how the code could be refactored:

- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```jsx
File: governance/contracts/OLAS.sol
98: function inflationRemainder() public view returns (uint256 remainder) {
        uint256 _totalSupply = totalSupply;
        // Current year
        uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
        // Calculate maximum mint amount to date
        uint256 supplyCap = tenYearSupplyCap;
        // After 10 years, adjust supplyCap according to the yearly inflation % set in maxMintCapFraction
        if (numYears > 9) {
            // Number of years after ten years have passed (including ongoing ones)
            numYears -= 9;
            for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100; //@audit
            }
        }
        // Check for the requested mint overflow
        remainder = supplyCap - _totalSupply;
    }
```

1. We can reduce the gas usage of the `_findPointByBlock()` function by memoizing the computations of `(minPointNumber + maxPointNumber + 1) / 2;`. i.e cache the results of their computations so that for subsequent computations we can use the cached values rather than having to recompute their values. The diff below shows how the code could be refactored:
- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L561

```jsx
File: governance/contracts/veOLAS.sol
561: for (uint256 i = 0; i < 128; ++i) {
            if ((minPointNumber + 1) > maxPointNumber) {
                break;
            }
            uint256 mid = (minPointNumber + maxPointNumber + 1) / 2; //@audit Do calcultion outside loop

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
```

## [G-02] Avoid caching the length of calldata

Caching the length of calldata increases gas cost instead of reducing it

### Proof of Concept

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CacheCallDataLength {

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {
        uint256 len = numbers.length;

        for(uint i; i < len; ++i) {
            total += numbers[i];
        }

    }
}

```

```
test for test/CacheCallDataLength.t.sol:CacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1633)

```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
contract NoCacheCallDataLength {

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {

        for(uint i; i < numbers.length; ++i) {
            total += numbers[i];
        }

    }
}
```

`test for test/NoCacheCallDataLength.t.sol:NoCacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1628)`

- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L117C1-L130C23

```solidity
File: governance/contracts/bridges/HomeMediator.sol
119: uint256 dataLength = data.length; //@audit
        if (dataLength < DEFAULT_DATA_LENGTH) {
            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);
        }

        // Unpack and process the data
        for (uint256 i = 0; i < dataLength;) {
```

## **[G-03] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state**

We can save 1 `SLOAD` `2100` gas units if we use `mapSlopeChanges[oldLocked.endTime]` in place of `oldDSlope` since their values are equal

- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L200C4-L205C36

```jsx
File: /governance/contracts/veOLAS.sol
199: // newLocked.endTime can ONLY be in the FUTURE unless everything is expired: then zeros
            oldDSlope = mapSlopeChanges[oldLocked.endTime];
            if (newLocked.endTime > 0) {
                if (newLocked.endTime == oldLocked.endTime) {
                    newDSlope = oldDSlope; //@audit We can use exting value
                } else {
                    newDSlope = mapSlopeChanges[newLocked.endTime];
                }
            }
```

## [G-04] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L213C8-L217C10

```jsx
File: governance/contracts/WveOLAS.sol
211: function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance) {
        // Get the zero account point
        PointVoting memory uPoint = getUserPoint(account, 0);
        // Check that the point exists and the zero point block number is not smaller than the specified blockNumber
        if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) { //@audit
            balance = IVEOLAS(ve).getPastVotes(account, blockNumber);
        }
    }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235C7-L239C1

```jsx
File: governance/contracts/wveOLAS.sol
231: function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance) {
        // Get the zero account point
        PointVoting memory uPoint = getUserPoint(account, 0);
        // Check that the zero point block number is not smaller than the specified blockNumber
        if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) { //@audit
            balance = IVEOLAS(ve).balanceOfAt(account, blockNumber);
        }
    }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol

```jsx
File: tokenomics/contracts/Tokenomics.sol
529: if (_epsilonRate > 0 && _epsilonRate <= 17e18) { //@audit
            epsilonRate = uint64(_epsilonRate);
        } else {
            _epsilonRate = epsilonRate;
        }

        // Check for the epochLen value to change
        if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) { //@audit
            nextEpochLen = uint32(_epochLen);
        } else {
            _epochLen = epochLen;
        }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol

```
File: tokenomics/contracts/Tokenomics.sol
750: if (topUpEligible && incentiveFlags[unitType + 2]) { //@audit Use nested if
                            mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
                            mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;
                        }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L800C1-L803C10

```jsx
File: tokenomics/contracts/Tokenomics.sol
800: address bList = donatorBlacklist;
        if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) { //@audit Use nested if 
            revert DonatorBlacklisted(donator);
        }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138C11-L1143C1

```jsx
File: tokenomics/contracts/Tokenomics.sol
1138: if (lastEpoch > 0 && lastEpoch < curEpoch) { //@audit
                _finalizeIncentivesForUnitId(lastEpoch, unitTypes[i], unitIds[i]);
                // Change the last epoch number
                mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch = 0;
            }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138C11-L1143C1

```jsx
File: tokenomics/contracts/Treasury.sol
402: if (accountRewards > 0 && amountETHFromServices >= accountRewards) { //@audit
            amountETHFromServices -= accountRewards;
            ETHFromServices = uint96(amountETHFromServices);
            emit Withdraw(ETH_TOKEN_ADDRESS, account, accountRewards);
            (success, ) = account.call{value: accountRewards}("");
            if (!success) {
                revert TransferFailed(address(0), address(this), account, accountRewards);
            }
        }
```

## [G-05] Use do while loop

- https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L260C5-L264C10

```jsx
File: registries/contracts/UnitRegistry.sol
261: subComponentIds = new uint32[](counter);
        for (uint32 i = 0; i < counter; ++i) { //@audit Use do while loop
            subComponentIds[i] = allComponents[i];
        }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L274C3-L276C10

```jsx
File: tokenomics/contracts/Depository.sol
273: closedProductIds = new uint256[](numClosedProducts);
        for (uint256 i = 0; i < numClosedProducts; ++i) { //@audit Use do while loop
            closedProductIds[i] = ids[i];
        }
```

## [G-06] **Refactor functions to avoid unnecessary external calls**

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L255C9-L271C1

```jsx
File: tokenomics/contracts/Depository.sol
255: for (uint256 i = 0; i < numProducts; ++i) {
            uint256 productId = productIds[i];
            // Check if the product is still open by getting its supply amount
            uint256 supply = mapBondProducts[productId].supply;
            // The supply is greater than zero only if the product is active, otherwise it is already closed
            if (supply > 0) {
                // Refund unused OLAS supply from the product if it was not used by the product completely
                ITokenomics(tokenomics).refundFromBondProgram(supply); //@audit External call in loop
                address token = mapBondProducts[productId].token;
                delete mapBondProducts[productId];

                ids[numClosedProducts] = productIds[i];
                ++numClosedProducts;
                emit CloseProduct(token, productId, supply);
            }
        }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L757C10-L770C22

```jsx
File: tokenomics/contracts/Tokenomics.sol
758: for (uint256 j = 0; j < numServiceUnits; ++j) {
                    // Check if the component / agent is used for the first time
                    if (!mapNewUnits[unitType][serviceUnitIds[j]]) {
                        mapNewUnits[unitType][serviceUnitIds[j]] = true;
                        mapEpochTokenomics[curEpoch].unitPoints[unitType].numNewUnits++;
                        // Check if the owner has introduced component / agent for the first time
                        // This is done together with the new unit check, otherwise it could be just a new unit owner
                        address unitOwner = IToken(registries[unitType]).ownerOf(serviceUnitIds[j]); //@note external call in loop
                        if (!mapNewOwners[unitOwner]) {
                            mapNewOwners[unitOwner] = true;
                            mapEpochTokenomics[curEpoch].epochPoint.numNewOwners++;
                        }
                    }
                }
```

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L808C5-L817C13

```jsx
File: tokenomics/contracts/Tokenomics.sol
808: for (uint256 i = 0; i < numServices; ++i) {
            // Check for the service Id existence
            if (!IServiceRegistry(serviceRegistry).exists(serviceIds[i])) { //@note external call in loop
                revert ServiceDoesNotExist(serviceIds[i]);
            }
        }
```

## [G-07] Make storage vriable outside loop

- https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L354C1-L371C1

```jsx
File: tokenomics/contracts/Depository.sol
356: function redeem(uint256[] memory bondIds) external returns (uint256 payout) { //@review
        for (uint256 i = 0; i < bondIds.length; ++i) {
            // Get the amount to pay and the maturity status
            uint256 pay = mapUserBonds[bondIds[i]].payout; //@audit
            bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;

            // Revert if the bond does not exist or is not matured yet
            if (pay == 0 || !matured) {
                revert BondNotRedeemable(bondIds[i]);
            }

            // Check that the msg.sender is the owner of the bond
            if (mapUserBonds[bondIds[i]].account != msg.sender) {
                revert OwnerOnly(msg.sender, mapUserBonds[bondIds[i]].account);
            }

            // Increase the payout
            payout += pay;

            // Get the productId
            uint256 productId = mapUserBonds[bondIds[i]].productId;

            // Delete the Bond struct and release the gas
            delete mapUserBonds[bondIds[i]];
            emit RedeemBond(productId, msg.sender, bondIds[i]);
        }
```