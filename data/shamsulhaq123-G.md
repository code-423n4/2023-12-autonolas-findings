## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  ## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106## Gas optimization

## Summary


|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use constants instead of type(uintx).max|13|
|[G-02]|Unnecessary look up in if condition|30|
|[G-03]|Use nested if statements instead of &&|15|
|[G-04]|Gas saving is achieved by removing the delete keyword (~60k)|3|
|[G-05]|+= costs more gas than = + for state variables|31|
|[G-06]|Use selfbalance() instead of address(this).balance|2|
|[G-07]|Use hardcode address instead of address(this)|8|
|[G-08]|Public Functions To External|17|
|[G-09]| Use != 0 instead of > 0 for unsigned integer comparison|1|
|[G-10]|Use assembly to perform efficient back-to-back calls|8|
|[G-11]|Using a positive conditional flow to save a NOT opcode|24|
|[G-12]|Avoid emitting constants.|21|

## [G-01] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

392    if (amount > type(uint96).max) {
393          revert Overflow(amount, type(uint96).max);
        }

449  if (amount > type(uint96).max) {
450         revert Overflow(amount, type(uint96).max);
        }   

474  if (amount > type(uint96).max) {
475        revert Overflow(amount, type(uint96).max);
        }             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449-L450
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L475

```solidity

file: registries/contracts/UnitRegistry.so

232     uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232

```solidity

file: tokenomics/contracts/Depository.sol

195  if (priceLP > type(uint160).max) {
196         revert Overflow(priceLP, type(uint160).max);
        }

205  if (supply > type(uint96).max) {
206        revert Overflow(supply, type(uint96).max);
        } 

216  if (maturity > type(uint32).max) {
217      revert Overflow(maturity, type(uint32).max);
        }      
311  if (maturity > type(uint32).max) {
312            revert Overflow(maturity, type(uint32).max);
        }                 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L205-L206
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216-217
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311-312

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol

59  if (totalTokenValue > type(uint192).max) {
60     revert Overflow(totalTokenValue, type(uint192).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60

```solidity

file: tokenomics/contracts/Tokenomics.sol

639  if (eBond > type(uint96).max) {
640         revert Overflow(eBond, type(uint96).max);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

52  address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

## [G-02]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: governance/contracts/wveOLAS.sol

147  if (_ve == address(0) || _token == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L147

```solidity

file: governance/contracts/multisigs/GuardCM.sol

144 if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
            revert ZeroAddress();
        }
259  if (chainId == 100 || chainId == 10200)   

304    if (chainId == 137 || chainId == 80001)

418  if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) 

453   if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) 

506   if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) 

513      if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0))
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L144
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L259
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L304
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L418
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L453
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L506
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L513

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

64  if (_fxChild == address(0) || _rootGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L64

```solidity

file: governance/contracts/bridges/HomeMediator.sol

64    if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L64

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

39   if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0))

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L39

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

42    if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L42

```solidity

file: registries/contracts/ComponentRegistry.sol

30  if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity

file: registries/contracts/AgentRegistry.sol

41      if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```soliodity

file: tokenomics/contracts/Depository.sol

111  if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) 

363   if (pay == 0 || !matured) 

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 

453    if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L111
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L363
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L453

```solidity

file: tokenomics/contracts/Dispenser.sol

36   if (_tokenomics == address(0) || _treasury == address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L36

```solidity

file: tokenomics/contracts/GenericBondCalculator.sol#

31  if (_olas == address(0) || _tokenomics == address(0)) 

82    if (token0 == olas || token1 == olas)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L31
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L82

```solidity

file: tokenomics/contracts/Tokenomics.so

283  if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
            _serviceRegistry == address(0)) {
            revert ZeroAddress();
        }
707  if (incentiveFlags[2] || incentiveFlags[3]) 

709   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
724     if (incentiveFlags[unitType] || incentiveFlags[unitType + 2])   

902  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) 

1063   if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) 

1117   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])

1187   if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]])
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L283
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L707
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L724
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L902
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1063
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.

110     if (positionData.tickLowerIndex != minTickLowerIndex || positionData.tickUpperIndex != maxTickLowerIndex)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L110

## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: governance/contracts/veOLAS.sol

188   if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) 

192   if (newLocked.endTime > block.timestamp && newLocked.amount > 0)

306 if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306

```solidity

file: governance/contracts/wveOLAS.sol

215   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 

235   if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L235

```solidity

file: governance/contracts/multisigs/GuardCM.sol

519    if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity

file: registries/contracts/GenericRegistry.sol

73    return unitId > 0 && unitId < (totalSupply + 1);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

```solidity

file: tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) 


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity

file: tokenomics/contracts/Tokenomics.sol

529  if (_epsilonRate > 0 && _epsilonRate <= 17e18) 

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR)

750    if (topUpEligible && incentiveFlags[unitType + 2]) 

801     if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) 

1138   if (lastEpoch > 0 && lastEpoch < curEpoch)

1206    if (lastEpoch > 0 && lastEpoch < curEpoch)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L536
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L750
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L801
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1138
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1206

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

## [G-04] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: tokenomics/contracts/Depository.sol

264    delete mapBondProducts[productId];

341    delete mapBondProducts[productId];

379  delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L264
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L341
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L379

## [G-05] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: governance/contracts/OLAS.sol

109   supplyCap += (supplyCap * maxMintCapFraction) / 100;

148   spenderAllowance += amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L148

```solidity

file: governance/contracts/veOLAS.sol

237   tStep += WEEK;

246   lastPoint.slope += dSlope;

263   curNumPoint += 1;    

280 lastPoint.slope += (uNew.slope - uOld.slope);
281   lastPoint.bias += (uNew.bias - uOld.bias

299  oldDSlope += uOld.slope;

350   lockedBalance.amount += uint128(amount);

664   blockTime += (dt * (blockNumber - point.blockNumber)) / dBlock;

696   tStep += WEEK;

708     lastPoint.slope += dSlope;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L246
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L280-281 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L299
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L350
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L664
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L708

```solidity

file: governance/contracts/multisigs/GuardCM.sol

241   i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L241

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

157  i += payloadLength;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L157

```solidity

file: governance/contracts/bridges/HomeMediator.sol

157  i += payloadLength;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L157

```solidity

file: tokenomics/contracts/Depository.sol

373  payout += pay;

460   payout += mapUserBonds[i].payout;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L373
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity

file: tokenomics/contracts/Tokenomics.sol

745    mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

751   mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
752   mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

817  donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

937  inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1021 inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1037 curMaxBond += effectiveBond;

1145 reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1148    topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1213  reward += totalIncentives / 100;

1224  topUp += totalIncentives / sumUnitIncentiv

1229  reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
 1231           topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L751-L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L817
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L937
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1021
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1145
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1148
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1213
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1229-L1231

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

56    supplyCap += (supplyCap * maxMintCapFraction) / 100;

93   supplyCap += (supplyCap * maxMintCapFraction) / 100;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L56
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L93

```solidity

file: lockbox-solana/solidity/liquidity_lockbox.sol

189     totalLiquidity += positionLiquidity;

355  liquiditySum += positionLiquidity;asssssssss
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L355

## [G-06]  Use selfbalance() instead of address(this).balance

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

147 if (value > address(this).balance) {
148    revert InsufficientBalance(value, address(this).balance);
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147-L148

```solidity

file: governance/contracts/bridges/HomeMediator.sol

147  if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
            }



```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147-L148

## [G-07]  Use hardcode address instead of address(this)

```solidity

file: governance/contracts/veOLAS.sol

364  IERC20(token).transferFrom(msg.sender, address(this), amount);

768       revert NonTransferable(address(this));
    }

    /// @dev Reverts the approval of this token.
    function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
803        revert NonDelegatable(address(this));
    }
}

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L768-L803

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84   if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84-L85

```solidity

file: governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {
85            revert SelfCallOnly(msg.sender, address(this));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84-L85

```solidity

file: overnance/contracts/bridges/FxERC20ChildTunnel.sol

81            revert TransferFailed(childToken, address(this), to, amount);

102   bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104    revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L104

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

98  bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        if (!success) {
100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98-L100

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: main/governance/contracts/GovernorOLAS.sol

33   function state(uint256 proposalId) public view override(IGovernor, Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
50  public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)

57    function proposalThreshold() public view override(Governor, GovernorSettings) returns (uint256)

105  function supportsInterface(bytes4 interfaceId) public view override(IERC165, Governor, GovernorTimelockControl)
``` 
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L33
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L57
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L105

```solidity

file: governance/contracts/OLAS.sol

91     function inflationControl(uint256 amount) public view returns (bool)

98   function inflationRemainder() public view returns (uint256 remainder) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L91
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L98

```solidity

file: governance/contracts/veOLAS.sol

607   function balanceOf(address account) public view override returns (uint256 balance) 

633    function getVotes(address account) public view override returns (uint256) 

672     function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance

719    function totalSupply() public view override returns (uint256) {
        return supply;     function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {
               
    }
738         function totalSupplyLockedAtT(uint256 ts) public view returns (uint256) {

748    function totalSupplyLocked() public view returns (uint256) 

752     function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L607
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L672
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L719
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L738
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L745
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L752
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761

```solidity

file: /governance/contracts/wveOLAS.sol

193    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L193

```solidity

file: registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135

```solidity

file: tokenomics/contracts/TokenomicsConstants.sol

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L66

## [G-09] Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: registries/contracts/GenericRegistry.sol

73  return unitId > 0 && unitId < (totalSupply + 1);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L73

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: governance/contracts/veOLAS.sol

772     function approve(address spender, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the transferFrom of this token.
    function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
        revert NonTransferable(address(this));
    }

    /// @dev Reverts delegates of this token.
    function delegates(address account) external view virtual override returns (address)
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegate for this token.
    function delegate(address delegatee) external virtual override
    {
        revert NonDelegatable(address(this));
    }

    /// @dev Reverts delegateBySig for this token.
800    function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
        revert NonDelegatable(address(this));
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772-L800

```solidity

file: governance/contracts/wveOLAS.sol

16     function totalNumPoints() external view returns (uint256 numPoints);

    /// @dev Gets the supply point of a specified index.
    /// @param idx Supply point number.
    /// @return sPoint Supply point.
    function mapSupplyPoints(uint256 idx) external view returns (PointVoting memory sPoint);

    /// @dev Gets the slope change for a specific timestamp.
    /// @param ts Timestamp.
    /// @return slopeChange Signed slope change.
    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

    /// @dev Gets the most recently recorded user point for `account`.
    /// @param account Account address.
    /// @return pv Last checkpoint.
    function getLastUserPoint(address account) external view returns (PointVoting memory pv);

    /// @dev Gets the number of user points.
    /// @param account Account address.
    /// @return userNumPoints Number of user points.
    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory uPoint);

    /// @dev Gets voting power at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Voting balance / power.
    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the account balance in native token.
    /// @param account Account address.
    /// @return balance Account balance.
    function balanceOf(address account) external view returns (uint256 balance);

    /// @dev Gets the account balance at a specific block number.
    /// @param account Account address.
    /// @param blockNumber Block number.
    /// @return balance Account balance.
    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

    /// @dev Gets the `account`'s lock end time.
    /// @param account Account address.
    /// @return unlockTime Lock end time.
    function lockedEnd(address account) external view returns (uint256 unlockTime);

    /// @dev Gets the voting power.
    /// @param account Account address.
    /// @return balance Account balance.
    function getVotes(address account) external view returns (uint256 balance);

    /// @dev Gets total token supply.
    /// @return supply Total token supply.
    function totalSupply() external view returns (uint256 supply);

    /// @dev Gets total token supply at a specific block number.
    /// @param blockNumber Block number.
    /// @return supplyAt Supply at the specified block number.
    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

    /// @dev Calculates total voting power at time `ts`.
    /// @param ts Time to get total voting power at.
    /// @return vPower Total voting power.
    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

    /// @dev Calculates current total voting power.
    /// @return vPower Total voting power.
    function totalSupplyLocked() external view returns (uint256 vPower);

    /// @dev Calculate total voting power at some point in the past.
    /// @param blockNumber Block number to calculate the total voting power at.
    /// @return vPower Total voting power.
    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

    /// @dev Gets information about the interface support.
    /// @param interfaceId A specified interface Id.
    /// @return True if this contract implements the interface defined by interfaceId.
    function supportsInterface(bytes4 interfaceId) external view returns (bool);

    /// @dev Reverts the allowance of this token.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Reverts delegates of this token.
104    function delegates(address account) external view returns (address);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16-L104

```solidity

file: governance/contracts/interfaces/IERC20.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
    function totalSupply() external view returns (uint256);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
25    function approve(address spender, uint256 amount) external returns (bool);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/interfaces/IERC20.sol#L9-L25

```solidity

file: registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

    /// @dev Gets subcomponents of a provided unit Id from a local public map.
    /// @param unitId Unit Id.
    /// @return subComponentIds Set of subcomponents.
    /// @return numSubComponents Number of subcomponents.
    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

    /// @dev Calculates the set of subcomponent Ids.
    /// @param unitIds Set of unit Ids.
    /// @return subComponentIds Subcomponent Ids.
    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

    /// @dev Gets updated component / agent hashes.
    /// @param unitId Unit Id.
    /// @return numHashes Number of hashes.
    /// @return unitHashes The list of component / agent hashes.
    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

    /// @dev Gets the total supply of components / agents.
    /// @return Total supply.
48    function totalSupply() external view returns (uint256);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27-L48

```solidity

file: tokenomics/contracts/interfaces/IServiceRegistry.sol

14    function exists(uint256 serviceId) external view returns (bool);

    /// @dev Gets the full set of linearized components / canonical agent Ids for a specified service.
    /// @notice The service must be / have been deployed in order to get the actual data.
    /// @param serviceId Service Id.
    /// @return numUnitIds Number of component / agent Ids.
    /// @return unitIds Set of component / agent Ids.
    function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view
        returns (uint256 numUnitIds, uint32[] memory unitIds);

    /// @dev Gets the value of slashed funds from the service registry.
    /// @return amount Drained amount.
    function slashedFunds() external view returns (uint256 amount);

    /// @dev Drains slashed funds.
    /// @return amount Drained amount.
30    function drain() external returns (uint256 amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IServiceRegistry.sol#L14-L30

```solidity

file: tokenomics/contracts/interfaces/IToken.sol

9   function balanceOf(address account) external view returns (uint256);

    /// @dev Gets the owner of the token Id.
    /// @param tokenId Token Id.
    /// @return Token Id owner address.
    function ownerOf(uint256 tokenId) external view returns (address);

    /// @dev Gets the total amount of tokens stored by the contract.
    /// @return Amount of tokens.
18    function totalSupply() external view returns (uint256);

24   function transfer(address to, uint256 amount) external returns (bool);

    /// @dev Gets remaining number of tokens that the `spender` can transfer on behalf of `owner`.
    /// @param owner Token owner.
    /// @param spender Account address that is able to transfer tokens on behalf of the owner.
    /// @return Token amount allowed to be transferred.
    function allowance(address owner, address spender) external view returns (uint256);

    /// @dev Sets `amount` as the allowance of `spender` over the caller's tokens.
    /// @param spender Account address that will be able to transfer tokens on behalf of the caller.
    /// @param amount Token amount.
    /// @return True if the function execution is successful.
    function approve(address spender, uint256 amount) external returns (bool);

    /// @dev Transfers the token amount that was previously approved up until the maximum allowance.
    /// @param from Account address to transfer from.
    /// @param to Account address to transfer to.
    /// @param amount Amount to transfer to.
    /// @return True if the function execution is successful.
43    function transferFrom(address from, address to, uint256 amount) external returns (bool);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L9-L18
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IToken.sol#L24-L43

```solidity

file: tokenomics/contracts/interfaces/IUniswapV2Pair.sol

6    function totalSupply() external view returns (uint);
    function token0() external view returns (address);
    function token1() external view returns (address);
9   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IUniswapV2Pair.sol#L6-L9

## [G-11] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file:governance/contracts/OLAS.sol

44   if (msg.sender != owner)

59   if (msg.sender != owner) 

77  if (msg.sender != minter)

131  if (spenderAllowance != type(uint256).max) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity

file: governance/contracts/veOLAS.sol

185  if (account != address(0))

278   if (account != address(0)) 

293   if (account != address(0)) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L185
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L293

```solidity

file: governance/contracts/multisigs/GuardCM.sol

155   if (msg.sender != owner) 

171  if (msg.sender != owner

325   if (fxGovernorTunnel != bridgeMediatorL2) 

367   if (bridgeMediatorL2 != address(0)) 

448    if (msg.sender != owner)

501   if (msg.sender != owner)

519     if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001)

562   if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L325
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L367
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

84     if (msg.sender != address(this)) 

109    if(msg.sender != fxChild)

114  if(rootMessageSender != rootGovernor) 

161   if (!success) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L114
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L161

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

32  if (msg.sender != owner)

50      if (msg.sender != owner)

61    if (msg.sender != owner) 
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61

```solidity

file: governance/contracts/bridges/FxERC20ChildTunnel.sol

103   if (!success)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103

```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

99  if (!success) 

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L99

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: governance/contracts/OLAS.sol

53  emit OwnerUpdated(newOwner);

68  emit MinterUpdated(newMinter);

134  emit Approval(msg.sender, spender, spenderAllowance);

150   emit Approval(msg.sender, spender, spenderAllowance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L53
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L68
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L150

```solidity

file: governance/contracts/veOLAS.sol

367    emit Deposit(account, amount, lockedBalance.endTime, depositType, block.timestamp);
368    emit Supply(supplyBefore, supplyAfter);

530 emit Withdraw(msg.sender, amount, block.timestamp);
531   emit Supply(supplyBefore, supplyAfter);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L367-L368
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L530-L531

```solidity

file: governance/contracts/multisigs/GuardCM.sol

165  emit GovernorUpdated(newGovernor);

181  emit GovernorCheckProposalIdChanged(proposalId);

486  emit SetTargetSelectors(targets, selectors, chainIds, statuses);

531     emit SetBridgeMediators(bridgeMediatorL1s, bridgeMediatorL2s, chainIds);

556    emit GuardPaused(msg.sender);

568   emit GuardUnpaused();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L165
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L181
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L486
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L531
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L556
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity

file: governance/contracts/bridges/FxGovernorTunnel.sol

74    emit FundsReceived(msg.sender, msg.value);

94    emit RootGovernorUpdated(newRootGovernor);

167   emit MessageReceived(stateId, rootMessageSender, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L167

```solidity

file: governance/contracts/bridges/HomeMediator.sol

74  emit FundsReceived(msg.sender, msg.value);

94    emit ForeignGovernorUpdated(newForeignGovernor);

167  emit MessageReceived(governor, data);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L74
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L94
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L167

```solidity

file: governance/contracts/bridges/BridgedERC20.sol

42   emit OwnerUpdated(newOwner);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L42


```solidity

file: governance/contracts/bridges/FxERC20RootTunnel.sol

79  
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106
        emit FxDepositERC20(childToken, rootToken, from, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L79
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L106