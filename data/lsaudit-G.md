# [G-01] `getSupplyCapForYear()` and `getInflationForYear()` from `TokenomicsConstants.sol` can be optimized

[File: TokenomicsConstants.sol]()
```
 numYears -= 9;

 [...]

 uint256 maxMintCapFraction = 2;

            // Get the supply cap until the current year
            for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
```

* `maxMintCapFraction` is used only once, thus it's redundant and there's no need to waste gas on declaring this variable
* Since `maxMintCapFraction == 2`, we can use bit-shifts instead of multiplication: `X * 2 == X << 1`
* `numYears -= 9` can be unchecked, since it's inside `if`/`else` condition, which defines that `numYears >= 10)`.

```
unchecked {
 numYears -= 9;	
}
[...]
 for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap << 1) / 100;
            }
}
```

The same recommendation can be implemented in `getInflationForYear()`:





# [G-02] Length of array is calculated multiple of times

[File: DonatorBlacklist.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/DonatorBlacklist.sol#L63-L67)
```
 if (accounts.length != statuses.length) {
            revert WrongArrayLength(accounts.length, statuses.length);
        }

        for (uint256 i = 0; i < accounts.length; ++i) {
```

While bot-race reported that `accounts.length` shouldn't be used inside loop (because `accounts.lenght` will be called during every loop-iteration) - its recommendation to cache `accounts.lenght` before the loop was not fully correct. Since `accounts.lenght` is being calculated multiple of times even before the loop (lines 63, 64, 67) it will be much better idea to cache the length even higher. The same issue occurs for `statutes.length`:

```
uint256 accountsLength = accounts.length;
uint256 statusesLength = statutes.length;

 if (accountsLength != statuses.length) {
            revert WrongArrayLength(accountsLength, statusesLength);
        }

        for (uint256 i = 0; i < accountsLength; ++i) {
```


[File: GuardCM.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L506)
```
if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) {
            revert WrongArrayLength(bridgeMediatorL1s.length, bridgeMediatorL2s.length, chainIds.length, chainIds.length);
[...]
for (uint256 i = 0; i < chainIds.length; ++i) {
```

`bridgeMediatorL1s.length` is calculated 3 times. `bridgeMediatorL2s.length` is calculated 2 times. `chainIds.length` is calculated 4 times.

# [G-03] Change the order of `if`-conditions in function `changeOwner()`

While performing check which might revert - it's recommended to perform the checks which uses less gas first. E.g.:
```
if (expensive_condition) revert();
if (less_expensive_condition) revert();
```

If `less_expensive_condition` will revert, we will waste gas on `expensive_condition`. This condition should be re-written to:

```
if (less_expensive_condition) revert();
if (expensive_condition) revert();
```
That way, when `less_expensive_condition` will revert, we won't be wasting gas on evaluating `expensive_condition`.

In function `changeOwner()`, in `BridgedERC20.sol`:

[File: BridgedERC20.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/BridgedERC20.sol#L32)
```
  if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Zero address check
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }
```

we should change the order of `if`-conditions to:

```
   // Zero address check
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

  if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

       
```

Reading `owner` state variable is more gas expensive than reading `newOwner`, thus this condition should be last.

The same issue occurs in:

`Treasury.sol`: `changeOwner()`, `changeMinAcceptedETH()`.
`Tokenomics.sol`: `changeTokenomicsImplementation()`, `changeOwner()`
`Depository.sol`: `changeOwner()`
`Dispenser.sol`: `changeOwner()`


# [G-04] Merge `mapTokenReserves` and `mapEnabledTokens` from `Treasury` into one mapping

[File: Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L85)
```
    // Token address => token reserves
    mapping(address => uint256) public mapTokenReserves;
    // Token address => enabled / disabled status
    mapping(address => bool) public mapEnabledTokens;
```

Both mappings shares the same key (`token)`, which mean they can be merged into one mapping.


# [G-05] Do not cache variables used only once

* [File: Treasury](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L224)
```
        uint256 reserves = mapTokenReserves[token] + tokenAmount;
        mapTokenReserves[token] = reserves;
```

Variable `reserves` is used only once, which means there's no need to declare it at all:

```
mapTokenReserves[token] = mapTokenReserves[token] + tokenAmount;
```

[File: GnosisSafeSameAddressMultisig.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L103)
```
 bytes32 multisigProxyHash = keccak256(multisig.code);
        if (proxyHash != multisigProxyHash) {
```

Variable `multisigProxyHash` is used only once, which means there's no need to declare it at all:

```
 if (proxyHash != keccak256(multisig.code)) {
```


[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L674)
```
            uint256 sumUnitIncentives = uint256(mapEpochTokenomics[epochNum].unitPoints[unitType].sumUnitTopUpsOLAS) * 100;
            totalIncentives = mapUnitIncentives[unitType][unitId].topUp + totalIncentives / sumUnitIncentives;
```

Variable `sumUnitIncentives` is used only once, which means there's no need to declare it at all:

```
totalIncentives = mapUnitIncentives[unitType][unitId].topUp + totalIncentives / (uint256(mapEpochTokenomics[epochNum].unitPoints[unitType].sumUnitTopUpsOLAS) * 100);
```


[File: OLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L99)
```
 uint256 _totalSupply = totalSupply;
 [...]
 remainder = supplyCap - _totalSupply;
```

`_totalSupply` is used only once, thus there's no need to declare this variable:

```
remainder = supplyCap - totalSupply;
```

[File: OLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L92)
```
        uint256 remainder = inflationRemainder();
        return (amount <= remainder);
```

`remained` is used only once, thus there's no need to declare this variable:

```
return (amount <= inflationRemainder());
```



# [G-06] Some calculations can be unchecked

[File: Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L333)
```
if (amountOwned >= tokenAmount) {
                // This branch is used to transfer ETH to a specified address
                amountOwned -= tokenAmount;

                [...]

                if (reserves >= tokenAmount) {
                reserves -= tokenAmount;

                [...]

            if (accountRewards > 0 && amountETHFromServices >= accountRewards) {
            amountETHFromServices -= accountRewards;
```

Because `amountOwned >= tokenAmount`, `amountOwned -= tokenAmount` can be unchecked.
Because `reserves >= tokenAmount`, `reserves -= tokenAmount` can be unchecked.
Because `amountETHFromServices >= accountRewards`, `amountETHFromServices -= accountRewards` can be unchecked.


[File: GnosisSafeMultisig.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeMultisig.sol#L76)
```
if (dataLength > DEFAULT_DATA_LENGTH) {
                uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
```

Because `dataLength > DEFAULT_DATA_LENGTH`, `dataLength - DEFAULT_DATA_LENGTH` can be unchecked.

[File: GnosisSafeSameAddressMultisig.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L110)
```
 if (dataLength > DEFAULT_DATA_LENGTH) {
            uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
```
Because `dataLength > DEFAULT_DATA_LENGTH`, `dataLength - DEFAULT_DATA_LENGTH` can be unchecked.


[File: Depository.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L323)
```
 if (payout > supply) {
            revert ProductSupplyLow(token, productId, payout, supply);
        }

        // Decrease the supply for the amount of payout
        supply -= payout;
```

Because `payout <= supply`, `supply -= payout` can be unchecked.

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L617)
```
 if (eBond >= amount) {
            // The effective bond value is adjusted with the amount that is reserved for bonding
            // The unrealized part of the bonding amount will be returned when the bonding program is closed
            eBond -= amount;
```

Because `eBond >= amount`, `eBond -= amount` can be unchecked.

[File: OLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L101)
```
uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
```

Because `timeLaunch` is set to `block.timestamp` in the `constructor()`, the current `block.timestamp >=` the `timeLaunch`, thus `(block.timestamp - timeLaunch) / oneYear` can be unchecked.


[File: OLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L105)
```
        if (numYears > 9) {
            // Number of years after ten years have passed (including ongoing ones)
            numYears -= 9;
```

Because `numYears > 9`, `numYears -= 9` can be unchecked.

# [G-07] `rebalanceTreasury()` from `Treasury.sol` can be optimized

* `success = true` can be moved inside `if`-condition
* `else` can be removed

[File: Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L440)
```
 success = true;
        if (treasuryRewards > 0) {
            uint256 amountETHFromServices = ETHFromServices;
            if (amountETHFromServices >= treasuryRewards) {
[...]
            } else {
                // There is not enough amount from services to allocate to the treasury
                success = false;
            }
```

The default (uninitialized) `success` value is `false`. This means, there's no need to assign it to `true` first, and then - when condition won't be met, re-assign it to `false`. Much better idea would be to assign it only once, inside `if`-condition. That way, we don't need to re-assign it to `false` in a `else` condition - thus `else` condition becomes redundant and can be removed.

```
 
        if (treasuryRewards > 0) {
            uint256 amountETHFromServices = ETHFromServices;
            if (amountETHFromServices >= treasuryRewards) {
            success = true;
[...]
            } /* else {
                // There is not enough amount from services to allocate to the treasury
                success = false;
            } */ - @audit - no need for else
```


# [G-08] Use `bytesN` insted of `string`

[File: GenericRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L33)
```
string public constant CID_PREFIX = "f01701220";
```

`CID` has a constant length which is smaller than 32-characters, which implies, it can be represented as `bytes` instead of `string`.


# [G-09] Use `<=` instead of `< + 1`

When `A, B` are `uint`, we know that `A < B + 1` is the same as `A <= B`, thus there's no need to waste gas on additional `+` operation.



A quick test in Remix-IDE was performed:

```
function aaa(uint a, uint b) external {

    uint gas = gasleft();
    bool x = a < b + 1;
    console.log(gas - gasleft());

}

function bbb(uint a, uint b) external {

    uint gas = gasleft();
    bool x = a <= b;
    console.log(gas - gasleft());

}
```

Function `aaa()` uses 98 gas, while function `bbb()` uses 22 gas. This proves, that every instance of `A < B + 1` should be changed to `A <= B`.

```
governance/contracts/veOLAS.sol:387:        if (lockedBalance.endTime < (block.timestamp + 1)) {
governance/contracts/veOLAS.sol:441:        if (unlockTime < (block.timestamp + 1)) {
governance/contracts/veOLAS.sol:469:        if (lockedBalance.endTime < (block.timestamp + 1)) {
governance/contracts/veOLAS.sol:494:        if (lockedBalance.endTime < (block.timestamp + 1)) {
governance/contracts/veOLAS.sol:498:        if (unlockTime < (lockedBalance.endTime + 1)) {
governance/contracts/veOLAS.sol:574:            if (point.blockNumber < (blockNumber + 1)) {
governance/contracts/veOLAS.sol:626:        if (uPoint.blockNumber < (blockNumber + 1)) {
governance/contracts/veOLAS.sol:730:        if (sPoint.blockNumber < (blockNumber + 1)) {
registries/contracts/AgentRegistry.sol:41:            if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) {
registries/contracts/ComponentRegistry.sol:30:            if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) {
registries/contracts/GenericRegistry.sol:73:        return unitId > 0 && unitId < (totalSupply + 1);
```

# [G-10] Change the order of `if`-conditions in `updateHash()` in `UnitRegistry.sol`

While performing check which might revert - it's recommended to perform the checks which uses less gas first. E.g.:
```
if (expensive_condition) revert();
if (less_expensive_condition) revert();
```

If `less_expensive_condition` will revert, we will waste gas on `expensive_condition`. This condition should be re-written to:

```
if (less_expensive_condition) revert();
if (expensive_condition) revert();
```
That way, when `less_expensive_condition` will revert, we won't be wasting gas on evaluating `expensive_condition`.

In function `updateHash()`:

[File: UnitRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L125)
```
  if (manager != msg.sender) {
            revert ManagerOnly(msg.sender, manager);
        }

       [...]
        if (ownerOf(unitId) != unitOwner) {
            if (unitType == UnitType.Component) {
                revert ComponentNotFound(unitId);
            } else {
                revert AgentNotFound(unitId);
            }
        }

        [...]
        if (unitHash == 0) {
            revert ZeroValue();
        }
```

reading state variable `manager` is more expensive than reading `unitHash`. Similarly, `ownerOf(unitId)` is more expensive than reading `unitHash`. This implies, that function should be rewritten to:

```
 if (unitHash == 0) {
            revert ZeroValue();
        }

        [...]

     if (ownerOf(unitId) != unitOwner) {
            if (unitType == UnitType.Component) {
                revert ComponentNotFound(unitId);
            } else {
                revert AgentNotFound(unitId);
            }
        }

        [...]

    if (manager != msg.sender) {
            revert ManagerOnly(msg.sender, manager);
        }
```


# [G-11] Do not perform any operation before checking if function should revert

[File: Dispenser.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Dispenser.sol#L30)
```
  constructor(address _tokenomics, address _treasury)
    {
        owner = msg.sender;
        _locked = 1;

        // Check for at least one zero contract address
        if (_tokenomics == address(0) || _treasury == address(0)) {
            revert ZeroAddress();
        }
```

`owner = msg.sender` and `_locked = 1` should be moved after `if (_tokenomics == address(0) || _treasury == address(0))`. When `constructor()` will revert, it will be a waste of gas on calling `owner = msg.sender` and `_locked = 1` operations.

We should execute operations only when we are sure that revert won't occur.

# [G-12] Use OR instead of addition

[File: Dispenser.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Dispenser.sol#L103)
```
if ((reward + topUp) > 0) {
```

should be changed to: `if (reward > 0 || totUp > 0) {`


# [G-13] Some conditions should be nested in `claimOwnerIncentives()` in `Dispenser.sol`

[File: Dispenser.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Dispenser.sol#L101)
```
 bool success;
        // Request treasury to transfer funds to msg.sender if reward > 0 or topUp > 0
        if ((reward + topUp) > 0) {
            success = ITreasury(treasury).withdrawToAccount(msg.sender, reward, topUp);
        }

        // Check if the claim is successful and has at least one non-zero incentive.
        if (!success) {
            revert ClaimIncentivesFailed(msg.sender, reward, topUp);
        }

        emit IncentivesClaimed(msg.sender, reward, topUp);
```

When `reward + topUp == 0`, variable `success` will be still false, thus there's no need to check it later. This condition should be nested:

```
if ((reward + topUp) > 0) {
            bool success = ITreasury(treasury).withdrawToAccount(msg.sender, reward, topUp);
            if (!success) {
            revert ClaimIncentivesFailed(msg.sender, reward, topUp);
        }

            emit IncentivesClaimed(msg.sender, reward, topUp);
        }
```

Please notice, that when `reward + topUp` is 0, we won't be calling `ITreasury(treasury).withdrawToAccount(msg.sender, reward, topUp)` thus there's no need to check `success` - since it's always false.


# [G-14] Do not initialize variables inside the loop

[File: Depository.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L255)
```
 for (uint256 i = 0; i < numProducts; ++i) {
            uint256 productId = productIds[i];
            // Check if the product is still open by getting its supply amount
            uint256 supply = mapBondProducts[productId].supply;
           [...]
                address token = mapBondProducts[productId].token;
```

Move variable initialization outside the loop:

```
 uint256 productId;
 uint256 supply;
 address token;
 for (uint256 i = 0; i < numProducts; ++i) {
            productId = productIds[i];
            // Check if the product is still open by getting its supply amount
             supply = mapBondProducts[productId].supply;
           [...]
                 token = mapBondProducts[productId].token;
```

That way, variables won't be initialized on every loop iteration.

[File: Depository.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L357)
```
  for (uint256 i = 0; i < bondIds.length; ++i) {
            // Get the amount to pay and the maturity status
            uint256 pay = mapUserBonds[bondIds[i]].payout;
            bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;
            [...]

             uint256 productId = mapUserBonds[bondIds[i]].productId;
```

should be changed to:

```
  uint256 pay;
  bool matured;
  uint256 productId;
  for (uint256 i = 0; i < bondIds.length; ++i) {
            // Get the amount to pay and the maturity status
            pay = mapUserBonds[bondIds[i]].payout;
            matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;
            [...]

             productId = mapUserBonds[bondIds[i]].productId;
```


[File: GuardCM.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L212)
```
 for (uint256 i = 0; i < data.length;) {
            address target;
            uint32 payloadLength;
```

should be changed to:

```
address target;
uint32 payloadLength;
for (uint256 i = 0; i < data.length;) {
            target = address(0);
            payloadLength = 0;
```

[File: GuardCM.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L511)
```
 for (uint256 i = 0; i < chainIds.length; ++i) {
 [...]
 uint256 chainId = chainIds[i];
 [...]
 uint256 bridgeMediatorL2ChainId = uint256(uint160(bridgeMediatorL2s[i]));
```

should be changed to:

```
 uint256 chainId;
 uint256 bridgeMediatorL2ChainId;
 for (uint256 i = 0; i < chainIds.length; ++i) {
 [...]
 chainId = chainIds[i];
 [...]
 bridgeMediatorL2ChainId = uint256(uint160(bridgeMediatorL2s[i]));
```


[File: HomeMediator.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/HomeMediator.sol#L125)
```
 for (uint256 i = 0; i < dataLength;) {
            address target;
            uint96 value;
            uint32 payloadLength;
```

should be changed to:

```
address target;
uint96 value;
uint32 payloadLength;
 for (uint256 i = 0; i < dataLength;) {
             target = address(0);
             value = 0;
             payloadLength = 0;
```

[File: FxGovernorTunnel.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxGovernorTunnel.sol#L125)
```
 for (uint256 i = 0; i < dataLength;) {
            address target;
            uint96 value;
            uint32 payloadLength;
```

should be changed to:

```
address target;
uint96 value;
uint32 payloadLength;
 for (uint256 i = 0; i < dataLength;) {
             target = address(0);
             value = 0;
             payloadLength = 0;
```

# [G-15] Combine multiple events into one event

 We can combine the events into one singular event to save gas that would otherwise be paid for the additional event

 Similar issue has been reported during the previous Code4rena contests, e.g., [here](https://code4rena.com/reports/2023-06-stader#g-14-combine-events-to-save-2-glogtopic-375-gas).

[File: Depository.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L340)
```
if (supply == 0) {
            delete mapBondProducts[productId];
            emit CloseProduct(token, productId, supply);
        }

        emit CreateBond(token, productId, msg.sender, bondId, payout, tokenAmount, maturity);
```

Create new event, e.g., `CloseProductAndCreateBond()` and emit it when `supply == 0`. When `supply == 0`, we will save gas on emitting only one event, instead of two:

```
if (supply == 0) {
            delete mapBondProducts[productId];
            emit CloseProductAndCreateBond([...]);
        }

     else {   emit CreateBond(token, productId, msg.sender, bondId, payout, tokenAmount, maturity); }
```

[File: veOLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L367)
```
        emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
        emit Supply(supplyBefore, supplyAfter);
```

Create new event, e.g., `DepositAndSupply()` and emit it instead of two events.


[File: veOLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L530)
```
        emit Withdraw(msg.sender, amount, block.timestamp);
        emit Supply(supplyBefore, supplyAfter);
```

Create new event, e.g., `WithdrawAndSupply()` and emit it instead of two events.

# [G-16] Reduce number of assignments in multiplication


`X *= A * B` costs less gas than `X *= A; X *= B;` because we do not assign the result to `X` twice.

This means, that below code:

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L672)
```
totalIncentives *= mapEpochTokenomics[epochNum].epochPoint.totalTopUpsOLAS;
totalIncentives *= mapEpochTokenomics[epochNum].unitPoints[unitType].topUpUnitFraction;
```

can be changed to:

```
totalIncentives *= mapEpochTokenomics[epochNum].epochPoint.totalTopUpsOLAS * mapEpochTokenomics[epochNum].unitPoints[unitType].topUpUnitFraction;
```


# [G-17] Small loops can be coded directly

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L714)
```
for (uint256 unitType = 0; unitType < 2; ++unitType) {
```

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L978)
```
for (uint256 i = 0; i < 2; ++i) {
```

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#1104)
```
for (uint256 i = 0; i < 2; ++i) {
```

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#1174)
```
 for (uint256 i = 0; i < 2; ++i) {
```

Above loop will iterate only twice (`unitType = 0`, `unitType = 1`). Instead of wasting gas on the loop initialization, the code can be implemented straightforwardly.
 


# [G-18] Do-while loops are more effective than for-loops

```
governance/contracts/bridges/FxGovernorTunnel.sol:125:        for (uint256 i = 0; i < dataLength;) {
governance/contracts/bridges/FxGovernorTunnel.sol:153:            for (uint256 j = 0; j < payloadLength; ++j) {
governance/contracts/bridges/HomeMediator.sol:125:        for (uint256 i = 0; i < dataLength;) {
governance/contracts/bridges/HomeMediator.sol:153:            for (uint256 j = 0; j < payloadLength; ++j) {
governance/contracts/multisigs/GuardCM.sol:212:        for (uint256 i = 0; i < data.length;) {
governance/contracts/multisigs/GuardCM.sol:237:            for (uint256 j = 0; j < payloadLength; ++j) {
governance/contracts/multisigs/GuardCM.sol:273:            for (uint256 i = 0; i < payload.length; ++i) {
governance/contracts/multisigs/GuardCM.sol:292:            for (uint256 i = 0; i < bridgePayload.length; ++i) {
governance/contracts/multisigs/GuardCM.sol:318:            for (uint256 i = 0; i < payload.length; ++i) {
governance/contracts/multisigs/GuardCM.sol:340:        for (uint256 i = 0; i < payload.length; ++i) {
governance/contracts/multisigs/GuardCM.sol:360:        for (uint i = 0; i < targets.length; ++i) {
governance/contracts/multisigs/GuardCM.sol:458:        for (uint256 i = 0; i < targets.length; ++i) {
governance/contracts/multisigs/GuardCM.sol:511:        for (uint256 i = 0; i < chainIds.length; ++i) {
governance/contracts/multisigs/test/MockTimelockCM.sol:26:        for (uint256 i = 0; i < targets.length; ++i) {
governance/contracts/OLAS.sol:108:            for (uint256 i = 0; i < numYears; ++i) {
governance/contracts/veOLAS.sol:232:            for (uint256 i = 0; i < 255; ++i) {
governance/contracts/veOLAS.sol:561:        for (uint256 i = 0; i < 128; ++i) {
governance/contracts/veOLAS.sol:693:        for (uint256 i = 0; i < 255; ++i) {
registries/contracts/AgentRegistry.sol:40:        for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
registries/contracts/ComponentRegistry.sol:29:        for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
registries/contracts/multisigs/GnosisSafeMultisig.sol:79:                for (uint256 i = 0; i < payloadLength; ++i) {
registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol:113:            for (uint256 i = 0; i < payloadLength; ++i) {
registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol:139:        for (uint256 i = 0; i < numOwners; ++i) {
registries/contracts/UnitRegistry.sol:98:            for (uint256 i = 0; i < numSubComponents; ++i) {
registries/contracts/UnitRegistry.sol:211:        for (uint32 i = 0; i < numUnits; ++i) {
registries/contracts/UnitRegistry.sol:227:        for (counter = 0; counter < maxNumComponents; ++counter) {
registries/contracts/UnitRegistry.sol:234:            for (uint32 i = 0; i < numUnits; ++i) {
registries/contracts/UnitRegistry.sol:236:                for (; processedComponents[i] < numComponents[i]; ++processedComponents[i]) {
registries/contracts/UnitRegistry.sol:262:        for (uint32 i = 0; i < counter; ++i) {
tokenomics/contracts/Depository.sol:255:        for (uint256 i = 0; i < numProducts; ++i) {
tokenomics/contracts/Depository.sol:274:        for (uint256 i = 0; i < numClosedProducts; ++i) {
tokenomics/contracts/Depository.sol:357:        for (uint256 i = 0; i < bondIds.length; ++i) {
tokenomics/contracts/Depository.sol:402:        for (uint256 i = 0; i < numProducts; ++i) {
tokenomics/contracts/Depository.sol:413:        for (uint256 i = 0; i < numProducts; ++i) {
tokenomics/contracts/Depository.sol:448:        for (uint256 i = 0; i < numBonds; ++i) {
tokenomics/contracts/Depository.sol:468:        for (uint256 i = 0; i < numBonds; ++i) {
tokenomics/contracts/DonatorBlacklist.sol:67:        for (uint256 i = 0; i < accounts.length; ++i) {
tokenomics/contracts/Tokenomics.sol:702:        for (uint256 i = 0; i < numServices; ++i) {
tokenomics/contracts/Tokenomics.sol:714:            for (uint256 unitType = 0; unitType < 2; ++unitType) {
tokenomics/contracts/Tokenomics.sol:728:                    for (uint256 j = 0; j < numServiceUnits; ++j) {
tokenomics/contracts/Tokenomics.sol:758:                for (uint256 j = 0; j < numServiceUnits; ++j) {
tokenomics/contracts/Tokenomics.sol:808:        for (uint256 i = 0; i < numServices; ++i) {
tokenomics/contracts/Tokenomics.sol:978:            for (uint256 i = 0; i < 2; ++i) {
tokenomics/contracts/Tokenomics.sol:1104:        for (uint256 i = 0; i < 2; ++i) {
tokenomics/contracts/Tokenomics.sol:1110:        for (uint256 i = 0; i < unitIds.length; ++i) {
tokenomics/contracts/Tokenomics.sol:1132:        for (uint256 i = 0; i < unitIds.length; ++i) {
tokenomics/contracts/Tokenomics.sol:1174:        for (uint256 i = 0; i < 2; ++i) {
tokenomics/contracts/Tokenomics.sol:1180:        for (uint256 i = 0; i < unitIds.length; ++i) {
tokenomics/contracts/Tokenomics.sol:1202:        for (uint256 i = 0; i < unitIds.length; ++i) {
tokenomics/contracts/TokenomicsConstants.sol:55:            for (uint256 i = 0; i < numYears; ++i) {
tokenomics/contracts/TokenomicsConstants.sol:92:            for (uint256 i = 1; i < numYears; ++i) {
tokenomics/contracts/Treasury.sol:276:        for (uint256 i = 0; i < numServices; ++i) {
```

# [G-19] Pre-incrementing is more efficient than post-incrementing

A quick test in Remix-IDE was done:

```
  function a() external {
    uint x = 1;
    uint gas = gasleft();
    x++;
    console.log(gas - gasleft());
   }

     function b() external {
    uint x = 1;
    uint gas = gasleft();
    ++x;
    console.log(gas - gasleft());
   }
```

`a()` uses 147 gas, `b()` uses 142 gas. This means, that pre-incrementing is better than post-incrementing.

```
registries/contracts/UnitRegistry.sol:78:        unitId++;
registries/contracts/UnitRegistry.sol:244:                        numComponentsCheck++;
registries/contracts/UnitRegistry.sol:254:                processedComponents[minIdxComponent]++;
tokenomics/contracts/Tokenomics.sol:762:                        mapEpochTokenomics[curEpoch].unitPoints[unitType].numNewUnits++;
tokenomics/contracts/Tokenomics.sol:768:                            mapEpochTokenomics[curEpoch].epochPoint.numNewOwners++;
```

Above variables can be pre-incremented, instead of post-incremented.