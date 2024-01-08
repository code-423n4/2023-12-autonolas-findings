
# Autonolas QA Report

| QA issues | Issues                                                                               | Instances | Severity    |
|-----------|--------------------------------------------------------------------------------------|-----------|-------------|
| [L-01](#l-01---function-name-getcurrentpricelp-and-returned-value-name-pricelp-is-confusing)      | Function name `getCurrentPriceLP` and returned value name `priceLP` is confusing     | 2         | Low         |
| [N-01](#n-01-claimownerincentives-do-not-specify-to-order-and-not-having-repeating-unitids)      | `claimOwnerIncentives` does not specify order and uniqueness of `unitIDs`            | 1         | Non-Critical|
| [N-02](#n-02-no-zero-address-check-in-changedonatorblacklist)      | No zero address check in `changeDonatorBlacklist`                                    | 1         | Non-Critical|
| [R-01](#r-01-make-componentregistry-and-agentregistry-upgradable-in-registriesmanagersol)      | Make `componentRegistry` and `agentRegistry` upgradable in `RegistriesManager.sol`    | 1         | Recommendation|
| [R-02](#r-02---variable-in-tokenbyindex-is-confusing)    | Variable in `tokenByIndex` is confusing                                              | 1         | Recommendation|
| [R-03](#r-03---the-changeincentivefractions-function-do-not-enforce-complete-allocation-of-the-olas-token-incentives)    |The `changeIncentiveFractions` function do not enforce complete allocation of the OLAS token incentives                        | 1         | Recommendation|



## L-01 - Function name `getCurrentPriceLP` and returned value name `priceLP` is confusing

### Context

#### GenericBondCalculator.sol

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/GenericBondCalculator.sol#L70C25-L70C25

#### Depository.sol

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L491C14-L491C31

### Description

In the Generic Bond Calculator, the function name `getCurrentPriceLP` is confusing. The name would suggest that it returns the price of the LP Token, while it only returns the OLAS part of the price. As the pools in this context are assumed to be UniswapV2-like, this represents, in fact, half the value of the pool LP Token.

DAO members or other users responsible for creating bonds in the future should not be confused and use the function's returned value as is in sensitive locations, such as in the creation of new bonding products ([`create`](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L183)). For instance, where the discount would be applied on an already discounted by half price.

Indeed, `priceLP` is an argument to the function used to create bonding products. This price will be used to compute the discount price to apply to mint OLAS tokens in the bond. The `priceLP` that this function (`create(address token, uint256 priceLP, uint256 supply, uint256 vesting)(uint256 productId)`) expects is 2 times the `priceLP` returned by `getCurrentPriceLP(address token)(uint256 priceLP)`.

### Recommendation

A function name `getCurrentHalfPriceLP` that returns a value named `halfPriceLP` would be more explicit. This would ensure that the function is accurately used and would not lead to potential misunderstandings or errors in the future that could lead to a loss of funds for the users depositing in the bonding mechanism.


#### GenericBondCalculator.sol

```diff
-function getCurrentPriceLP(address token) external view returns (uint256 priceLP)
+function getCurrentHalfPriceLP(address token) external view returns (uint256 halfPriceLP)
{
IUniswapV2Pair pair = IUniswapV2Pair(token);
uint256 totalSupply = pair.totalSupply();
if (totalSupply > 0) {
   address token0 = pair.token0();
   address token1 = pair.token1();
   uint256 reserve0;
   uint256 reserve1;
   // requires low gas
   (reserve0, reserve1, ) = pair.getReserves();
   // token0 != olas && token1 != olas, this should never happen
   if (token0 == olas || token1 == olas) {
       // If OLAS is in token0, assign its reserve to reserve1, otherwise the reserve1 is already correct
       if (token0 == olas) {
           reserve1 = reserve0;
       }
       // Calculate the LP price based on reserves and totalSupply ratio multiplied by 1e18
       // Inspired by: https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy#L262
-       priceLP = (reserve1 * 1e18) / totalSupply;
+       halfPriceLP = (reserve1 * 1e18) / totalSupply;
   }
}
}
```
#### Depository.sol

```diff
-function getCurrentPriceLP(address token) external view returns (uint256 priceLP)
+function getCurrentHalfPriceLP(address token) external view returns (uint256 halfPriceLP)
{
IUniswapV2Pair pair = IUniswapV2Pair(token);
uint256 totalSupply = pair.totalSupply();
if (totalSupply > 0) {
   address token0 = pair.token0();
   address token1 = pair.token1();
   uint256 reserve0;
   uint256 reserve1;
   // requires low gas
   (reserve0, reserve1, ) = pair.getReserves();
   // token0 != olas && token1 != olas, this should never happen
   if (token0 == olas || token1 == olas) {
       // If OLAS is in token0, assign its reserve to reserve1, otherwise the reserve1 is already correct
       if (token0 == olas) {
           reserve1 = reserve0;
       }
       // Calculate the LP price based on reserves and totalSupply ratio multiplied by 1e18
       // Inspired by: https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy#L262
-       priceLP = (reserve1 * 1e18) / totalSupply;
+       halfPriceLP = (reserve1 * 1e18) / totalSupply;
   }
}
}
```

## N-01 claimOwnerIncentives do not specify to order and not having repeating unitIDs 

### Context

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Dispenser.sol#L89

### Description

The function `claimOwnerIncentives` docstring does not specify that the `unitIDs` must be ordered and not repeat. The `claimOwnerIncentives` function will fail if the `unitIDs` are not ordered and not unique as it is a requirement of the function `accountOwnerIncentives` of the Tokenomics.sol contract that is called passing the `unitTypes` and `unitIds` argument as is during `claimOwnerIncentives` execution. 

As `claimOwnerIncentives` is the only function that can be called externally for user to claim incentives, this is the function that can be integrated in the different apps. The code must be explicitely clear that the `unitIDs` must be ordered and unique to avoid any confusion and potential loss of gas for users.

### Recommendation

Specify in the docstring that the `unitIDs` must be ordered and unique.

``` diff
/// @dev Claims incentives for the owner of components / agents.
/// @notice `msg.sender` must be the owner of components / agents they are passing, otherwise the function will revert.
/// @notice If not all `unitIds` belonging to `msg.sender` were provided, they will be untouched and keep accumulating.
+ /// @notice Component and agent Ids must be provided in the ascending order and must not repeat.
/// @param unitTypes Set of unit types (component / agent).
/// @param unitIds Set of corresponding unit Ids where account is the owner.
/// @return reward Reward amount in ETH.
/// @return topUp Top-up amount in OLAS.
function claimOwnerIncentives(uint256[] memory unitTypes, uint256[] memory unitIds) external
  returns (uint256 reward, uint256 topUp)
{
 ... // claimOwnerIncentives code 
}
```

## N-02 No zero address check in changeDonatorBlacklist

### Context 

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L474

### Description

The setter for the donator blacklist contract address does not check for the zero address. This could lead to a situation where the donator blacklist contract address is set to the zero address and the donator blacklist is disabled. This could lead to a situation where blacklisted addresses are no longer prevented from being donator.

### Recommendation

Check for the zero address in the setter for the donator blacklist contract address.

```diff
function changeDonatorBlacklist(address _donatorBlacklist) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
+     if (_agentRegistry != address(0)) {
        donatorBlacklist = _donatorBlacklist;
        emit DonatorBlacklistUpdated(_donatorBlacklist);
+     }
    }
```


## R-01 Make componentRegistry and agentRegistry upgradable in RegistriesManager.sol

### Context

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol

### Description

Currently all users have to pass through the Registries Manager to interact with the component and agent registries. And the RegistriesManager cannot update the componentRegistry and agentRegistry addresses. This means that if the DAO decide to change the componentRegistry and agentRegistry addresses, they will need to redeploy all three contracts `ComponentRegistry`, `AgentRegistry` and `RegistriesManager`. All the integrations will have to update their code to use the new addresses. And users of the different app that are not active in the DAO won't necessarly be notified of the change and will continue to use the old addresses or use the new address through the app without knowing that the address have changed.

### Recommandation

Make componentRegistry and agentRegistry upgradable in RegistriesManager.sol with the spcific events and setter functions to notify the user of the change:

- `event ComponentRegistryUpdated(address indexed componentRegistry)`
- `event AgentRegistryUpdated(address indexed agentRegistry)`
- `setComponentRegistry(address _componentRegistry) external`
- `setAgentRegistry(address _agentRegistry) external`
  

## R-02 - Variable in tokenByIndex is confusing

### Context

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L97


### Description

In the registries part of the codebase, the `tokenByIndex` function is used to get the unit ID from a provided index number ranging from `0` to `1`. The variable name `id` is confusing with respect to the docstring and the context of the function call. A variable name `index` would be more explicit.

### Recommendation

Rename the `id` variable to `index` to be more explicit.

```diff
    /// @dev Gets the valid unit Id from the provided index.
    /// @notice Unit counter starts from 1.
-     /// @param id Unit counter.
+     /// @param index Unit counter.
    /// @return unitId Unit Id.
-   function tokenByIndex(uint256 id) external view virtual returns (uint256 unitId) {
+   function tokenByIndex(uint256 index) external view virtual returns (uint256 unitId) {
-        unitId = id + 1;
+        unitId = index + 1;
        if (unitId > totalSupply) {
            revert Overflow(unitId, totalSupply);
        }
    }
```


## R-03 - The `changeIncentiveFractions` function do not enforce complete allocation of the OLAS token incentives

### Context

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L562C1-L562C1

### Description

In the `changeIncentiveFractions` function, the sum of the fractions is not checked to be equal to 100% as the docstring would suggest. In the first case for the ETH rewards we check if the allocation is above 100% because down in the code we use the remaining and specifically allocate it to the DAO treasury:

```javascript
// Check that the sum of fractions is 100%
if (_rewardComponentFraction + _rewardAgentFraction > 100) {
revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);
}
// ...
tp.epochPoint.rewardTreasuryFraction = uint8(100 - _rewardComponentFraction - _rewardAgentFraction);
```

In the topup case we check that the sum of fractions is below 100% but then the remaining is unallocated. This is not consistent with the first case and the docstring that the sum is 100% suggesting everything should be allocated. 

        // Same check for top-up fractions
        if (_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction > 100) {
            revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);
        }

### Recommendation

- If this is the intended behavior as a way to regulate the OLAS inflation rate, it could be specifically stored in a variable (e.g., `_unallocatedOLASRewards`) and clearly documented in the code. Additionally, an equality check should be implemented instead:

```javascript
if (_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction + _unallocatedOLASRewards > 100) {
            revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);
}
```

- Alternatively, these funds could by default be allocated to the DAO treasury, as in the first scenario. The treasury could mint these tokens and utilize them in bounty programs or other incentives not directly related to the protocol's tokenomics.

- Another option is to allocate these funds to the `maxboundfraction`, to be used by default in the bonding mechanism. This approach allows the inflation to be controlled by managing the supply allocated to different products and by monitoring the `effectiveBond` (the OLAS leftover from expired bonding products). However, with this method, the allocation becomes explicit and more transparent.