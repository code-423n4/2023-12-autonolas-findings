# [1] Lack of `address(0)` check in `mint()` function in `BridgedERC20.sol`

[File: BridgedERC20.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/BridgedERC20.sol#L48)
```
function mint(address account, uint256 amount) external {
        // Only the contract owner is allowed to mint
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        
        _mint(account, amount);
    }
```

Protocol uses Solmate's ERC-20 implementation, which - in comparison to OpenZeppelin's implementation's, does not verify if `account` is `address(0)`.

OZ's implementation: 

```
 function _mint(address account, uint256 value) internal {
        if (account == address(0)) {
            revert ERC20InvalidReceiver(address(0));
        }
        _update(address(0), account, value);
    }
```

Solmate's implementation:

```
 function _mint(address to, uint256 amount) internal virtual {
        totalSupply += amount;

        // Cannot overflow because the sum of all user
        // balances can't exceed the max uint256 value.
        unchecked {
            balanceOf[to] += amount;
        }

        emit Transfer(address(0), to, amount);
    }
```

This implies, that we should implement additional check in function `mint()`, which verifies if `account` is not `address(0)`. Otherwise, it would be possible to mint to `address(0)`.


# [2] Events should emit both old and new value whenever critical parameter is being changed

**This instance was  missed in the bot-report**
```
[N-42] Events that mark critical parameter changes should contain both the old and the new value
There are 48 instance(s) of this issue:
```

The bot-report contains 48 instances of events which does not mark both the old and the new value. However, it misses one event: `TokenomicsParametersUpdateRequested`, thus this instance is being reported in our report:

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L551)
```
 emit TokenomicsParametersUpdateRequested(epochCounter + 1, _devsPerCapital, _codePerDev, _epsilonRate, _epochLen,
            _veOLASThreshold);
```

When an important state variable is being updated, it's a good practice to emit an event which will inform about both old and new value.

Function `changeTokenomicsParameters()` updates:

```
    // @param _devsPerCapital Number of valuable devs can be paid per units of capital per epoch.
    /// @param _codePerDev Number of units of useful code that can be built by a developer during one epoch.
    /// @param _epsilonRate Epsilon rate that contributes to the interest rate value.
    /// @param _epochLen New epoch length.
```
However, `TokenomicsParametersUpdateRequested()` event emits only the new (updated) values, instead of both old and new ones. 

# [3] Functions which update the value do not verify if the value was really changed

**Issues reported in this report were missed by the bot-report**
```
[N-50] Setters should prevent re-setting of the same value
*There are 31 instance(s) of this issue:*
```

The bot-report describes 31 instances of this issues. However, some of the functions were missed. The current report lists all the instances missed by the bot-report.


Whenever we update/set a new value of some state variable, it's a good practice to make sure that, that value is being indeed updated.
E.g., in function `changeMinAcceptedETH()` from `Treasury.sol`, even when the `minAcceptedETH` won't be changed, function will still emit `MinAcceptedETHUpdated()` event - which might be misleading to the end-user.
E.g. let's assume that `minAcceptedETH = 123`. Calling `changeMinAcceptedETH(123)` won't update/set any new `minAcceptedETH`, but `MinAcceptedETHUpdated()` will still be emitted.

Our recommendation is to add additional `require` check, to verify, that provided values are indeed different than the current one:

```
require(minAcceptedETH != _minAcceptedETH, "Nothing to update!");
```

This issue was identified in the multiple of contracts. Below, we present a list of affected functions which were not included in the bot-report:


* `Tokenomics.sol`
functions: `changeTokenomicsImplementation()`, `changeTokenomicsParameters()`

* `Treasury.sol`
function: `changeMinAcceptedETH()`

* `DonatorBlacklist.sol`
function: `setDonatorsStatuses()`




# [4] Lack of `address(0)` check in `veOLAS.sol` `constructor()`.

**This instance was missed in the bot-report**
```
[L-05] Missing checks in constructor

File: governance/contracts/veOLAS.sol

//@audit _name, _symbol,  are not checked
132:     constructor(address _token, string memory _name, string memory _symbol)
133:     {
```
The bot-report suggests, that only `_name` and `_symbol` are not checked. However, `constructor()` in `veOLAS.sol` does not check `_token` parameter. This parameter was not included in the bot-report, thus it's reported here.

[File: veOLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L134)
```
token = _token;
```

`constructor()` does not verify if `_token` parameter, passed by the user - is not `address(0)`. Our recommendation is to add additional check, which would verify that:

```
if (_token == address(0)) {
    revert ZeroAddress();
}
```

# [5] Lack of length control in `updateHash()` in `UnitRegistry.sol`

[File: UnitRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L143)
```
 mapUnitIdHashes[unitId].push(unitHash);
```

Updating the hash pushes `unitHash` into `mapUnitIdHashes[unitId]`. Calling `updateHash()` multiple of times makes an `mapUnitIdHashes[unitId]` array bigger and bigger (every time, new element is being pushed).

When this array becomes too big, it won't be possible to return it by calling `getUpatedHashes()`:

[File: UnitRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L171)
```
function getUpdatedHashes(uint256 unitId) external view virtual
        returns (uint256 numHashes, bytes32[] memory unitHashes)
    {
        unitHashes = mapUnitIdHashes[unitId];
        return (unitHashes.length, unitHashes);
    }
```

When `mapUnitIdHashes[unitId]` becomes too big - `getUpdatedHashes()` will revert with out of gas error.

A simple Remix-IDE test has been prepared to simulate this behavior:

```
    mapping(uint256 => bytes32[]) public mapUnitIdHashes;
      
      function getUpdatedHashes(uint256 unitId) public view 
        returns (uint256 numHashes, bytes32[] memory unitHashes) {
        unitHashes = mapUnitIdHashes[unitId];
        return (unitHashes.length, unitHashes);
    }

    function updateHash(uint256 unitId, bytes32 unitHash) public 
        returns (bool success) {
        mapUnitIdHashes[unitId].push(unitHash);
        success = true;    
    }

    function testSingle() external  {   
        updateHash(1, bytes32("test"));
        uint gas = gasleft();
        getUpdatedHashes(1);
        console.log(gas - gasleft());
    }

       function testMultiple() external  {
        for (uint i; i < 1000; i++) updateHash(1, bytes32("test"));
        uint gas = gasleft();
        getUpdatedHashes(1);
        console.log(gas - gasleft());
    }
```

Function `testMultiple()` uses 155500 gas, while `testSingle()` uses 681 gas. This proves, that when `mapUnitIdHashes[unitId]` increases in its size, `getUpdatedHashes()` may finally revert with out of gas error.

Our recommendation is to implement a size-check on the `mapUnitIdHashes[unitId]`. It should not be possible to push new elements there, if the size of `mapUnitIdHashes[unitId]` is too big.


# [6] Looping over unbounded array may result in DoS 

Function `getProducts()` from `Depository.sol` executes below loop:

[File: Depository.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L396)
```
 function getProducts(bool active) external view returns (uint256[] memory productIds) {
        // Calculate the number of existing products
        uint256 numProducts = productCounter;
[...]
        for (uint256 i = 0; i < numProducts; ++i) {
[...]
```

It basically iterates from 0 to `productCounter`. The `productCounter` is defined in function `create()`:

[File: Depository.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L231)
```
        productId = productCounter;
        mapBondProducts[productId] = Product(uint160(priceLP), uint32(vesting), token, uint96(supply));
        // Even if we create a bond product every second, 2^32 - 1 is enough for the next 136 years
        productCounter = uint32(productId + 1);
```

Every time function `create()` is called, `productCounter` is being increased. This implies, that `getProducts()` iterates over every previously created product. This might constitute an issue, when number of products is big. When there are too many products created, `productCounter` will point to very big number. Since `getProducts()` iterates from `0` to `productCounter` - the number of iterations may be too big and function might revert with out of gas error. 

We've estimated this issue as Low, because although the risk of DoS (due to out of gas error) is feasible - the number of products can be increased only by calling `create()` function. This function can be called only by the owner:

```
if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
```

This implies, that the owner should be aware of the products' number limitations (when too many products will be created, function `getProducts()` will revert because of looping over too big array). Nonetheless, we do recommend to implement additional check in function `create()` - which won't allow to create too many products.

# [7] Use constants for previously defined values in `TokenomicsConstants.sol`

[File: TokenomicsConstants.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L51)
```
            // After that the inflation is 2% per year as defined by the OLAS contract
            uint256 maxMintCapFraction = 2;
```

[File: TokenomicsConstants.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L89)
```
            // After that the inflation is 2% per year as defined by the OLAS contract
            uint256 maxMintCapFraction = 2;
```

Since  `maxMintCapFraction` is used only once and is previously defined: `the inflation is 2% per year as defined by the OLAS contract`, it should be declared as a constant variable.


# [8] Divide `changeManagers()` from `Treasury.sol` into 3 separate functions

[File: Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L163-L176)
```
if (_tokenomics != address(0)) {
            tokenomics = _tokenomics;
            emit TokenomicsUpdated(_tokenomics);
        }
        // Change Depository contract address
        if (_depository != address(0)) {
            depository = _depository;
            emit DepositoryUpdated(_depository);
        }
        // Change Dispenser contract address
        if (_dispenser != address(0)) {
            dispenser = _dispenser;
            emit DispenserUpdated(_dispenser);
        }
```

The same issue occurs in `changeManagers()` in `Dispenser.sol`.
The same issue occurs in `changeManagers()` in `Depository.sol`.
The same issue occurs in `changeManagers()` in `Tokenomics.sol`.


Function `changeManagers()` sets Tokenomics, Depository and Dispenser addresses. It sets the particular address depending on the `address(0)` value. E.g. if we want to set only Tokenomics - we need to set other parameters (`_depository` and `_dispenser`) to `address(0)`. This behavior may be misleading to the end-user, moreover he/she can incorrectly pass parameters. Much better idea would be to create 3 separate functions, responsible for updating only specific address: `changeTokenomicsManagers()`, `changeDepositoryManagers()`, `changeDispenserManagers()`.


# [9] Change the order of returned parameters in `UnitRegistry.sol`

[File: UnitRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L160)
```
function getDependencies(uint256 unitId) external view virtual
        returns (uint256 numDependencies, uint32[] memory dependencies)
  [...]
    function getUpdatedHashes(uint256 unitId) external view virtual
        returns (uint256 numHashes, bytes32[] memory unitHashes)
 [...]
    function getLocalSubComponents(uint256 unitId) external view
        returns (uint32[] memory subComponentIds, uint256 numSubComponents)
[...]
```

Function `getDependencies()` returns: number of units in the dependency list and the list of unit dependencies.
Function `getUpdatedHashes()` returns: numbers of hashes and the list of updated unit hashes.

Please notice, that we firstly return the number of elements (units or hashes) and then the list (of unit dependencies or updated unit hashes).
However, this order is different in `getLocalSubComponents()`. It firstly returns the list of subcomponent IDs, and then, the number of subcomponents. Our recommendation is to stick to the previously defined schema and change the returned parameters in `getLocalSUbComponents()`. It should - similarly to `getUpdatedHashes()` and `getDependencies()`, firstly return the number of subcomponents and then, the list of subcomponents Ids:

```
function getLocalSubComponents(uint256 unitId) external view
        returns (uint256 numSubComponents, uint32[] memory subComponentIds)
```

# [10] Standarize how paused contract is represented

`GuardCM.sol`, `Treasury.sol` implements `uint8 public paused` variable which defines if contract is paused or not. This variable is either `1` or `2`. 

However, when we take a look at `GenericManager.sol` - we can notice, that variable `paused` is represented as `bool`: `bool public paused` (thus it can be either `true` or `false`). To improve the code readability, it's a good practice to represent a paused state in a same way across every contract.

[File: GenericManager.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericManager.sol#L16)
```
bool public paused;
```

Our recommendation is to modify `GenericManager.sol`, and implement `paused` as `uint8`. This will also require to change `RegistriesManager.sol`, which uses that variable to decide if contract is paused or not.

# [11] Incorrect grammar

[File: IErrorsTokenomics](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/interfaces/IErrorsTokenomics.sol#L20-L21)
```
/// @param numValues1 Number of values in a first array.
/// @param numValues2 Number of values in a second array.
```

`a first array` should be changed to `the first array`.
`a second array` should be changed to `the second array`.

[File: IErrorsRegistries.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/interfaces/IErrorsRegistries.sol#L27-L28)
```
/// @param numValues1 Number of values in a first array.
/// @param numValues2 Numberf of values in a second array.
```

`a first array` should be changed to `the first array`.
`a second array` should be changed to `the second array`.


# [12] Some `IERC20` comments should be more descriptive

* `transferFrom()`

[File: IERC20.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/interfaces/IERC20.sol#L33)
```
/// @dev Transfers the token amount that was previously approved up until the maximum allowance.
```

`until the maximum allowance` is very vague description. It'll be much clearer to include: what, from who, to whom and how much - tokens can be transferred. Our recommendation is to copy-paste OpenZeppelin's NatSpec for `transferFrom()`: 

```
Moves an `amount` amount of tokens from `from` to `to` using the allowance mechanism. `amount` is then deducted from the caller's allowance.
```

* `burn()`

[File: IERC20.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/interfaces/IERC20.sol#L45)
```
/// @dev Burns tokens.
```

Misses the information that the caller's tokens will be burned. Our recommendation is to copy-paste OpenZeppelin's NatSpec: `Destroys an amount of tokens from the caller.`


# [13] Typos

[File: UnitRegistry.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L235)
```
// Either get a component that has a higher id than the last one ore reach the end of the processed Ids
```

`ore` should be changed to `or`.

[File: GnosisSafeMultisig.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeMultisig.sol#L42)
```
/// @dev Parses (unpacks) the data to gnosis safe specific parameters.
    /// @notice If the provided data is not empty, its length must be at least 144 bytes to be parsed correctly.
    /// @param data Packed data related to the creation of a gnosis safe multisig.
```

[File: GnosisSafeMultisig.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeMultisig.sol#L86)
```
/// @dev Creates a gnosis safe multisig.
```

`gnosis safe` should be changed to `Gnosis Safe`


# [14] Constants should be upper-cased

Constant variables should be UPPERCASED:

[File: OLAS.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L22)
```
 uint256 public constant oneYear = 1 days * 365;
    // Total supply cap for the first ten years (one billion OLAS tokens)
    uint256 public constant tenYearSupplyCap = 1_000_000_000e18;
    // Maximum annual inflation after first ten years
    uint256 public constant maxMintCapFraction = 2;
```

# [15] Use constants for Gnosis/Polygon chain IDs

[File: GuardCM.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L259)
```
256: if (chainId == 100 || chainId == 10200) {
[...]
304:  if (chainId == 137 || chainId == 80001) {
```


# [16] Incorrect comment in `changeIncentiveFractions()` in `Tokenomics.sol`

[File: Tokenomics.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L575)
```
 // Check that the sum of fractions is 100%
        if (_rewardComponentFraction + _rewardAgentFraction > 100) {
            revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);
        }
[...]

 // Rewards are always distributed in full: the leftovers will be allocated to treasury
        tp.epochPoint.rewardTreasuryFraction = uint8(100 - _rewardComponentFraction - _rewardAgentFraction);
```


Please notice, that in line 592, the leftovers are allocated to the treasury: ` uint8(100 - _rewardComponentFraction - _rewardAgentFraction)`.
This means, that `_rewardComponentFraction` and `_rewardAgentFractionwe` does not need to sum up to 100. If `_rewardComponentFraction + _rewardAgentFraction < 100`, the leftovers are allocated to the treasure. Literally, we're checking if the sum of fractions does not exceed 100% (while comment states that we are checking that the sum of fractions is 100%). The above comment should be changed to: `Check that the sum of fractions does not exceed 100%`.

# [17] Divide `changeRegistries()` from `Tokenomics.sol` into 3 separate functions

Function `changeRegistries()` updates `componentRegistry`, `agentRegistry`, `serviceRegistry`. It sets the particular address depending on the `address(0)` value. E.g., if we want to update only `componentRegistry` address, we need to set other parameters (`_agentRegistry`, `_componentRegistry`) to `address(0)`. This behavior may be very misleading to the end user. It's better to divide this function into 3 separate functions: `changeConponentRegistry()`, `changeAgentRegistry()`, `changeServiceRegistry()`.

# [18] Use `uint256` instead of `uint`

While `uint` is a shorter form of `uint256`, using `uint256` in the code-base improves the code readability.
While most of the code use `uint256`, we notice that one file: `GuardCM.sol` uses `uint` instead of `uint256`:

[File: GuardCM.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L360)
```
for (uint i = 0; i < targets.length; ++i) {
```

# [19] `Treasury.sol` misses some comments which informs if function can be called when contract is paused

`Treasury.sol` can be paused/unpaused. Most of the functions describe in their NatSpec if function can be called on a paused/unpaused state.

E.g. 

[File: Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L209)
```
    /// #if_succeeds {:msg "any paused"} paused == 1 || paused == 2;
```

This comment suggests, that `depositTokenForOLAS()` can be called either when contract is paused or unpaused.
Multiple of `Treasury.sol` functions use comments like that to help the code-readers to quickly verify if a specific function may be called when contract is paused/unpased. This is a list of such functions:

* `depositServiceDonationsETH()`
* `depositTokenForOLAS()`
* `withdraw()`
* `withdrawToAccount()`
* `rebalanceTreasury()`

However, some other functions miss this type of comment. Our recommendation is to stick to the NatSpec-style and add missing information about calling those functions in a paused/unpaused state:

* `changeOwner()`
* `changeManagers()`
* `changeMinAcceptedETH()`
* `drainServiceSlashedFunds()`
* `enableToken()`
* `disableToken()`

E.g. `enableToken()` should be called either on paused or unpaused state. Moreover, it does not implement any check which verifies if contract is paused/unpaused. This suggests, that above comment should be added to its NatSpec:

```
    /// @dev Enables an LP token to be bonded for OLAS.
    /// @param token Token address.
    // #if_succeeds {:msg "any paused"} paused == 1 || paused == 2;
    function enableToken(address token) external {
```

# [20] Invalid Markdown syntax in README.md
```
| [tokenomics/contracts/Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol])
```

The link creation syntax is invalid. File points to `Treasury.sol]` instead of `Treasury.sol`. Above Markdown code should be changed to:

```
| [tokenomics/contracts/Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol)
```

