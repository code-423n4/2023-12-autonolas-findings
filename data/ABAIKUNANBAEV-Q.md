## Findings Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-the-owner-uses-hard-coded-price-lp-instead-of-fetching) | The owner uses hard-coded `priceLP` instead of fetching it | Low |
| [L-02](#l-02-users-can-front-run-the-owner-when-closing-the-product) | Users can front-run the owner when closing the product| Low |
| [L-03](#l-02-blacklisted-donators-can-easily-bypass-the-blacklist) | Blacklisted donators can easily bypass the blacklist | Low |



## [L-01] The owner uses hard-coded `priceLP` instead of fetching it

When creating a product for bonding, the owner provides the price as `priceLP` param in the arguments of the function instead of fetching it from `getCurrentPriceLP()` which may lead to incorrect price being fetched:

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L183

```
 function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId)

```

`getCurrentPriceLP()`:

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L491-493

```
  function getCurrentPriceLP(address token) external view returns (uint256 priceLP) {
        return IGenericBondCalculator(bondCalculator).getCurrentPriceLP(token);
    }

```

### Recommendaton

Use the `getCurrentPriceLP()` function to determine the `priceLP` when creating a product.


## [L-02] Users can front-run the owner when closing the product

When the owner tries to close the product, it checks whether there is still supply left to refund from the bond program. However, at the same time the user may deposit some tokens for OLAS and make supply zero and the owner tx will be reverted:

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L260-262

```

 if (supply > 0) {
                // Refund unused OLAS supply from the product if it was not used by the product completely
                ITokenomics(tokenomics).refundFromBondProgram(supply);

```

The supply will be decreased by a payout value:

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L328

```
  supply -= payout;

```


### Recommendaton

Execute the tx via private mempool.


## [L-03] Blacklisted donators can easily bypass the blacklist

The protocol has a special contract to blacklist undesired donators - `DonatorBlacklist`. However, the function `setDonatorStatuses()` doesn't really protect from such donations as this can be simply bypassed by creating new multiple accounts and donate from them:

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L56

```
 function setDonatorsStatuses(address[] memory accounts, bool[] memory statuses) external returns (bool success)
```

### Recommendation

Consider implementing different functionality to prevent undesired depositors from depositing.


