# Olas QA Report

| ID            | Title                                                                                                                   | Severity       |
| ------------- | ----------------------------------------------------------------------------------------------------------------------- | -------------- |
| [L-01](#l-01) | `Depository.create` function can create duplicate products                                                              | Low            |
| [L-02](#l-02) | `Depository.isActiveProduct` function doesn't check if the product exists                                               | Low            |
| [L-03](#l-03) | `Treasury.withdrawToAccount` function shouldn't be paused                                                               | Low            |
| [L-04](#l-04) | `Tokenomics` contract: no check on `veOLASThreshold` value when the implementation is initialized                       | Low            |
| [L-05](#l-05) | Donations made in the new epoch can be counted with the old epoch affecting treasury rewards                            | Low            |
| [L-06](#l-06) | `Tokenomics.getLastIDF` could return an un-updated idf                                                                  | Low            |
| [L-07](#l-07) | `Tokenomics.trackServiceDonations` function: small amounts of donations that round down to zero cause system imbalances | Low            |
| [L-08](#l-08) | The protocol can be griefed and stalled by frontrrunning `Tokenomics.checkpoint` function call                          | Low            |
| [L-09](#l-09) | `GuardCM` can be disabled in the case of a hardfork of the supported L2 chains                                          | Low            |
| [L-10](#l-10) | Some tokens don't return bool with transfer operations                                                                  | Low            |
| [L-11](#l-11) | `Treasury.depositTokenForOLAS` function allows depositing while the contract is paused                                  | Low            |
| [I-01](#i-01) | `Depository.deposit` function lacks minimum OLAS payout determined by the user                                          | Informational  |
| [I-02](#i-02) | `Tokenomics.initializeTokenomics` : some assigned variables values contradict with documentation and Natspec            | Informational  |
| [I-03](#i-03) | `FxERC20ChildTunnel.deposit` function: update Natspec to warn users from using it from a multisig wallets               | Informational  |
| [I-04](#i-04) | Typo in `Depository` contract                                                                                           | Informational  |
| [I-05](#i-05) | Typo in `Tokenomics` contract                                                                                           | Informational  |
| [R-01](#r-01) | `Dispenser.claimOwnerIncentives` function: add another parameter to prevent locking owner rewards                       | Recommendation |
| [R-02](#r-02) | `FxGovernorTunnel` contract: consider adding a mechanism to prevent replaying proposals on Polygon                      | Recommendation |

# Summary

| Severity       | Description                                               | Instances |
| -------------- | --------------------------------------------------------- | --------- |
| Low severity   | Vulnerabilities with medium-low impact and low likelihood | 11        |
| Informational  | Suggestions on best practices and code readability        | 05        |
| Recommendation | Design recommendations                                    | 02        |

# Low Severity Findings

## [L-01] `Depository.create` function can create duplicate products<a id="l-01" ></a>

## Details

- `Depository.create` function is accessed by the contract owner to create a new bonding program by creating a product with an LP token address, its price, its supply and vesting period.

- But this function can create a duplicate product of an existing active product with the same parameters, as there's no check for that.

## Proof of Concept

[Depository.create function](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L183)

```javascript
function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {
```

## Recommendation

Check that the created product is not a duplicate of an already active preoduct twith the same parameters.

## [L-02] `Depository.isActiveProduct` function doesn't check if the product exists<a id="l-02" ></a>

## Details

`Depository.isActiveProduct` function is meant to return the product activity by checking it's residual supply, but it was noticed that there's no check on the provided productId if it is for an existing (created) product or not.

## Proof of Concept

[Depository.isActiveProduct function](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L424-L426)

```javascript
    function isActiveProduct(uint256 productId) external view returns (bool status) {
        status = (mapBondProducts[productId].supply > 0);
    }
```

## Recommendation

```diff
    function isActiveProduct(uint256 productId) external view returns (bool status) {
+       require(productId <= productCounter,"Product doesn't exist");
        status = (mapBondProducts[productId].supply > 0);
    }
```

## [L-03] `Treasury.withdrawToAccount` function shouldn't be paused<a id="l-03" ></a>

## Details

`Treasury.withdrawToAccount` function is intended to allow the `Dispenser` contract to transfer the amounts accountReward and accountTopUps to the address account (service owner account), but this function can't be accessible if the `Treasury` contract is paused, while withdrawals should never be paused in cases of emergencies to allow usere retrieving their rewards/funds.

## Proof of Concept

[Treasury.withdrawToAccount function](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L387C1-L393C10)

```javascript
    function withdrawToAccount(address account, uint256 accountRewards, uint256 accountTopUps) external
        returns (bool success)
    {
        // Check if the contract is paused
        if (paused == 2) {
            revert Paused();
        }
```

## Recommendation

Remaove pause check in `withdrawToAccount` function.

## [L-04] `Tokenomics` contract: no check on `veOLASThreshold` value when the implementation is initialized <a id="l-04" ></a>

## Details

- In `Tokenomics` contract: the `veOLASThreshold` varaible represents the minimum veOlas voting power for the service donor or the service owner to hold in order to make the service owner elligible for top-ups.

- The voting power is calculated based on the locked OLAS token over time; the more time these tokens are locked (vested); the greater the power.

- `veOLASThreshold` value should never be greater than the OLAS totalSupply at any time, but when the `Tokenomics` implementation is initialized; it's assigned a value of `10_000e18` without checking if it exceeds the OLAS totalSupply at the time `Tokenomics` implementation contract is deployed.

- Same issue when `veOLASThreshold` variable is changed by the contract owner via `Tokenomics.changeTokenomicsParameters` function.

- Assigning `veOLASThreshold` with a value greater than veOLAS totalSupply will result in service owners not being able to receive top-ups as neither the donor nor the service owner would satisfy this threshold at early stages:

  ```javascript
  // @@notice Tokenomics._trackServiceDonations function : to check if the service owner is elligible for top-ups rewards
  topUpEligible =
    IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold ||
    IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold
      ? true
      : false;
  ```

## Proof of Concept

[Tokenomics.initializeTokenomics function/L293](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L293)

```javascript
veOLASThreshold = 10_000e18;
```

[Tokenomics.changeTokenomicsParameters function/L542-L547](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L542-L547)

```javascript
// Adjust veOLAS threshold for the next epoch
if (uint96(_veOLASThreshold) > 0) {
  nextVeOLASThreshold = uint96(_veOLASThreshold);
} else {
  _veOLASThreshold = veOLASThreshold;
}
```

## Tools Used

Manual Review.

## Recommended Mitigation Steps

Check new value of `veOLASThreshold` against OLAS totalSupply before assigning it.

## [L-05] Donations made in the new epoch can be counted with the old epoch affecting treasury rewards<a id="l-05" ></a>

## Details

- The protocol allows anyone to deposit ETH donations for services via `Treasury.depositServiceDonationsETH` function, and these donations are stored in `Tokenomics.mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH` which represents the total service balance of the current epoch.

- In `Tokenomics.checkpoint` function: the `treasury` rewards (incentives) are calculated as `totalDonationsETH * rewardsTreasuryFraction / 100` and sent to the `treasury` via `Treasury.rebalanceTreasury` function, where the `rewardsTreasuryFraction` is a parameter of the current epoch (not the new one).

- But if the time for the current epoch ends and the `checkpoint` function hasn't been called to start the next epoch, update the Tokenomics variables and send treasury rewards; any donations made will still be added to the current epoch, which will result in treasury contract getting its rewards for these late donations based on the `rewardsTreasuryFraction` of the current epoch.

## Proof of Concept

[Tokenomics.checkpoint function/L1062-L1071](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L1062-L1071)

```javascript
// Treasury contract rebalances ETH funds depending on the treasury rewards
        if (incentives[1] == 0 || ITreasury(treasury).rebalanceTreasury(incentives[1])) {
            // Emit settled epoch written to the last economics point
            emit EpochSettled(eCounter, incentives[1], accountRewards, accountTopUps);
            // Start new epoch
            epochCounter = uint32(eCounter + 1);
        } else {
            // If the treasury rebalance was not executed correctly, the new epoch does not start
            revert TreasuryRebalanceFailed(eCounter);
        }
```

## Recommendation

Consider adding a mechanism to add any donations made after the end of the current epoch (but before updating to the next epoch) to the next epoch total donations.

## [L-06] `Tokenomics.getLastIDF` could return an un-updated idf<a id="l-06" ></a>

## Details

- `Tokenomics.getLastIDF` function is meant to return the inverse discount factor of the **latest** epoch, and this value is used to calculate the OLAS payout amount when the user deposits LP shares and creates a bond in `Depository.deposit`:

```javascript
// @@notice :GenericBondCalculator.calculatePayoutOLAS function that is called by Depository.deposit
amountOLAS = (ITokenomics(tokenomics).getLastIDF() * totalTokenValue) / 1e36;
```

- The value of the idf is supposed to be changed each epoch via `Tokenomics.checkpoint` function based on the updated values of treasury rewards and the number of new owners (via `_calculateIDF`), and this update is done when the time passed since last epoch end time is greater than the specified epoch length:

  ```javascript
  // @@notice :Tokenomics.checkpoint
  // Check if the time passed since the last epoch end time is bigger than the specified epoch length,
  // but not bigger than a year in seconds
  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) {
    return false;
  }
  ```

- So the protocol could use an outdated idf value if the time for the update is passed and the `checkpoint` hasn't been called to update the Tokenomics variables; which will result in using an outdated idf value when calculating users payout when they deposit their LP share in a product to create bonds.

## Proof of Concept

[Tokenomics.getLastIDF function](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L1260C1-L1268C6)

```javascript
    /// @dev Gets inverse discount factor with the multiple of 1e18 of the last epoch.
    /// @return idf Discount factor with the multiple of 1e18.
    function getLastIDF() external view returns (uint256 idf)
    {
        idf = mapEpochTokenomics[epochCounter - 1].epochPoint.idf;
        if (idf == 0) {
            idf = 1e18;
        }
    }
```

## Recommendation

- Update `Tokenomics.getLastIDF` function to check if a new idf is to be used:

  ```diff
      function getLastIDF() external view returns (uint256 idf)
      {
  +       uint256 prevEpochTime = mapEpochTokenomics[epochCounter - 1].epochPoint.endTime;
  +       uint256 diffNumSeconds = block.timestamp - prevEpochTime;
  +       uint256 curEpochLen = epochLen;

  +       if (diffNumSeconds >= curEpochLen) {
  +          checkpoint();
  +       }

          idf = mapEpochTokenomics[epochCounter - 1].epochPoint.idf;
          if (idf == 0) {
              idf = 1e18;
          }
      }
  ```

  but since `checkpoint` function is only allowed to be called via `TokenomicsProxy` contract; I will leave it for the protocol team to find a solution for this.

- Also front-running issue that aims to revert `checkpoint` function calls when a donation is made in the same block should be considered and mitigated.

## [L-07] `Tokenomics.trackServiceDonations` function: small amounts of donations that round down to zero cause system imbalances<a id="l-07" ></a>

## Details

- The protocol allows anyone to deposit ETH donations for services via `Treasury.depositServiceDonationsETH` function, which will in return call `ITokenomics(tokenomics).trackServiceDonations` to track these donations and distribute it to the service owners.

- Each donation is recorded in `Treasury.ETHFromServices` variable and in `Tokenomics.mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH` that represents the total service balance per epoch, and this value is used later in `checkpoint` function to update tokenomics parameters for the next epoch.

- The `Tokenomics.mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH` is used to:

  1. Update the `effectiveBond` that's used as a supply cap for the created products in the `Depository`
  2. Update the idf of the new epoch that's used to calculate bonds payout when user creates bonds

- In `Tokenomics.trackServiceDonations` function : the donation is distributed for the service owners, and these donations are split into ETH rewards and top-ups rewards (OLAS tokens), the amount of the donation allocated for a service unit owner is calculated based on the number of deployed units of that service:

  ```javascript
  int96 amount = uint96(amounts[i] / numServiceUnits);
  ```

- But the calculated `amount = uint96(amounts[i] / numServiceUnits)` might round down to zero if the allocated amount for that service unit is very small or if the number of units in a service is very large; which will result in actual amount being distributed to the service owners being less than the total amount of the donation, and this creates imbalances in the recorded `totalDonationsETH` as it doesn't reflect the actual donations granted for the owners, so the calculated idf and `effectiveBond` would be incorrect/inaccurate.

## Proof of Concept

[Tokenomics.\_finalizeIncentivesForUnitId function/L725-L726](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L725-L726)

```javascript
// The amount has to be adjusted for the number of units in the service
uint96 amount = uint96(amounts[i] / numServiceUnits);
```

## Recommendation

Check that the sum of the actual amounts donated for unit services is > 0, and assign this value to the `Tokenomics.mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH`, and refund the residual donation to the donor (make sure that the actual donated amounts >= `Treasury.minAcceptedETH`).

## [L-08] The protocol can be griefed and stalled by frontrrunning `Tokenomics.checkpoint` function call<a id="l-08" ></a>

## Details

- `Tokenomics.checkpoint` function is intended to update the system epoch and its tokenomics parameters if the current epoch time has ended and no more than one year has passed from its end:

  ```javascript
          // New point can be calculated only if we passed the number of blocks equal to the epoch length
          uint256 prevEpochTime = mapEpochTokenomics[epochCounter - 1]
              .epochPoint
              .endTime;

          uint256 diffNumSeconds = block.timestamp - prevEpochTime;
          uint256 curEpochLen = epochLen;

          // Check if the time passed since the last epoch end time is bigger than the specified epoch length,
          // but not bigger than a year in seconds
          if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) {
              return false;
          }
  ```

- And this function can't be called if any donation is made in the same `block.timestamp` to avoid the possibility of a flash loan attack (as per the documentation):

  ```javascript
  // Check the last donation block number to avoid the possibility of a flash loan attack
  if (lastDonationBlockNumber == block.number) {
              revert SameBlockNumberViolation();
          }
  ```

- So how can this be exploited?

  1. Since this function is called manually not automatically; it can be left sometime without being called after the end of the current epoch.

  2. Assume that it's left for almost a `block.timestamp - year - 1 second` without being called, and once it's called; a malicious user sees the transaction and frontruns it by depositing a donation where `lastDonationBlockNumber` is set to the current `block.number`, so the `checkpoint` call will return `false` without updating the system parameters.

  3. Another `checkpoint` function call is made in the next block; but it will never update the tokenomics epoch and its parameters as the time passed from the previous epoch is > year, so the contract will be stuck in the last epoch and become stale:

  ```javascript
  // Check if the time passed since the last epoch end time is bigger than the specified epoch length,
  // but not bigger than a year in seconds
  if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) {
    return false;
  }
  ```

## Proof of Concept

[Tokenomics.checkpoint function/L891-L894](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L891-L894)

```javascript
        // Check the last donation block number to avoid the possibility of a flash loan attack
        if (lastDonationBlockNumber == block.number) {
            revert SameBlockNumberViolation();
        }
```

## Recommendation

Add a mechanism to pause the donations whenever `checkpoint` function is needed to be called by the contract owner.

## [L-09] `GuardCM` can be disabled in the case of a hardfork of the supported L2 chains<a id="l-09" ></a>

## Details

- In `GuardCM._processBridgeData` function processes the bridged data by checking the header and verifies the payload, first it checks if the chainId is of one of the supported L2 chains (Gnosis & Polygon) where chainId of 100 or 10200 is for Gnosis and its testnet, 137 and 80001 are for Polygon mainnet and Mumbai testnet respectively:

  ```javascript
  if (chainId == 100 || chainId == 10200) {

      //...

  if (chainId == 137 || chainId == 80001) {
  ```

- But if a hardfork happens in any of the supported chains; its `chain.id` will change, and the comparision made to validate chainId against constant values will be invalid resulting in reverting the transaction and rendering the `GuardCM` inoperable and disabled as it can't handle messages sent frm the forked chain.

## Proof of Concept

[GuardCM.\_processBridgeData function/L259](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L259)

```javascript
if (chainId == 100 || chainId == 10200) {
```

[GuardCM.\_processBridgeData function/L304](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L304)

```javascript
if (chainId == 137 || chainId == 80001) {
```

[GuardCM.setBridgeMediatorChainIdsfunction/L519-L521](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L519-L521)

```javascript
   if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001) {
                revert L2ChainIdNotSupported(chainId);
    }
```

## Recommendation

Assign the L2 chainIds constant (100,137,10200,80001) to state variables or a mapping with a function to update these values if changed in the future due to hardforks.

## [L-10] Some tokens don't return bool with transfer operations<a id="l-10" ></a>

## Details

- `FxERC20ChildTunnel` and `FxERC20RootTunnel` are supposed to facilitate bridging tokens from L2 to L1 and withdrawing these bridged tokens from L1 to L2.

- These contracts handle the tranfers of child and root tokens by assuming that the transfer operation will return a boolean, but this is not the case with some tokens such as USDT token. This will result in these types of tokens not working with these tunnel implementations.

## Proof of Concept

[FxERC20ChildTunnel.\_processMessageFromRoot function/L79](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79)

```javascript
bool success = IERC20(childToken).transfer(to, amount);
```

[FxERC20ChildTunnel.\_processMessageFromRoot function/L102](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102)

```javascript
bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);
```

[FxERC20RootTunnel.\_withdraw function/L98](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20RootTunnel.sol#L98)

```javascript
bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
```

## Recommendation

Use `SafeERC20.safeTransferFrom` to handle tokens transfers.

## [L-11] `Treasury.depositTokenForOLAS` function allows depositing while the contract is paused<a id="l-11" ></a>

## Details

- `Treasury` contract has pausability functionality that enables the owner from pausing the contract in cases of emrgencies, and in such cases; no deposits should be allowed.

- But it was noticed that users can still deposit in the treasury (by calling `Depository.deposit` which will call `Treasury.depositTokenForOLAS` function) even if the contract is paused as the function doesn't have pausability check.

## Proof of Concept

[Treasury.depositTokenForOLAS function](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L212-L245)

```javascript
    function depositTokenForOLAS(address account, uint256 tokenAmount, address token, uint256 olasMintAmount) external {
        // Check for the depository access
        if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }

        // Check if the token is authorized by the registry
        if (!mapEnabledTokens[token]) {
            revert UnauthorizedToken(token);
        }

        // Increase the amount of LP token reserves
        uint256 reserves = mapTokenReserves[token] + tokenAmount;
        mapTokenReserves[token] = reserves;

        // Uniswap allowance implementation does not revert with the accurate message, need to check before the transfer is engaged
        if (IToken(token).allowance(account, address(this)) < tokenAmount) {
            revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);
        }

        // Transfer tokens from account to treasury and add to the token treasury reserves
        // We assume that authorized LP tokens in the protocol are safe as they are enabled via the governance
        // UniswapV2ERC20 realization has a standard transferFrom() function that returns a boolean value
        bool success = IToken(token).transferFrom(account, address(this), tokenAmount);
        if (!success) {
            revert TransferFailed(token, account, address(this), tokenAmount);
        }

        // Mint specified number of OLAS tokens corresponding to tokens bonding deposit
        // The olasMintAmount is guaranteed by the product supply limit, which is limited by the effectiveBond
        IOLAS(olas).mint(msg.sender, olasMintAmount);

        emit DepositTokenFromAccount(account, token, tokenAmount, olasMintAmount);
    }
```

## Tools Used

Manual Review.

## Recommended Mitigation Steps

Pause `depositTokenForOLAS` when the `Treasury` is paused.

# Informational Findings

## [I-01] `Depository.deposit` function lacks minimum OLAS payout determined by the user <a id="i-01" ></a>

## Details

- `Depository.deposit` function allows users to purchase OLAS at a discount by creating a bond in exchange for a deposit of the user's LP-share (Uniswap v2 LP token).

- The amount of user's bond payout is calculated by `GenericBondCalculator.calculatePayoutOLAS` function based on the deposited LP token amount and the price of it statically recorded in the product when the product is created (static price determined when the owner creates a product) and the idf value of the current epoch (inverse discount factor), then the user can redeem his discounted OLAS after the bond vesting period passes:

  ```javascript
  // @ Depository.deposit calculated payout:
  payout = IGenericBondCalculator(bondCalculator).calculatePayoutOLAS(
    tokenAmount,
    product.priceLP
  );

  // @ GenericBondCalculator.calculatePayoutOLAS calculated amountOLAS:
  amountOLAS = (ITokenomics(tokenomics).getLastIDF() * totalTokenValue) / 1e36;
  ```

- As can be noticed; the only variable that might affect the payout of the user is the idf, where the minimum value of it is 1e18, but the `deposit` function doesn't have a minimum value of payout determined by the user, which might result in the user getting less OLAS payout than he wishes for.

## Proof of Concept

[Depository.deposit function](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L291-293)

```javascript
    function deposit(uint256 productId, uint256 tokenAmount) external
        returns (uint256 payout, uint256 maturity, uint256 bondId)
    {
```

## Recommendation

Update `deposit` function to have a minimum payout parameter and check the calculated payout against it.

## [I-02] `Tokenomics.initializeTokenomics` : some assigned variables values contradict with documentation and Natspec<a id="i-02" ></a>

## Details

1. `rewardUnitFraction` assigned values contradict Natspec:

   ```javascript
   // The initial target is to distribute around 2/3 of incentives reserved to fund owners of the code
   // for components royalties and 1/3 for agents royalties
   tp.unitPoints[0].rewardUnitFraction = 83;
   tp.unitPoints[1].rewardUnitFraction = 17;
   ```

2. Top-up fractions assigned values contradict the [documentation pages 25-26](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/docs/Autonolas_tokenomics_audit.pdf) that says 49,34,17 :
   ```javascript
       // Top-up fractions
           uint256 _maxBondFraction = 50;
           tp.epochPoint.maxBondFraction = uint8(_maxBondFraction);
           tp.unitPoints[0].topUpUnitFraction = 41;
           tp.unitPoints[1].topUpUnitFraction = 9;
   ```

## Proof of Concept

[Tokenomics.initializeTokenomics function/L346-L349](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L346-L349)

```javascript
// The initial target is to distribute around 2/3 of incentives reserved to fund owners of the code
// for components royalties and 1/3 for agents royalties
tp.unitPoints[0].rewardUnitFraction = 83;
tp.unitPoints[1].rewardUnitFraction = 17;
```

[Tokenomics.initializeTokenomics function/L359-L363](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L359-L363)

```javascript
       // Top-up fractions
        uint256 _maxBondFraction = 50;
        tp.epochPoint.maxBondFraction = uint8(_maxBondFraction);
        tp.unitPoints[0].topUpUnitFraction = 41;
        tp.unitPoints[1].topUpUnitFraction = 9;
```

## Recommendation

Update values to match documentation or documentation to match assigned values; whichever is correct.

## [I-03] `FxERC20ChildTunnel.deposit` function: update Natspec to warn users from using it from a multisig wallets<a id="i-03" ></a>

## Details

- `FxERC20ChildTunnel` contract has two deposit functions to bridge childToken from L2 to L1, `deposit` and `depositTo` function, where the first one assumes that the `msg.sender` address is the same across L2 chains, and the second function givse the user the choice to bridge his tokens to a different address on L1 chain.

- So basically if a user uses a multisg wallet that has different addresses on different chains; he must use `depositTo` function in order not to lose his bridge tokens if he uses `deposit` function, but it was noticed that the `deposit` function is missing this warning in its Natspec as there's no indication of the dangers of using this function with a multisig wallet.

## Proof of Concept

[FxERC20ChildTunnel.depositTo function natspecs](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20ChildTunnel.sol#L54C1-L57C62)

```javascript
    /// @dev Deposits tokens on L2 in order to obtain their corresponding bridged version on L1 by a specified address.
    /// @param to Destination address on L1.
    /// @param amount Token amount to be deposited.
    function depositTo(address to, uint256 amount) external {
```

## Recommendation

```diff
    /// @dev Deposits tokens on L2 in order to obtain their corresponding bridged version on L1 by a specified address.
+   /// @notice Shouldn't be used from a multisig wallet to prevent loss of bridged tokens
    /// @param to Destination address on L1.
    /// @param amount Token amount to be deposited.
    function depositTo(address to, uint256 amount) external {
```

## [I-04] Typo in `Depository` contract<a id="i-04" ></a>

## Details

The comment on `tokenomics` variable is typed as [`Tkenomics`](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L90) instead of `Tokenomics`

    ```javascript
    // Tkenomics contract address
    ```

## Recommendation

```diff
-// Tkenomics contract address
+// Tokenomics contract address
```

#

## [I-05] Typo in `Tokenomics` contract<a id="i-05" ></a>

## Details

A comment made in `checkpoint` function when checking the first bit of `tokenomicsParametersUpdated` is mistakenly indicating checking the second bit instead:

    ```javascript
            // Check if the second bit is set to one
            if (tokenomicsParametersUpdated & 0x01 == 0x01) {
    ```

## Recommendation

```diff
- // Check if the second bit is set to one
+ // Check if the first bit is set to one
```

# Recommendation Findings

## [R-01] `Dispenser.claimOwnerIncentives` function: add another parameter to prevent locking owner rewards<a id="r-01" ></a>

## Details

- `Dispenser.claimOwnerIncentives` function allows servivce NFTs owners to claim their rewards, where the OLAS top-ups and ETH rewards are calculated and sent to the owner (`msg.sender`) via `ITreasury(treasury).withdrawToAccount` function.

- But if the owner of the service/component/agent is an un-payable contract that can't receive native tokens; then his rewards will be stuck.

- So it's recommended to add `receiver` address parameter to the `claimOwnerIncentives` function to determine to where these rewards are going to be sent.

## Recommendation

```diff
-  function claimOwnerIncentives(uint256[] memory unitTypes, uint256[] memory unitIds) external
+  function claimOwnerIncentives(uint256[] memory unitTypes, uint256[] memory unitIds, address receiver) external

        returns (uint256 reward, uint256 topUp)
    {
        // Reentrancy guard
        if (_locked > 1) {
            revert ReentrancyGuard();
        }
        _locked = 2;

        // Calculate incentives
        (reward, topUp) = ITokenomics(tokenomics).accountOwnerIncentives(msg.sender, unitTypes, unitIds);

        bool success;
        // Request treasury to transfer funds to msg.sender if reward > 0 or topUp > 0
        if ((reward + topUp) > 0) {
-           success = ITreasury(treasury).withdrawToAccount(msg.sender, reward, topUp);
+           success = ITreasury(treasury).withdrawToAccount(receiver, reward, topUp);
        }

        // Check if the claim is successful and has at least one non-zero incentive.
        if (!success) {
            revert ClaimIncentivesFailed(msg.sender, reward, topUp);
        }

        emit IncentivesClaimed(msg.sender, reward, topUp);

        _locked = 1;
    }
```

## [R-02] `FxGovernorTunnel` contract: consider adding a mechanism to prevent replaying proposals on Polygon<a id="r-02" ></a>

## Details

- As per the documentation; the `TimeLock` contract is going to **broadcast** the successfully executed proposal to Polygon by `FxGovernorTunnel` that's deployed on Polygon, this is done by adding a new proposal that have an encoded call to the method `sendMessageToChild()` of the Polygon [FxRoot contract](https://github.com/0xPolygon/fx-portal/blob/731959279a77b0779f8a1eccdaea710e0babee19/contracts/FxRoot.sol#L29C1-L32C6) deployed on Ethereum:

  ```javascript
      function sendMessageToChild(address _receiver, bytes calldata _data) public override {
          bytes memory data = abi.encode(msg.sender, _receiver, _data);
          stateSender.syncState(fxChild, data);
      }
  ```

- And as per `governance_bridge` documentation :

  > Polygon validators listening for this StateSync event then trigger the onStateReceived() in the Polygon [FxChild contract](https://github.com/0xPolygon/fx-portal/blob/731959279a77b0779f8a1eccdaea710e0babee19/contracts/FxChild.sol#L27C1-L32C6) deployed to Polygon

  ```javascript
      function onStateReceive(uint256 stateId, bytes calldata _data) external override {
          require(msg.sender == address(0x0000000000000000000000000000000000001001), "Invalid sender");
          (address rootMessageSender, address receiver, bytes memory data) = abi.decode(_data, (address, address, bytes));
          emit NewFxMessage(rootMessageSender, receiver, data);
          IFxMessageProcessor(receiver).processMessageFromRoot(stateId, rootMessageSender, data);
      }
  ```

## Recommendation

- As can be noticed; the `FxRoot.sendMessageToChild` and `FxChild.onStateReceive` are going to be executed in **two different transactions** , so to prevent the same message/proposal from being replayed on Polygon (executed more than once) by malicious validators; it's recommended to hash the executed message/proposal and store it in a mapping in the `FxGovernorTunnel` contract.
