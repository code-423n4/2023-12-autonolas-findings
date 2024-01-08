## Gas Optimizations

### I have written a report on gas optimization, which consists of two parts:

#### Part 1: Identification of Instances Caused by Automated Bot Issue
Identification of instances of the issue caused by automated bots: In this part, I investigated the occurrence of the issue and identified instances where it was caused by automated bots. While some instances were successfully identified, there were also cases where the issue remained undetected.
#### summary:
- [[G-01]Optimization for Gas Efficiency through State Variable Caching](#g-01optimization-for-gas-efficiency-through-state-variable-caching)
- [[G-02]Gas Efficiency Improvement via State Variable Caching in Mint Function](#g-02gas-efficiency-improvement-via-state-variable-caching-in-mint-function)
- [[G-03]Gas Optimization through Consolidated Ownership Check in GenericManager Contract](#g-03gas-optimization-through-consolidated-ownership-check-in-genericmanager-contract)
- [[G-04]Gas Efficiency Enhancement via Ownership Modifier in GenericRegistry Contract](#g-04gas-efficiency-enhancement-via-ownership-modifier-in-genericregistry-contract)
- [[G-05]Optimizing State Variable Access in Dispenser Contract](#g-05optimizing-state-variable-access-in-dispenser-contract)
- [[G-06]Optimizing State Variable Access and Reducing Gas Costs in Depository Contract](#g-06optimizing-state-variable-access-and-reducing-gas-costs-in-depository-contract)
- [[G-07]Gas Efficiency Improvement by Consolidating State Variable Access in Tokenomics Contract](#g-07gas-efficiency-improvement-by-consolidating-state-variable-access-in-tokenomics-contract)
- [[G-08]Optimizing Gas Consumption by Reducing State Variable Access in Tokenomics Contract](#g-08optimizing-gas-consumption-by-reducing-state-variable-access-in-tokenomics-contract)
- [[G-09]Enhancing Gas Efficiency in BridgedERC20 Contract](#g-09enhancing-gas-efficiency-in-bridgederc20-contract)
- [[G-10]Reducing Gas Consumption in Loop Iteration](#g-10reducing-gas-consumption-in-loop-iteration)
- [[G-11] Use calldata instead of memory for function arguments that do not get mutated](#g-11-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)
- [[G-12] Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate](#g-12-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate)
- [[G-13] Avoid transferring amounts of zero in order to save gas](#g-13-avoid-transferring-amounts-of-zero-in-order-to-save-gas)
- [[G-14] Use assembly in place of abi.decode to extract calldata values more efficiently](#g-14-use-assembly-in-place-of-abidecode-to-extract-calldata-values-more-efficiently)



#### Part 2: Identification of Issues and Associated Instances
#### summary:

- [[G-15] Optimizing External Call Cache Within Loop in Depository.close() function](#g-15-optimizing-external-call-cache-within-loop-in-depositoryclose-function)
- [[G-16] Optimizing External Calls in Loop for Gas Efficiency in Tokenomics._trackServiceDonations() function](#g-16-optimizing-external-calls-in-loop-for-gas-efficiency-in-tokenomics_trackservicedonations-function)
- [[G-17]Optimizing External Call Caching for Gas Efficiency in Tokenomics.trackServiceDonations() function](#g-17optimizing-external-call-caching-for-gas-efficiency-in-tokenomicstrackservicedonations-function)
- [[G-18] Optimize External Calls with Assembly for Memory Efficiency](#g-18-optimize-external-calls-with-assembly-for-memory-efficiency)
- [[G-19] Refactor External/Internal Functions to Avoid Redundant External Calls in GnosisSafeSameAddressMultisig.create() function](#g-19-refactor-externalinternal-functions-to-avoid-redundant-external-calls-in-gnosissafesameaddressmultisigcreate-function)
- [[G-20] Refactor External/Internal Functions to Avoid Redundant External Calls in Depository.create() function](#g-20-refactor-externalinternal-functions-to-avoid-redundant-external-calls-in-depositorycreate-function)
- [[G-21] Refactor External/Internal Functions to Eliminate Redundant External Calls in Treasury.depositTokenForOLAS() function](#g-21-refactor-externalinternal-functions-to-eliminate-redundant-external-calls-in-treasurydeposittokenforolas-function)
- [[G-22]Minimize Redundant External Calls in `drainServiceSlashedFunds()` Function](#g-22minimize-redundant-external-calls-in-drainserviceslashedfunds-function)
- [[G-23] Gas Optimization by Changing Storage Variable Visibility in veOLAS Contract](#g-23-gas-optimization-by-changing-storage-variable-visibility-in-veolas-contract)
- [[G-24] Utilize Constants Instead of `type(uintx).max`](#g-24-utilize-constants-instead-of-typeuintxmax)
- [[G-25] Admin functions can be payable](#g-25-admin-functions-can-be-payable)
- [[G-26] bytes constants are more eficient than string constans](#g-26-bytes-constants-are-more-eficient-than-string-constans)
- [[G-27] Use selfbalance() instead of address(this).balance](#g-27-use-selfbalance-instead-of-addressthisbalance)
- [[G-28] Pre-increment and pre-decrement are cheaper than +1 ,-1](#g-28-pre-increment-and-pre-decrement-are-cheaper-than-1--1)
- [[G-29] Ternary operation is cheaper than if-else statement](#g-29-ternary-operation-is-cheaper-than-if-else-statement)
- [[G-30] Access mappings directly rather than using accessor functions](#g-30-access-mappings-directly-rather-than-using-accessor-functions)

- [[G‑31] Unlimited gas consumption risk due to external call recipients](#g-31-unlimited-gas-consumption-risk-due-to-external-call-recipients)
- [[G-32] Move storage pointer to top of function to avoid offset calculation](#g-32-move-storage-pointer-to-top-of-function-to-avoid-offset-calculation)
- [[G-33] Initialize variables with no default value](#g-33-initialize-variables-with-no-default-value)

## [G-01]Optimization for Gas Efficiency through State Variable Caching
#### Note: These are instances the automated report missed
**Description:** The code base underwent an optimization process to enhance gas efficiency by reducing redundant state variable reads from storage within specific functions.

**Original code**
```solidity
File:  governance/contracts/OLAS.sol
43  function changeOwner(address newOwner) external {
        if (msg.sender != owner) {
            revert ManagerOnly(msg.sender, owner);
        }

        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Changes the minter address.
    /// @param newMinter Address of a new minter.
    function changeMinter(address newMinter) external {
        if (msg.sender != owner) {
            revert ManagerOnly(msg.sender, owner);
        }

        if (newMinter == address(0)) {
            revert ZeroAddress();
        }

        minter = newMinter;
        emit MinterUpdated(newMinter);
69  }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L43-L69


**optimized code**
```solidity
// Define a modifier to check if the sender is the owner
modifier onlyOwner() {
    if (msg.sender != owner) {
            revert ManagerOnly(msg.sender, owner);
    }
    _;
}

// Apply the modifier to both functions
function changeOwner(address newOwner) external onlyOwner {
    // ... rest of the function remains unchanged
}

function changeMinter(address newMinter) external onlyOwner {
    // ... rest of the function remains unchanged
}
```

**Mitigation:**
- **State Variable Caching:** The modifier `onlyOwner` was introduced, consolidating the logic to check if the message sender matches the owner into a single reusable modifier. This modifier directly uses the locally cached `owner` state variable stored in the contract's storage, eliminating the need for repeated storage reads in the `changeOwner` and `changeMinter` functions.
  
- **Gas Savings:** This modification significantly reduces gas consumption by avoiding repeated retrievals of the `owner` state variable from storage. By utilizing a stack variable within the modifier, gas costs associated with unnecessary storage reads during function execution have been mitigated.







## [G-02]Gas Efficiency Improvement via State Variable Caching in Mint Function
#### Note: These are instances the automated report missed
**Description:** The smart contract's `mint` function underwent an optimization process aimed at enhancing gas efficiency by minimizing redundant state variable reads from storage.

**Original code**
```solidity
function mint(address account, uint256 amount) external {
    address minterAddress = minter; // Cache state variable in a local variable

    // Access control using the cached variable
    if (msg.sender != minterAddress) {
        revert ManagerOnly(msg.sender, minterAddress);
    }

    // Check the inflation schedule and mint
    if (inflationControl(amount)) {
        _mint(account, amount);
    }
}
```

**optimized code**
```solidity
File: governance/contracts/OLAS.sol
    function mint(address account, uint256 amount) external {
        // Access control
        if (msg.sender != minter) {
            revert ManagerOnly(msg.sender, minter);
        }

        // Check the inflation schedule and mint
        if (inflationControl(amount)) {
            _mint(account, amount);
        }
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L75-L85

**Mitigation:**
- **State Variable Caching:** The `minter` state variable was cached into a local variable within the `mint` function. By storing the `minter` value locally, subsequent checks for the sender's authority against the `minter` address now utilize this cached variable, avoiding repeated retrievals from storage.
  
- **Gas Savings:** This modification significantly reduces gas consumption by eliminating redundant storage reads for the `minter` state variable within the function execution. Utilizing a stack variable enhances gas efficiency, reducing overall transaction costs associated with contract execution.



## [G-03]Gas Optimization through Consolidated Ownership Check in GenericManager Contract
#### Note: These are instances the automated report missed
**Description:** The codebase underwent a gas optimization process to streamline ownership verification by consolidating the logic into a reusable modifier within the `GenericManager` abstract contract. This modification aims to improve gas efficiency by avoiding redundant state variable reads from storage.

**Original code**
```solidity
File:  registries/contracts/GenericManager.sol
abstract contract GenericManager is IErrorsRegistries {
    event OwnerUpdated(address indexed owner);
    event Pause(address indexed owner);
    event Unpause(address indexed owner);

    // Owner address
    address public owner;
    // Pause switch
    bool public paused;

    /// @dev Changes the owner address.
    /// @param newOwner Address of a new owner.
    function changeOwner(address newOwner) external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Pauses the contract.
    function pause() external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        paused = true;
        emit Pause(msg.sender);
    }

    /// @dev Unpauses the contract.
    function unpause() external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        paused = false;
        emit Unpause(msg.sender);
    }
}
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L8-L56

**optimized code**

```solidity
File:  registries/contracts/GenericManager.sol
abstract contract GenericManager is IErrorsRegistries {
    event OwnerUpdated(address indexed owner);
    event Pause(address indexed owner);
    event Unpause(address indexed owner);

    // Owner address
    address public owner;
    // Pause switch
    bool public paused;
    
    // Modifier to check for ownership
    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        _;
    }


    /// @dev Changes the owner address.
    /// @param newOwner Address of a new owner.
    function changeOwner(address newOwner) external virtual onlyOwner {
        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Pauses the contract.
    function pause() external virtual onlyOwner {
        paused = true;
        emit Pause(msg.sender);
    }

    /// @dev Unpauses the contract.
    function unpause() external virtual onlyOwner {
        paused = false;
        emit Unpause(msg.sender);
    }
}
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L8-L56

**Mitigation:**
- **Single Modifier for Ownership Check:** A modifier named `onlyOwner` was introduced, encapsulating the ownership verification logic. This modifier is applied to functions requiring owner-level access control, reducing code duplication by centralizing the ownership check into a single reusable modifier.

- **Gas Efficiency:** By utilizing the `onlyOwner` modifier, the need for multiple repetitive checks for ownership against the `owner` state variable has been eliminated. This optimization reduces gas consumption by avoiding redundant state variable reads from storage during function execution.




## [G-04]Gas Efficiency Enhancement via Ownership Modifier in GenericRegistry Contract
#### Note: These are instances the automated report missed
**Description:** The optimization of the contract focused on improving gas efficiency by consolidating the ownership verification logic into a single reusable modifier within the `GenericRegistry` abstract contract. This adjustment aims to reduce gas consumption by minimizing redundant state variable reads from storage.

**Original code**
```solidity
File: registries/contracts/GenericRegistry.sol
    function changeOwner(address newOwner) external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Changes the unit manager.
    /// @param newManager Address of a new unit manager.
    function changeManager(address newManager) external virtual {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newManager == address(0)) {
            revert ZeroAddress();
        }

        manager = newManager;
        emit ManagerUpdated(newManager);
    }

    /// @dev Checks for the unit existence.
    /// @notice Unit counter starts from 1.
    /// @param unitId Unit Id.
    /// @return true if the unit exists, false otherwise.
    function exists(uint256 unitId) external view virtual returns (bool) {
        return unitId > 0 && unitId < (totalSupply + 1);
    }
    
    /// @dev Sets unit base URI.
    /// @param bURI Base URI string.
    function setBaseURI(string memory bURI) external virtual {
        // Check for the ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero value
        if (bytes(bURI).length == 0) {
            revert ZeroValue();
        }

        baseURI = bURI;
        emit BaseURIChanged(bURI);
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L37-L91

**optimized code**


```solidity
File: registries/contracts/GenericRegistry.sol
    // Check for the ownership
    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
    }

    function changeOwner(address newOwner) external virtual onlyOwner {
        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Changes the unit manager.
    /// @param newManager Address of a new unit manager.
    function changeManager(address newManager) external virtual onlyOwner {
        // Check for the zero address
        if (newManager == address(0)) {
            revert ZeroAddress();
        }

        manager = newManager;
        emit ManagerUpdated(newManager);
    }

    /// @dev Checks for the unit existence.
    /// @notice Unit counter starts from 1.
    /// @param unitId Unit Id.
    /// @return true if the unit exists, false otherwise.
    function exists(uint256 unitId) external view virtual returns (bool) {
        return unitId > 0 && unitId < (totalSupply + 1);
    }
    
    /// @dev Sets unit base URI.
    /// @param bURI Base URI string.
    function setBaseURI(string memory bURI) external virtual onlyOwner {
        // Check for the zero value
        if (bytes(bURI).length == 0) {
            revert ZeroValue();
        }

        baseURI = bURI;
        emit BaseURIChanged(bURI);
    }

```


**Mitigation:**
- **Single Modifier for Ownership Check:** The introduction of the `onlyOwner` modifier encapsulates the ownership verification logic. This modifier has been applied to functions requiring owner-level access control, eliminating repetitive checks against the `owner` state variable and consolidating the logic into a single reusable modifier.

- **Gas Savings:** By utilizing the `onlyOwner` modifier, the need for multiple redundant checks for ownership against the `owner` state variable has been eliminated. This optimization effectively reduces gas consumption by avoiding repetitive state variable reads from storage during function execution.






## [G-05]Optimizing State Variable Access in Dispenser Contract
#### Note: These are instances the automated report missed
**Description:**
In Solidity contracts, accessing state variables directly incurs gas costs, especially when done multiple times within a function or across multiple functions. This can result in inefficiencies and increased gas consumption. 


**Original code**
```solidity
File: tokenomics/contracts/Dispenser.sol
    function changeOwner(address newOwner) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Changes various managing contract addresses.
    /// @param _tokenomics Tokenomics address.
    /// @param _treasury Treasury address.
    function changeManagers(address _tokenomics, address _treasury) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Change Tokenomics contract address
        if (_tokenomics != address(0)) {
            tokenomics = _tokenomics;
            emit TokenomicsUpdated(_tokenomics);
        }
        // Change Treasury contract address
        if (_treasury != address(0)) {
            treasury = _treasury;
            emit TreasuryUpdated(_treasury);
        }
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L46-L80

The provided code contains multiple checks for the contract ownership within different functions, each verifying if the `msg.sender` matches the `owner` state variable. These checks result in redundant state variable reads, impacting gas costs.


**optimized code**
**Mitigation:**
1. Implement a modifier for `onlyOwner`:

```solidity
modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        _;
}
```

2. Apply the `onlyOwner` modifier to functions where ownership validation is required:

```solidity
function changeOwner(address newOwner) external onlyOwner {
    // Check for the zero address
    if (newOwner == address(0)) {
        revert ZeroAddress();
    }

    owner = newOwner;
    emit OwnerUpdated(newOwner);
}

function changeManagers(address _tokenomics, address _treasury) external onlyOwner {
    // Change Tokenomics contract address
    if (_tokenomics != address(0)) {
        tokenomics = _tokenomics;
        emit TokenomicsUpdated(_tokenomics);
    }
    // Change Treasury contract address
    if (_treasury != address(0)) {
        treasury = _treasury;
        emit TreasuryUpdated(_treasury);
    }
}
```

By using the `onlyOwner` modifier, we eliminate the need to directly access the `owner` state variable multiple times across functions. This approach reduces gas costs by caching the owner's address in the stack variable, thereby optimizing the contract's gas consumption during execution.



## [G-06]Optimizing State Variable Access and Reducing Gas Costs in Depository Contract
#### Note: These are instances the automated report missed
**Description:**
The provided Solidity code contains multiple functions that frequently access the `owner` state variable to check for contract ownership. These repeated accesses to the state variable result in increased gas consumption due to storage reads, impacting contract efficiency.

**Original code**
```solidity
File: tokenomics/contracts/Depository.sol
    function changeOwner(address newOwner) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Changes various managing contract addresses.
    /// @param _tokenomics Tokenomics address.
    /// @param _treasury Treasury address.
    /// #if_succeeds {:msg "tokenomics changed"} _tokenomics != address(0) ==> tokenomics == _tokenomics;
    /// #if_succeeds {:msg "treasury changed"} _treasury != address(0) ==> treasury == _treasury;
    function changeManagers(address _tokenomics, address _treasury) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Change Tokenomics contract address
        if (_tokenomics != address(0)) {
            tokenomics = _tokenomics;
            emit TokenomicsUpdated(_tokenomics);
        }
        // Change Treasury contract address
        if (_treasury != address(0)) {
            treasury = _treasury;
            emit TreasuryUpdated(_treasury);
        }
    }

    /// @dev Changes Bond Calculator contract address
    /// #if_succeeds {:msg "bondCalculator changed"} _bondCalculator != address(0) ==> bondCalculator == _bondCalculator;
    function changeBondCalculator(address _bondCalculator) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        if (_bondCalculator != address(0)) {
            bondCalculator = _bondCalculator;
            emit BondCalculatorUpdated(_bondCalculator);
        }
    }

    /// @dev Creates a new bond product.
    /// @param token LP token to be deposited for pairs like OLAS-DAI, OLAS-ETH, etc.
    /// @param priceLP LP token price with 18 additional decimals.
    /// @param supply Supply in OLAS tokens.
    /// @param vesting Vesting period (in seconds).
    /// @return productId New bond product Id.
    /// #if_succeeds {:msg "productCounter increases"} productCounter == old(productCounter) + 1;
    /// #if_succeeds {:msg "isActive"} mapBondProducts[productId].supply > 0 && mapBondProducts[productId].vesting == vesting;
    function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the pool liquidity as the LP price being greater than zero
        if (priceLP == 0) {
            revert ZeroValue();
        }

        // Check the priceLP limit value
        if (priceLP > type(uint160).max) {
            revert Overflow(priceLP, type(uint160).max);
        }

        // Check that the supply is greater than zero
        if (supply == 0) {
            revert ZeroValue();
        }

        // Check the supply limit value
        if (supply > type(uint96).max) {
            revert Overflow(supply, type(uint96).max);
        }

        // Check the vesting minimum limit value
        if (vesting < MIN_VESTING) {
            revert LowerThan(vesting, MIN_VESTING);
        }

        // Check for the maturity time overflow for the current timestamp
        uint256 maturity = block.timestamp + vesting;
        if (maturity > type(uint32).max) {
            revert Overflow(maturity, type(uint32).max);
        }

        // Check if the LP token is enabled
        if (!ITreasury(treasury).isEnabled(token)) {
            revert UnauthorizedToken(token);
        }

        // Check if the bond amount is beyond the limits
        if (!ITokenomics(tokenomics).reserveAmountForBondProgram(supply)) {
            revert LowerThan(ITokenomics(tokenomics).effectiveBond(), supply);
        }

        // Push newly created bond product into the list of products
        productId = productCounter;
        mapBondProducts[productId] = Product(uint160(priceLP), uint32(vesting), token, uint96(supply));
        // Even if we create a bond product every second, 2^32 - 1 is enough for the next 136 years
        productCounter = uint32(productId + 1);
        emit CreateProduct(token, productId, supply, priceLP, vesting);
    }

    /// @dev Closes bonding products.
    /// @notice This will terminate programs regardless of their vesting time.
    /// @param productIds Set of product Ids.
    /// @return closedProductIds Set of closed product Ids.
    /// #if_succeeds {:msg "productCounter not touched"} productCounter == old(productCounter);
    /// #if_succeeds {:msg "success closed"} forall (uint k in productIds) mapBondProducts[productIds[k]].vesting == 0 && mapBondProducts[productIds[k]].supply == 0;
    function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Calculate the number of closed products
        uint256 numProducts = productIds.length;
        uint256[] memory ids = new uint256[](numProducts);
        uint256 numClosedProducts;
        // Traverse to close all possible products
        for (uint256 i = 0; i < numProducts; ++i) {
            uint256 productId = productIds[i];
            // Check if the product is still open by getting its supply amount
            uint256 supply = mapBondProducts[productId].supply;
            // The supply is greater than zero only if the product is active, otherwise it is already closed
            if (supply > 0) {
                // Refund unused OLAS supply from the product if it was not used by the product completely
                ITokenomics(tokenomics).refundFromBondProgram(supply);
                address token = mapBondProducts[productId].token;
                delete mapBondProducts[productId];

                ids[numClosedProducts] = productIds[i];
                ++numClosedProducts;
                emit CloseProduct(token, productId, supply);
            }
        }

        // Get the correct array size of closed product Ids
        closedProductIds = new uint256[](numClosedProducts);
        for (uint256 i = 0; i < numClosedProducts; ++i) {
            closedProductIds[i] = ids[i];
        }
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L123-L277


**optimized code**
**Mitigation:**
1. Implement an `onlyOwner` modifier to centralize and optimize ownership checks:

```solidity
modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        _;
}
```

2. Apply the `onlyOwner` modifier to functions where ownership validation is required:

```solidity
function changeOwner(address newOwner) external onlyOwner {
    // Check for the zero address
    if (newOwner == address(0)) {
        revert ZeroAddress();
    }

    owner = newOwner;
    emit OwnerUpdated(newOwner);
}

function changeManagers(address _tokenomics, address _treasury) external onlyOwner {
    // Change Tokenomics contract address
    if (_tokenomics != address(0)) {
        tokenomics = _tokenomics;
        emit TokenomicsUpdated(_tokenomics);
    }
    // Change Treasury contract address
    if (_treasury != address(0)) {
        treasury = _treasury;
        emit TreasuryUpdated(_treasury);
    }
}

function changeBondCalculator(address _bondCalculator) external onlyOwner {
    if (_bondCalculator != address(0)) {
        bondCalculator = _bondCalculator;
        emit BondCalculatorUpdated(_bondCalculator);
    }
}

function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external onlyOwner returns (uint256 productId) {
    // Function body remains unchanged
    // Implement remaining checks and functionalities
}

function close(uint256[] memory productIds) external onlyOwner returns (uint256[] memory closedProductIds) {
    // Function body remains unchanged
    // Implement remaining checks and functionalities
}
```
By utilizing the `onlyOwner` modifier and applying it to functions requiring ownership validation, the number of times the `owner` state variable is accessed from storage is significantly reduced. This consolidation optimizes gas consumption by minimizing storage reads and enhances the contract's efficiency.




## [G-07]Gas Efficiency Improvement by Consolidating State Variable Access in Tokenomics Contract
#### Note: These are instances the automated report missed
**Description:**
The provided Solidity code repeatedly accesses the `owner` state variable across multiple functions to check for contract ownership. These frequent state variable reads result in increased gas consumption due to storage accesses, impacting contract efficiency.

Multiple functions contain ownership validation checks using the `owner` state variable. This redundancy in accessing the state variable leads to higher gas costs during contract execution.

**Original code**
```solidity
File: tokenomics/contracts/Tokenomics.sol
    function changeTokenomicsImplementation(address implementation) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (implementation == address(0)) {
            revert ZeroAddress();
        }

        // Store the implementation address under the designated storage slot
        assembly {
            sstore(PROXY_TOKENOMICS, implementation)
        }
        emit TokenomicsImplementationUpdated(implementation);
    }

    /// @dev Changes the owner address.
    /// @param newOwner Address of a new owner.
    function changeOwner(address newOwner) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Changes various managing contract addresses.
    /// @param _treasury Treasury address.
    /// @param _depository Depository address.
    /// @param _dispenser Dispenser address.
    function changeManagers(address _treasury, address _depository, address _dispenser) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Change Treasury contract address
        if (_treasury != address(0)) {
            treasury = _treasury;
            emit TreasuryUpdated(_treasury);
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
    }

    /// @dev Changes registries contract addresses.
    /// @param _componentRegistry Component registry address.
    /// @param _agentRegistry Agent registry address.
    /// @param _serviceRegistry Service registry address.
    function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for registries addresses
        if (_componentRegistry != address(0)) {
            componentRegistry = _componentRegistry;
            emit ComponentRegistryUpdated(_componentRegistry);
        }
        if (_agentRegistry != address(0)) {
            agentRegistry = _agentRegistry;
            emit AgentRegistryUpdated(_agentRegistry);
        }
        if (_serviceRegistry != address(0)) {
            serviceRegistry = _serviceRegistry;
            emit ServiceRegistryUpdated(_serviceRegistry);
        }
    }

    /// @dev Changes donator blacklist contract address.
    /// @notice DonatorBlacklist contract can be disabled by setting its address to zero.
    /// @param _donatorBlacklist DonatorBlacklist contract address.
    function changeDonatorBlacklist(address _donatorBlacklist) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        donatorBlacklist = _donatorBlacklist;
        emit DonatorBlacklistUpdated(_donatorBlacklist);
    }

    /// @dev Changes tokenomics parameters.
    /// @notice Parameter values are not updated for those that are passed as zero or out of defined bounds.
    /// @param _devsPerCapital Number of valuable devs can be paid per units of capital per epoch.
    /// @param _codePerDev Number of units of useful code that can be built by a developer during one epoch.
    /// @param _epsilonRate Epsilon rate that contributes to the interest rate value.
    /// @param _epochLen New epoch length.
    /// #if_succeeds {:msg "ep is correct endTime"} epochCounter > 1
    /// ==> mapEpochTokenomics[epochCounter - 1].epochPoint.endTime > mapEpochTokenomics[epochCounter - 2].epochPoint.endTime;
    /// #if_succeeds {:msg "epochLen"} old(_epochLen > MIN_EPOCH_LENGTH && _epochLen <= ONE_YEAR && epochLen != _epochLen) ==> nextEpochLen == _epochLen;
    /// #if_succeeds {:msg "devsPerCapital"} _devsPerCapital > MIN_PARAM_VALUE && _devsPerCapital <= type(uint72).max ==> devsPerCapital == _devsPerCapital;
    /// #if_succeeds {:msg "codePerDev"} _codePerDev > MIN_PARAM_VALUE && _codePerDev <= type(uint72).max ==> codePerDev == _codePerDev;
    /// #if_succeeds {:msg "epsilonRate"} _epsilonRate > 0 && _epsilonRate < 17e18 ==> epsilonRate == _epsilonRate;
    /// #if_succeeds {:msg "veOLASThreshold"} _veOLASThreshold > 0 && _veOLASThreshold <= type(uint96).max ==> nextVeOLASThreshold == _veOLASThreshold;
    function changeTokenomicsParameters(
        uint256 _devsPerCapital,
        uint256 _codePerDev,
        uint256 _epsilonRate,
        uint256 _epochLen,
        uint256 _veOLASThreshold
    ) external
    {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // devsPerCapital is the part of the IDF calculation and thus its change will be accounted for in the next epoch
        if (uint72(_devsPerCapital) > MIN_PARAM_VALUE) {
            devsPerCapital = uint72(_devsPerCapital);
        } else {
            // This is done in order not to pass incorrect parameters into the event
            _devsPerCapital = devsPerCapital;
        }

        // devsPerCapital is the part of the IDF calculation and thus its change will be accounted for in the next epoch
        if (uint72(_codePerDev) > MIN_PARAM_VALUE) {
            codePerDev = uint72(_codePerDev);
        } else {
            // This is done in order not to pass incorrect parameters into the event
            _codePerDev = codePerDev;
        }

        // Check the epsilonRate value for idf to fit in its size
        // 2^64 - 1 < 18.5e18, idf is equal at most 1 + epsilonRate < 18e18, which fits in the variable size
        // epsilonRate is the part of the IDF calculation and thus its change will be accounted for in the next epoch
        if (_epsilonRate > 0 && _epsilonRate <= 17e18) {
            epsilonRate = uint64(_epsilonRate);
        } else {
            _epsilonRate = epsilonRate;
        }

        // Check for the epochLen value to change
        if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) {
            nextEpochLen = uint32(_epochLen);
        } else {
            _epochLen = epochLen;
        }

        // Adjust veOLAS threshold for the next epoch
        if (uint96(_veOLASThreshold) > 0) {
            nextVeOLASThreshold = uint96(_veOLASThreshold);
        } else {
            _veOLASThreshold = veOLASThreshold;
        }

        // Set the flag that tokenomics parameters are requested to be updated (1st bit is set to one)
        tokenomicsParametersUpdated = tokenomicsParametersUpdated | 0x01;
        emit TokenomicsParametersUpdateRequested(epochCounter + 1, _devsPerCapital, _codePerDev, _epsilonRate, _epochLen,
            _veOLASThreshold);
    }

    /// @dev Sets incentive parameter fractions.
    /// @param _rewardComponentFraction Fraction for component owner rewards funded by ETH donations.
    /// @param _rewardAgentFraction Fraction for agent owner rewards funded by ETH donations.
    /// @param _maxBondFraction Fraction for the maxBond that depends on the OLAS inflation.
    /// @param _topUpComponentFraction Fraction for component owners OLAS top-up.
    /// @param _topUpAgentFraction Fraction for agent owners OLAS top-up.
    /// #if_succeeds {:msg "maxBond"} mapEpochTokenomics[epochCounter + 1].epochPoint.maxBondFraction == _maxBondFraction;
    function changeIncentiveFractions(
        uint256 _rewardComponentFraction,
        uint256 _rewardAgentFraction,
        uint256 _maxBondFraction,
        uint256 _topUpComponentFraction,
        uint256 _topUpAgentFraction
    ) external
    {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check that the sum of fractions is 100%
        if (_rewardComponentFraction + _rewardAgentFraction > 100) {
            revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);
        }

        // Same check for top-up fractions
        if (_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction > 100) {
            revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);
        }

        // All the adjustments will be accounted for in the next epoch
        uint256 eCounter = epochCounter + 1;
        TokenomicsPoint storage tp = mapEpochTokenomics[eCounter];
        // 0 stands for components and 1 for agents
        tp.unitPoints[0].rewardUnitFraction = uint8(_rewardComponentFraction);
        tp.unitPoints[1].rewardUnitFraction = uint8(_rewardAgentFraction);
        // Rewards are always distributed in full: the leftovers will be allocated to treasury
        tp.epochPoint.rewardTreasuryFraction = uint8(100 - _rewardComponentFraction - _rewardAgentFraction);

        tp.epochPoint.maxBondFraction = uint8(_maxBondFraction);
        tp.unitPoints[0].topUpUnitFraction = uint8(_topUpComponentFraction);
        tp.unitPoints[1].topUpUnitFraction = uint8(_topUpAgentFraction);

        // Set the flag that incentive fractions are requested to be updated (2nd bit is set to one)
        tokenomicsParametersUpdated = tokenomicsParametersUpdated | 0x02;
        emit IncentiveFractionsUpdateRequested(eCounter, _rewardComponentFraction, _rewardAgentFraction,
            _maxBondFraction, _topUpComponentFraction, _topUpAgentFraction);
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L384-L602

**optimized code**
**Mitigation:**
1. Implement an `onlyOwner` modifier to consolidate ownership checks:

```solidity
modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        _;
}
```

2. Apply the `onlyOwner` modifier to functions where ownership validation is required:

```solidity
function changeTokenomicsImplementation(address implementation) external onlyOwner {
    // Remaining function logic remains unchanged
    // ...
}

function changeOwner(address newOwner) external onlyOwner {
    // Remaining function logic remains unchanged
    // ...
}

function changeManagers(address _treasury, address _depository, address _dispenser) external onlyOwner {
    // Remaining function logic remains unchanged
    // ...
}

function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external onlyOwner {
    // Remaining function logic remains unchanged
    // ...
}

function changeDonatorBlacklist(address _donatorBlacklist) external onlyOwner {
    // Remaining function logic remains unchanged
    // ...
}

function changeTokenomicsParameters(
    uint256 _devsPerCapital,
    uint256 _codePerDev,
    uint256 _epsilonRate,
    uint256 _epochLen,
    uint256 _veOLASThreshold
) external onlyOwner {
    // Remaining function logic remains unchanged
    // ...
}

function changeIncentiveFractions(
    uint256 _rewardComponentFraction,
    uint256 _rewardAgentFraction,
    uint256 _maxBondFraction,
    uint256 _topUpComponentFraction,
    uint256 _topUpAgentFraction
) external onlyOwner {
    // Remaining function logic remains unchanged
    // ...
}
```

By utilizing the `onlyOwner` modifier and applying it to functions requiring ownership validation, the number of times the `owner` state variable is accessed from storage is significantly reduced. This consolidation optimizes gas consumption by minimizing storage reads and enhances the contract's efficiency. The modifier promotes cleaner and more readable code by centralizing ownership-related checks in a reusable and gas-efficient manner.



## [G-08]Optimizing Gas Consumption by Reducing State Variable Access in Tokenomics Contract
#### Note: These are instances the automated report missed
**Description:**
The provided Solidity code contains functions that repeatedly access the `depository` state variable to verify the access permission. These frequent state variable reads result in increased gas consumption due to storage accesses, impacting contract efficiency.

Multiple functions check the `msg.sender` against the `depository` state variable for access permission. This repetitive state variable access leads to higher gas costs during contract execution.

**Original code**
```solidity
File:  tokenomics/contracts/Tokenomics.sol
    function reserveAmountForBondProgram(uint256 amount) external returns (bool success) {
        // Check for the depository access
        if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }

        // Effective bond must be bigger than the requested amount
        uint256 eBond = effectiveBond;
        if (eBond >= amount) {
            // The effective bond value is adjusted with the amount that is reserved for bonding
            // The unrealized part of the bonding amount will be returned when the bonding program is closed
            eBond -= amount;
            effectiveBond = uint96(eBond);
            success = true;
            emit EffectiveBondUpdated(eBond);
        }
    }

    /// @dev Refunds unused bond program amount when the program is closed.
    /// @param amount Amount to be refunded from the closed bond program.
    /// #if_succeeds {:msg "effectiveBond"} old(effectiveBond + amount) <= type(uint96).max ==> effectiveBond == old(effectiveBond) + amount;
    function refundFromBondProgram(uint256 amount) external {
        // Check for the depository access
        if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }

        uint256 eBond = effectiveBond + amount;
        // This scenario is not realistically possible. It is only possible when closing the bonding program
        // with the effectiveBond value close to uint96 max
        if (eBond > type(uint96).max) {
            revert Overflow(eBond, type(uint96).max);
        }
        effectiveBond = uint96(eBond);
        emit EffectiveBondUpdated(eBond);
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L609-L644

**optimized code**
**Mitigation:**
1. Implement an `onlyDepository` modifier to centralize access permission checks:

```solidity
modifier onlyDepository() {
        if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }
       _;
}
```

2. Apply the `onlyDepository` modifier to functions requiring depository access validation:

```solidity
function reserveAmountForBondProgram(uint256 amount) external onlyDepository returns (bool success) {
    // Remaining function logic remains unchanged
    // ...
}

function refundFromBondProgram(uint256 amount) external onlyDepository {
    // Remaining function logic remains unchanged
    // ...
}
```

By implementing the `onlyDepository` modifier and applying it to functions requiring depository access validation, the number of times the `depository` state variable is accessed from storage is significantly reduced. This consolidation optimizes gas consumption by minimizing storage reads and enhances the contract's efficiency. The modifier promotes cleaner and more readable code by centralizing permission-related checks in a reusable and gas-efficient manner.


## [G-09]Enhancing Gas Efficiency in BridgedERC20 Contract
#### Note: These are instances the automated report missed
The current implementation of the BridgedERC20 contract was evaluated, revealing significant gas-consuming patterns:
1. **Excessive Access to `owner` State Variable:**
   - The `owner` state variable was accessed multiple times across various functions, resulting in increased gas consumption.
   - Multiple conditional checks were being made, contributing to higher gas costs for each function call.
**Optimization Strategy and Code Changes:**
The following optimizations were proposed and implemented to reduce gas usage in the BridgedERC20 contract:
1. **Reducing `owner` State Variable Access:**
   - Introduced a `onlyOwner` modifier to restrict direct access to the `owner` state variable, significantly reducing the number of times the variable is accessed.
2. **Consolidating Conditional Checks:**
   - Replaced redundant conditional checks with the `onlyOwner` modifier in functions `changeOwner`, `mint`, and `burn`, streamlining the code and reducing gas costs associated with excessive conditional checks.
```solidity
File: governance/contracts/bridges/BridgedERC20.sol
contract BridgedERC20 is ERC20 {
    event OwnerUpdated(address indexed owner);

    // Bridged token owner
    address public owner;

    constructor(string memory _name, string memory _symbol, uint8 _decimals) ERC20(_name, _symbol, _decimals) {
        owner = msg.sender;
    }

    /// @dev Changes the owner address.
    /// @param newOwner Address of a new owner.
    function changeOwner(address newOwner) external {
        // Only the contract owner is allowed to change the owner
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Zero address check
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Mints bridged tokens.
    /// @param account Account address.
    /// @param amount Bridged token amount.
    function mint(address account, uint256 amount) external {
        // Only the contract owner is allowed to mint
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        
        _mint(account, amount);
    }

    /// @dev Burns bridged tokens.
    /// @param amount Bridged token amount to burn.
    function burn(uint256 amount) external {
        // Only the contract owner is allowed to burn
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        _burn(msg.sender, amount);
    }
}
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L18-L6



**Optimized Code Changes:**

```solidity
contract BridgedERC20 is ERC20 {
    event OwnerUpdated(address indexed owner);

    // Bridged token owner
    address public owner;

    constructor(string memory _name, string memory _symbol, uint8 _decimals) ERC20(_name, _symbol, _decimals) {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        _;
    }

    /// @dev Changes the owner address.
    /// @param newOwner Address of a new owner.
    function changeOwner(address newOwner) external onlyOwner {
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

    /// @dev Mints bridged tokens.
    /// @param account Account address.
    /// @param amount Bridged token amount.
    function mint(address account, uint256 amount) external onlyOwner {
        _mint(account, amount);
    }

    /// @dev Burns bridged tokens.
    /// @param amount Bridged token amount to burn.
    function burn(uint256 amount) external onlyOwner {
        _burn(msg.sender, amount);
    }
}
```
**Explanation of Gas Saving:**
1. **Reduced `owner` State Variable Access:**
   - The introduction of the `onlyOwner` modifier effectively limits direct access to the `owner` state variable to only two instances across all functions, significantly reducing gas consumption from six accesses to two.

2. **Streamlined Conditional Checks:**
   - Consolidating conditional checks into the `onlyOwner` modifier removed redundant checks within functions, resulting in a more efficient code execution and reduced gas usage for each function call.



## [G-10]Reducing Gas Consumption in Loop Iteration
#### Note: These are instances the automated report missed
The current contract contains a gas inefficiency when accessing a state mapping `mapBridgeMediatorL1L2ChainIds` within a loop, impacting gas costs during function execution.

### Explanation
The function `_verifySchedule` accesses the `mapBridgeMediatorL1L2ChainIds` mapping within a loop, leading to repeated state variable reads. Each read operation incurs gas costs, potentially increasing the overall gas consumption due to repetitive state accesses within the loop.

```solidity
File: governance/contracts/multisigs/GuardCM.sol
362  uint256 bridgeMediatorL2ChainId = mapBridgeMediatorL1L2ChainIds[targets[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L362

By creating a `bridgeMediatorCache` local mapping variable and referencing the state mapping `mapBridgeMediatorL1L2ChainIds` outside the loop, gas consumption is minimized as it reduces the repetitive state variable reads during the loop iteration.
```diff
 function _verifySchedule(bytes memory data, bytes4 selector) internal {
+    // Create a local mapping to cache values from mapBridgeMediatorL1L2ChainIds
+    mapping(address => uint256) memory bridgeMediatorCache = mapBridgeMediatorL1L2ChainIds;

    // Traverse all the schedule targets and selectors extracted from calldatas
    for (uint i = 0; i < targets.length; ++i) {
        // Get the bridgeMediatorL2 and L2 chain Id from the cached local mapping
-       uint256 bridgeMediatorL2ChainId = mapBridgeMediatorL1L2ChainIds[targets[i]];
+       uint256 bridgeMediatorL2ChainId = bridgeMediatorCache[targets[i]];
        // bridgeMediatorL2 occupies first 160 bits
        address bridgeMediatorL2 = address(uint160(bridgeMediatorL2ChainId));

        // Rest of the logic remains the same...
    }
 }

```

### Mitigation

To mitigate the gas issue, consider implementing a local mapping variable to cache values from the state mapping outside the loop. This approach minimizes gas costs by reducing the number of state variable reads during the loop execution.





## [G-11] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

#### Note: These are instances the automated report missed
There are 8 instance(s) of this issue:
```solidity
File: governance/contracts/bridges/HomeMediator.sol
105  function processMessageFromForeign(bytes memory data) external {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L105

```solidity
File: governance/contracts/multisigs/GuardCM.sol
387    function checkTransaction(
        address to,
        uint256,
        bytes memory data,
        Enum.Operation operation,
        uint256,
        uint256,
        uint256,
        address,
        address payable,
        bytes memory,
        address
    ) external {

441  function setTargetSelectorChainIds(
        address[] memory targets,
        bytes4[] memory selectors,
        uint256[] memory chainIds,
        bool[] memory statuses
    ) external {


495  function setBridgeMediatorChainIds(
        address[] memory bridgeMediatorL1s,
        address[] memory bridgeMediatorL2s,
        uint256[] memory chainIds
    ) external {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L387-L399


```solidity
File: registries/contracts/AgentRegistry.sol
70   function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L70


```solidity
File: registries/contracts/GenericRegistry.sol
78  function setBaseURI(string memory bURI) external virtual {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L78


```solidity
File: registries/contracts/RegistriesManager.sol
27  function create(
        IRegistry.UnitType unitType,
        address unitOwner,
        bytes32 unitHash,
        uint32[] memory dependencies
    ) external returns (uint256 unitId)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L27-L32


```solidity
File: registries/contracts/UnitRegistry.sol
49    function create(address unitOwner, bytes32 unitHash, uint32[] memory dependencies)
50        external virtual returns (uint256 unitId)
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L49-L50


## [G-12] Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

#### Note: These are instances the automated report missed
There are 3 instance(s) of this issue:
```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol
50   mapping(address => uint64) public mapPositionAccountLiquidity;
51   mapping(address => address) public mapPositionAccountPdaAta;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L50-L51


```solidity
File: registries/contracts/UnitRegistry.sol
29  mapping(uint256 => bytes32[]) public mapUnitIdHashes;
    // Map of unit Id => set of subcomponents (possible to derive from any registry)
    mapping(uint256 => uint32[]) public mapSubComponents;
    // Map of unit Id => unit
    mapping(uint256 => Unit) public mapUnits;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L29-L33


```solidity
File: tokenomics/contracts/Depository.sol
98  mapping(uint256 => Bond) public mapUserBonds;
    // Mapping of product Id => bond product instance
    mapping(uint256 => Product) public mapBondProducts;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L98-L100


## [G-13] Avoid transferring amounts of zero in order to save gas
Skipping the external call when nothing will be transferred, will save at least 100 gas

#### Note: These are instances the automated report missed

There are 2 instance(s) of this issue:
```solidity
File: governance/contracts/veOLAS.sol
534  IERC20(token).transfer(msg.sender, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L534


```solidity
File: tokenomics/contracts/Treasury.sol
235  bool success = IToken(token).transferFrom(account, address(this), tokenAmount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L235


## [G-14] Use assembly in place of abi.decode to extract calldata values more efficiently
Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.
[Reffrence](https://code4rena.com/reports/2023-05-juicebox#g-04-use-assembly-in-place-of-abidecode-to-extract-calldata-values-more-efficiently)

#### Note: These are instances the automated report missed

There are 6 instance(s) of this issue:

```solidity
File:  governance/contracts/bridges/FxERC20ChildTunnel.sol
76  (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L76


```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol
74  (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L74


```solidity
File: governance/contracts/multisigs/GuardCM.sol
297   (bytes memory l2Message) = abi.decode(bridgePayload, (bytes));

323   (address fxGovernorTunnel, bytes memory l2Message) = abi.decode(payload, (address, bytes));

352   abi.decode(payload, (address, uint256, bytes, bytes32, bytes32, uint256));

356   abi.decode(payload, (address[], uint256[], bytes[], bytes32, bytes32, uint256));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L297









## [G-15] Optimizing External Call Cache Within Loop in Depository.close() function

### Description:
In the `close` function, an external call (`ITokenomics(tokenomics).refundFromBondProgram(supply)`) is made within a loop, which is inefficient and leads to unnecessary gas consumption. By caching this external call outside the loop, the contract can save gas by avoiding repetitive calls during each iteration.

```solidity
File: tokenomics/contracts/Depository.sol
262       ITokenomics(tokenomics).refundFromBondProgram(supply);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L262


### Mitigation:
- Cached the external call to `ITokenomics(tokenomics)` outside the loop in the `close` function to improve gas efficiency.

- By caching the external call, the gas cost for STATICCALLs per loop iteration (100 gas) is saved, optimizing gas usage and improving overall contract efficiency.

```diff
    function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
        
        // Calculate the number of closed products
        uint256 numProducts = productIds.length;
        uint256[] memory ids = new uint256[](numProducts);
        uint256 numClosedProducts;
+       // Cache external call outside of loop to save gas
+       ITokenomics tokenomicsContract = ITokenomics(tokenomics);
        // Traverse to close all possible products
        for (uint256 i = 0; i < numProducts; ++i) {
            uint256 productId = productIds[i];
            // Check if the product is still open by getting its supply amount
            uint256 supply = mapBondProducts[productId].supply;
            // The supply is greater than zero only if the product is active, otherwise it is already closed
            if (supply > 0) {
                // Refund unused OLAS supply from the product if it was not used by the product completely
-               ITokenomics(tokenomics).refundFromBondProgram(supply);
+               tokenomicsContract.refundFromBondProgram(supply);
                address token = mapBondProducts[productId].token;
                delete mapBondProducts[productId];

                ids[numClosedProducts] = productIds[i];
                ++numClosedProducts;
                emit CloseProduct(token, productId, supply);
            }
        }

        // Get the correct array size of closed product Ids
        closedProductIds = new uint256[](numClosedProducts);
        for (uint256 i = 0; i < numClosedProducts; ++i) {
            closedProductIds[i] = ids[i];
        }
    }
```


## [G-16] Optimizing External Calls in Loop for Gas Efficiency in Tokenomics._trackServiceDonations() function

### Description:
The `_trackServiceDonations` function contains multiple external calls (`IToken(serviceRegistry)`, `IVotingEscrow(ve)`, `IServiceRegistry(serviceRegistry)`) within loops, which can lead to increased gas consumption due to redundant calls. By caching these external calls outside the loop, the contract can save gas by avoiding repetitive external calls during each iteration.

```solidity
File: tokenomics/contracts/Tokenomics.sol
                address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
                    IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
            }

            // Loop over component and agent Ids
            for (uint256 unitType = 0; unitType < 2; ++unitType) {
                // Get the number and set of units in the service
                (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol708-L716
### Mitigation:
- Cached the external calls to `IToken(serviceRegistry)`, `IVotingEscrow(ve)`, `IServiceRegistry(serviceRegistry)` outside the loop in the `_trackServiceDonations` function to improve gas efficiency.
- Utilized cached instances (`_tokenContract`, `_votingEscrowContract`, `_serviceRegistryContract`) within the loop to avoid repeated external calls, resulting in reduced gas costs for STATICCALLs (100 gas per iteration) and enhancing overall contract optimization.



```diff
function _trackServiceDonations(address donator, uint256[] memory serviceIds, uint256[] memory amounts, uint256 curEpoch) internal {
        // Component / agent registry addresses
        address[] memory registries = new address[](2);
        (registries[0], registries[1]) = (componentRegistry, agentRegistry);

        // Check all the unit fractions and identify those that need accounting of incentives
        bool[] memory incentiveFlags = new bool[](4);
        incentiveFlags[0] = (mapEpochTokenomics[curEpoch].unitPoints[0].rewardUnitFraction > 0);
        incentiveFlags[1] = (mapEpochTokenomics[curEpoch].unitPoints[1].rewardUnitFraction > 0);
        incentiveFlags[2] = (mapEpochTokenomics[curEpoch].unitPoints[0].topUpUnitFraction > 0);
        incentiveFlags[3] = (mapEpochTokenomics[curEpoch].unitPoints[1].topUpUnitFraction > 0);

+       // Cache external calls outside of the loop to save gas
+       IToken _tokenContract = IToken(serviceRegistry);
+       IVotingEscrow _votingEscrowContract = IVotingEscrow(ve);
+       IServiceRegistry _serviceRegistryContract = IServiceRegistry(serviceRegistry);

        // Get the number of services
        uint256 numServices = serviceIds.length;
        // Loop over service Ids to calculate their partial contributions
        for (uint256 i = 0; i < numServices; ++i) {
            // Check if the service owner or donator stakes enough OLAS for its components / agents to get a top-up
            // If both component and agent owner top-up fractions are zero, there is no need to call external contract
            // functions to check each service owner veOLAS balance
            bool topUpEligible;
            if (incentiveFlags[2] || incentiveFlags[3]) {
-               address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
+               address serviceOwner = _tokenContract.ownerOf(serviceIds[i]);               
-               topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
+               topUpEligible = (_votingEscrowContract.getVotes(serviceOwner) >= veOLASThreshold  ||
-                   IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
+                   _votingEscrowContract.getVotes(donator) >= veOLASThreshold) ? true : false;
            }

            // Loop over component and agent Ids
            for (uint256 unitType = 0; unitType < 2; ++unitType) {
                // Get the number and set of units in the service
-               (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).
+               (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = _serviceRegistryContract.
                    getUnitIdsOfService(IServiceRegistry.UnitType(unitType), serviceIds[i]);
                // Service has to be deployed at least once to be able to receive donations,
                // otherwise its components and agents are undefined
                if (numServiceUnits == 0) {
                    revert ServiceNeverDeployed(serviceIds[i]);
                }
                // Record amounts data only if at least one incentive unit fraction is not zero
                if (incentiveFlags[unitType] || incentiveFlags[unitType + 2]) {
                    // The amount has to be adjusted for the number of units in the service
                    uint96 amount = uint96(amounts[i] / numServiceUnits);
                    // Accumulate amounts for each unit Id
                    for (uint256 j = 0; j < numServiceUnits; ++j) {
                        // Get the last epoch number the incentives were accumulated for
                        uint256 lastEpoch = mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch;
                        // Check if there were no donations in previous epochs and set the current epoch
                        if (lastEpoch == 0) {
                            mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);
                        } else if (lastEpoch < curEpoch) {
                            // Finalize unit rewards and top-ups if there were pending ones from the previous epoch
                            // Pending incentives are getting finalized during the next epoch the component / agent
                            // receives donations. If this is not the case before claiming incentives, the finalization
                            // happens in the accountOwnerIncentives() where the incentives are issued
                            _finalizeIncentivesForUnitId(lastEpoch, unitType, serviceUnitIds[j]);
                            // Change the last epoch number
                            mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);
                        }
                        // Sum the relative amounts for the corresponding components / agents
                        if (incentiveFlags[unitType]) {
                            mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;
                        }
                        // If eligible, add relative top-up weights in the form of donation amounts.
                        // These weights will represent the fraction of top-ups for each component / agent relative
                        // to the overall amount of top-ups that must be allocated
                        if (topUpEligible && incentiveFlags[unitType + 2]) {
                            mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
                            mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;
                        }
                    }
                }

                // Record new units and new unit owners
                for (uint256 j = 0; j < numServiceUnits; ++j) {
                    // Check if the component / agent is used for the first time
                    if (!mapNewUnits[unitType][serviceUnitIds[j]]) {
                        mapNewUnits[unitType][serviceUnitIds[j]] = true;
                        mapEpochTokenomics[curEpoch].unitPoints[unitType].numNewUnits++;
                        // Check if the owner has introduced component / agent for the first time
                        // This is done together with the new unit check, otherwise it could be just a new unit owner
                        address unitOwner = IToken(registries[unitType]).ownerOf(serviceUnitIds[j]);
                        if (!mapNewOwners[unitOwner]) {
                            mapNewOwners[unitOwner] = true;
                            mapEpochTokenomics[curEpoch].epochPoint.numNewOwners++;
                        }
                    }
                }
            }
        }
    }

```


## [G-17]Optimizing External Call Caching for Gas Efficiency in Tokenomics.trackServiceDonations() function

### Description:
The `trackServiceDonations` function utilizes an external call `IServiceRegistry(serviceRegistry)` within a loop, which can result in increased gas consumption due to redundant calls. By caching this external call outside the loop in the function, gas efficiency can be improved by avoiding repeated external calls during each iteration.

```solidity
File: tokenomics/contracts/Tokenomics.sol
810  if (!IServiceRegistry(serviceRegistry).exists(serviceIds[i])) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L810


### Mitigation:
- Cached the external call `IServiceRegistry(serviceRegistry)` outside the loop in the `trackServiceDonations` function to enhance gas efficiency.
- Utilized the cached instance `_serviceRegistryContract` within the loop to prevent repetitive external calls, resulting in reduced gas costs for STATICCALLs (100 gas per iteration) and overall optimization of gas consumption.


```diff
    function trackServiceDonations(
        address donator,
        uint256[] memory serviceIds,
        uint256[] memory amounts,
        uint256 donationETH
    ) external {
        // Check for the treasury access
        if (treasury != msg.sender) {
            revert ManagerOnly(msg.sender, treasury);
        }

        // Check if the donator blacklist is enabled, and the status of the donator address
        address bList = donatorBlacklist;
        if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) {
            revert DonatorBlacklisted(donator);
        }
+       // Cache external call for gas efficiency
+       IServiceRegistry _serviceRegistryContract = IServiceRegistry(serviceRegistry);
        // Get the number of services
        uint256 numServices = serviceIds.length;
        // Loop over service Ids, accumulate donation value and check for the service existence
        for (uint256 i = 0; i < numServices; ++i) {
            // Check for the service Id existence
-           if (!IServiceRegistry(serviceRegistry).exists(serviceIds[i])) {
+           if (!_serviceRegistryContract.exists(serviceIds[i])) {
                revert ServiceDoesNotExist(serviceIds[i]);
            }
        }
        // Get the current epoch
        uint256 curEpoch = epochCounter;
        // Increase the total service donation balance per epoch
        donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;
        mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH = uint96(donationETH);

        // Track service donations
        _trackServiceDonations(donator, serviceIds, amounts, curEpoch);

        // Set the current block number
        lastDonationBlockNumber = uint32(block.number);
    }
```



## [G-18] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

[Ref]{https://github.com/code-423n4/2023-10-ethena/blob/main/bot-report.md#g-10-optimize-external-calls-with-assembly-for-memory-efficiency}

```solidity
File: governance/contracts/veOLAS.sol
364   IERC20(token).transferFrom(msg.sender, address(this), amount);

534   IERC20(token).transfer(msg.sender, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364


```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol
77  IERC20(rootToken).mint(to, amount);

104 IERC20(rootToken).burn(amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L77


```solidity
File: tokenomics/contracts/Depository.sol
262  ITokenomics(tokenomics).refundFromBondProgram(supply);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L262


```solidity
File: tokenomics/contracts/Treasury.sol
242  IOLAS(olas).mint(msg.sender, olasMintAmount);

298  ITokenomics(tokenomics).trackServiceDonations(msg.sender, serviceIds, amounts, msg.value);

416  IOLAS(olas).mint(account, accountTopUps);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L242



```solidity
function create(
    address[] memory owners,
    uint256 threshold,
    bytes memory data
) external returns (address multisig)
{
    // Check for the correct data length
    uint256 dataLength = data.length;
    if (dataLength < DEFAULT_DATA_LENGTH) {
        revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);
    }

    // Read the proxy multisig address (20 bytes) and the multisig-related data
    assembly {
        multisig := mload(add(data, DEFAULT_DATA_LENGTH))
    }

    // Check that the multisig address corresponds to the authorized multisig proxy bytecode hash
    bytes32 multisigProxyHash = keccak256(multisig.code);
    if (proxyHash != multisigProxyHash) {
        revert UnauthorizedMultisig(multisig);
    }



    // If provided, read the payload that is going to change the multisig ownership and threshold
    // The payload is expected to be the `execTransaction()` function call with all its arguments and signature(s)
    if (dataLength > DEFAULT_DATA_LENGTH) {
        uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
        bytes memory payload = new bytes(payloadLength);
        for (uint256 i = 0; i < payloadLength; ++i) {
            payload[i] = data[i + DEFAULT_DATA_LENGTH];
        }

        // Call the multisig with the provided payload
        (bool success, ) = multisig.call(payload);
        if (!success) {
            revert MultisigExecFailed(multisig);
        }
    }

    // Get the provided proxy multisig owners and threshold without re-calling external functions
    address[] memory checkOwners = gnosisSafe.getOwners();
    uint256 checkThreshold = gnosisSafe.getThreshold();

    // Verify updated multisig proxy for provided owners and threshold
    if (threshold != checkThreshold) {
        revert WrongThreshold(checkThreshold, threshold);
    }
    uint256 numOwners = owners.length;
    if (numOwners != checkOwners.length) {
        revert WrongNumOwners(checkOwners.length, numOwners);
    }
    // The owners' addresses in the multisig itself are stored in reverse order compared to how they were added:
    // https://etherscan.io/address/0xd9db270c1b5e3bd161e8c8503c55ceabee709552#code#F6#L56
    // Thus, the check must be carried out accordingly.
    for (uint256 i = 0; i < numOwners; ++i) {
        if (owners[i] != checkOwners[numOwners - i - 1]) {
            revert WrongOwner(owners[i]);
        }
    }
}
```


## [G-19] Refactor External/Internal Functions to Avoid Redundant External Calls in GnosisSafeSameAddressMultisig.create() function

### Description:
The `create` function performs multiple external calls to `IGnosisSafe(multisig)` within its logic, causing redundant external calls and increasing gas consumption. By caching the external call instance `IGnosisSafe` at the beginning of the function, these redundant calls are eliminated, optimizing gas usage and improving overall efficiency.

```solidity
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol
125     address[] memory checkOwners = IGnosisSafe(multisig).getOwners();
126     uint256 checkThreshold = IGnosisSafe(multisig).getThreshold();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L125-L126


### Mitigation:
- Cached the external call `IGnosisSafe(multisig)` at the beginning of the function to eliminate unnecessary external calls within the function.
- Utilized the cached instance `gnosisSafe` for subsequent function calls to `getOwners()` and `getThreshold()` without re-calling external functions, resulting in reduced gas costs and improved gas efficiency.


```diff
    function create(
        address[] memory owners,
        uint256 threshold,
        bytes memory data
    ) external returns (address multisig)
    {
        // Check for the correct data length
        uint256 dataLength = data.length;
        if (dataLength < DEFAULT_DATA_LENGTH) {
            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);
        }

        // Read the proxy multisig address (20 bytes) and the multisig-related data
        assembly {
            multisig := mload(add(data, DEFAULT_DATA_LENGTH))
        }

        // Check that the multisig address corresponds to the authorized multisig proxy bytecode hash
        bytes32 multisigProxyHash = keccak256(multisig.code);
        if (proxyHash != multisigProxyHash) {
            revert UnauthorizedMultisig(multisig);
        }

+       // Cache external call for gas efficiency
+       IGnosisSafe gnosisSafe = IGnosisSafe(multisig);

        // If provided, read the payload that is going to change the multisig ownership and threshold
        // The payload is expected to be the `execTransaction()` function call with all its arguments and signature(s)
        if (dataLength > DEFAULT_DATA_LENGTH) {
            uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
            bytes memory payload = new bytes(payloadLength);
            for (uint256 i = 0; i < payloadLength; ++i) {
                payload[i] = data[i + DEFAULT_DATA_LENGTH];
            }

            // Call the multisig with the provided payload
            (bool success, ) = multisig.call(payload);
            if (!success) {
                revert MultisigExecFailed(multisig);
            }
        }

        // Get the provided proxy multisig owners and threshold
-       address[] memory checkOwners = IGnosisSafe(multisig).getOwners();
+       address[] memory checkOwners = gnosisSafe.getOwners();
-       uint256 checkThreshold = IGnosisSafe(multisig).getThreshold();
+       uint256 checkThreshold = gnosisSafe.getThreshold();

        // Verify updated multisig proxy for provided owners and threshold
        if (threshold != checkThreshold) {
            revert WrongThreshold(checkThreshold, threshold);
        }
        uint256 numOwners = owners.length;
        if (numOwners != checkOwners.length) {
            revert WrongNumOwners(checkOwners.length, numOwners);
        }
        // The owners' addresses in the multisig itself are stored in reverse order compared to how they were added:
        // https://etherscan.io/address/0xd9db270c1b5e3bd161e8c8503c55ceabee709552#code#F6#L56
        // Thus, the check must be carried out accordingly.
        for (uint256 i = 0; i < numOwners; ++i) {
            if (owners[i] != checkOwners[numOwners - i - 1]) {
                revert WrongOwner(owners[i]);
            }
        }
    }
```


## [G-20] Refactor External/Internal Functions to Avoid Redundant External Calls in Depository.create() function

### Description:
The `create` function previously made multiple external calls to `ITokenomics(tokenomics)` within its logic, resulting in redundant external calls and increased gas consumption. By caching the external call instance `tokenomicsContract` at the beginning of the function, this redundant call has been eliminated, optimizing gas usage and enhancing overall efficiency.

```solidity
File: tokenomics/contracts/Depository.sol
226     if (!ITokenomics(tokenomics).reserveAmountForBondProgram(supply)) {
227         revert LowerThan(ITokenomics(tokenomics).effectiveBond(), supply);
228     }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L226-L228

### Mitigation:
- Cached the external call `ITokenomics(tokenomics)` at the start of the function to eliminate unnecessary external calls within the function.
- Utilized the cached instance `tokenomicsContract` for subsequent function calls without re-calling the external function, resulting in reduced gas costs and improved gas efficiency.


```diff
    function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {
        // ... (rest of the function remains)
+       // Cache external call for gas efficiency
+       ITokenomics tokenomicsContract = ITokenomics(tokenomics);
        // Check if the bond amount is beyond the limits
-       if (!ITokenomics(tokenomics).reserveAmountForBondProgram(supply)) {
+       if (!tokenomicsContract.reserveAmountForBondProgram(supply)) {
-           revert LowerThan(ITokenomics(tokenomics).effectiveBond(), supply);
+           revert LowerThan(tokenomicsContract.effectiveBond(), supply);
        }

        // Push newly created bond product into the list of products
        productId = productCounter;
        mapBondProducts[productId] = Product(uint160(priceLP), uint32(vesting), token, uint96(supply));
        // Even if we create a bond product every second, 2^32 - 1 is enough for the next 136 years
        productCounter = uint32(productId + 1);
        emit CreateProduct(token, productId, supply, priceLP, vesting);
    }

```



## [G-21] Refactor External/Internal Functions to Eliminate Redundant External Calls in Treasury.depositTokenForOLAS() function

### Description:
The `depositTokenForOLAS` function includes multiple external calls to `IToken(token)` within its logic. These repeated external calls can be optimized to improve gas efficiency. By caching the external call instance `tokenContract` at the start of the function, we can prevent redundant calls, reduce gas costs, and enhance overall efficiency.
```solidity
File: tokenomics/contracts/Treasury.sol
228     if (IToken(token).allowance(account, address(this)) < tokenAmount) {
            revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);
        }

        // Transfer tokens from account to treasury and add to the token treasury reserves
        // We assume that authorized LP tokens in the protocol are safe as they are enabled via the governance
        // UniswapV2ERC20 realization has a standard transferFrom() function that returns a boolean value
        bool success = IToken(token).transferFrom(account, address(this), tokenAmount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L228-L235

### Mitigation:
- Cached the external call `IToken(token)` as `tokenContract` at the beginning of the function.
- Avoided multiple calls to `IToken(token)` within the function by using the cached instance `tokenContract`.
- Enhanced gas efficiency by eliminating unnecessary external calls and utilizing the cached instance for subsequent function calls.



```diff
    function depositTokenForOLAS(address account, uint256 tokenAmount, address token, uint256 olasMintAmount) external {
        // Check for the depository access
        if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }

        // Check if the token is authorized by the registry
        if (!mapEnabledTokens[token]) {
            revert UnauthorizedToken(token);
        }

+       // Cache external call IToken(token)
+       IToken tokenContract = IToken(token);

        // Increase the amount of LP token reserves
        uint256 reserves = mapTokenReserves[token] + tokenAmount;
        mapTokenReserves[token] = reserves;

        // Uniswap allowance implementation does not revert with the accurate message, need to check before the transfer is engaged
-       if (IToken(token).allowance(account, address(this)) < tokenAmount) {
+       if (tokenContract.allowance(account, address(this)) < tokenAmount) {    
-           revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);
+           revert InsufficientAllowance(tokenContract.allowance((account), address(this)), tokenAmount);
        }

        // Transfer tokens from account to treasury and add to the token treasury reserves
        // We assume that authorized LP tokens in the protocol are safe as they are enabled via the governance
        // UniswapV2ERC20 realization has a standard transferFrom() function that returns a boolean value
-       bool success = IToken(token).transferFrom(account, address(this), tokenAmount);
+       bool success = tokenContract.transferFrom(account, address(this), tokenAmount);
        if (!success) {
            revert TransferFailed(token, account, address(this), tokenAmount);
        }

        // Mint specified number of OLAS tokens corresponding to tokens bonding deposit
        // The olasMintAmount is guaranteed by the product supply limit, which is limited by the effectiveBond
        IOLAS(olas).mint(msg.sender, olasMintAmount);

        emit DepositTokenFromAccount(account, token, tokenAmount, olasMintAmount);
    }
```



## [G-22] Minimize Redundant External Calls in `drainServiceSlashedFunds()` Function

### Description:
The `drainServiceSlashedFunds()` function contains redundant external calls to `IServiceRegistry(serviceRegistry)`. These repeated calls could be avoided by caching the result of the external call within the function. The current implementation executes the same external call multiple times, which results in increased gas consumption. 

```solidity
File: tokenomics/contracts/Treasury.sol
476  uint256 slashedFunds = IServiceRegistry(serviceRegistry).slashedFunds();

482  amount = IServiceRegistry(serviceRegistry).drain();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L476-L482

### Mitigation:
To optimize gas usage, the `IServiceRegistry(serviceRegistry)` call can be cached as a contract instance within the function. By doing this, subsequent calls within the function can utilize the cached instance, reducing gas costs by preventing repetitive external calls. 

Example of Optimized Code:

```diff
    function drainServiceSlashedFunds() external returns (uint256 amount) {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Get the service registry contract address
        address serviceRegistry = ITokenomics(tokenomics).serviceRegistry();
+       // Cache external call to serviceRegistryContract
+       IServiceRegistry serviceRegistryContract = IServiceRegistry(serviceRegistry);
        // Check if the amount of slashed funds are at least the minimum required amount to receive by the Treasury
-       uint256 slashedFunds = IServiceRegistry(serviceRegistry).slashedFunds();
+       uint256 slashedFunds = serviceRegistryContract.slashedFunds();
        if (slashedFunds < minAcceptedETH) {
            revert LowerThan(slashedFunds, minAcceptedETH);
        }

        // Call the service registry drain function
-       amount = IServiceRegistry(serviceRegistry).drain();
+       amount = serviceRegistryContract.drain();
    }

```


## [G-23] Gas Optimization by Changing Storage Variable Visibility in veOLAS Contract

Description:
In the `veOLAS` contract, the visibility of the `supply` storage variable is currently set to `public`. Additionally, there exists a function `totalSupply()` that retrieves the value of `supply` using a `public view` modifier.

There are 1 instance(s) of this issue:
```solidity
File: governance/contracts/veOLAS.sol
110  uint256 public supply;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L110

Mitigation:
To optimize gas consumption, it is recommended to change the visibility of the `supply` storage variable from `public` to `private`. The reason for this gas-saving measure is to avoid unnecessary gas costs incurred when accessing storage variables directly from external contracts or interfaces.

Reasoning:
Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.


## [G-24] Utilize Constants Instead of `type(uintx).max`

Using constants instead of `type(uintx).max` can optimize gas usage as constants are precomputed at compile-time and reusable throughout the code. In contrast, `type(uintx).max` requires runtime computation.

### Example:

Consider replacing `type(uint256).max` with a constant declaration:
Optimized Code Using Constants:
```solidity
uint256 constant MAX_VALUE = 2**256 - 1;

// then use
uint256 maxValue = MAX_VALUE;
```

The use of a constant (`MAX_VALUE`) precomputed at compile-time reduces gas consumption compared to calculating `type(uint256).max` at runtime. Always prefer constants for fixed values to improve gas efficiency.

there are 28 insteances of this issue
```solidity
File: governance/contracts/OLAS.sol
134  if (spenderAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L134


```solidity
File:   governance/contracts/veOLAS.sol
392     if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);


449     if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);

474     if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);                        
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392-L393


```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol
52  address[type(uint32).max] public positionAccounts;

95  if (positionData.liquidity > type(uint64).max) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52


```solidity
File: registries/contracts/UnitRegistry.sol
232  uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L232


```solidity
File: tokenomics/contracts/Depository.sol
195       if (priceLP > type(uint160).max) {
            revert Overflow(priceLP, type(uint160).max);

205       if (supply > type(uint96).max) {
            revert Overflow(supply, type(uint96).max); 

216       if (maturity > type(uint32).max) {
            revert Overflow(maturity, type(uint32).max);

311       if (maturity > type(uint32).max) {
            revert Overflow(maturity, type(uint32).max);                                  
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195-L196


```solidity
File: tokenomics/contracts/GenericBondCalculator.sol
59       if (totalTokenValue > type(uint192).max) {
            revert Overflow(totalTokenValue, type(uint192).max);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59-L60


```solidity
File: tokenomics/contracts/Tokenomics.sol
639       if (eBond > type(uint96).max) {
            revert Overflow(eBond, type(uint96).max);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639-L640


```solidity
File: tokenomics/contracts/Treasury.sol
128      if (amount + ETHFromServices > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);

194      if (_minAcceptedETH > type(uint96).max) {
            revert Overflow(_minAcceptedETH, type(uint96).max);   

291      if (donationETH + ETHOwned > type(uint96).max) {
            revert Overflow(donationETH, type(uint96).max);                     
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L128-L129



## [G-25] Admin functions can be payable
We can make admin specific functions payable to save gas, because the compiler won’t be checking the callvalue of the function.

This will also make the contract smaller and cheaper to deploy as there will be fewer opcodes in the creation and runtime code.

There are 43 instances of this issue:

```solidity
File: governance/contracts/OLAS.sol
43  function changeOwner(address newOwner) external {

58  function changeMinter(address newMinter) external {

75  function mint(address account, uint256 amount) external {        
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L43


```solidity
File: governance/contracts/bridges/BridgedERC20.sol
30  function changeOwner(address newOwner) external {

48  function mint(address account, uint256 amount) external {

59  function burn(uint256 amount) external {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L30


```solidity
File: governance/contracts/multisigs/GuardCM.sol
154  function changeGovernor(address newGovernor) external {

170  function changeGovernorCheckProposalId(uint256 proposalId) external {

441  function setTargetSelectorChainIds(

495  function setBridgeMediatorChainIds(

539  function pause() external {

560  function unpause() external {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L154


```solidity
File: registries/contracts/GenericManager.sol
20  function changeOwner(address newOwner) external virtual {

36  function pause() external virtual {

47  function unpause() external virtual {        
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L20


```solidity
File: registries/contracts/GenericRegistry.sol
37  function changeOwner(address newOwner) external virtual {

54  function changeManager(address newManager) external virtual {

78  function setBaseURI(string memory bURI) external virtual {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L37


```solidity
File: tokenomics/contracts/Depository.sol
123    function changeOwner(address newOwner) external {

143    function changeManagers(address _tokenomics, address _treasury) external {

163    function changeBondCalculator(address _bondCalculator) external {

183    function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {
      
244    function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L123


```solidity
File: tokenomics/contracts/Dispenser.sol
46   function changeOwner(address newOwner) external {

64   function changeManagers(address _tokenomics, address _treasury) external {    
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L46


```solidity
File: tokenomics/contracts/DonatorBlacklist.sol
36   function changeOwner(address newOwner) external {

56   function setDonatorsStatuses(address[] memory accounts, bool[] memory statuses) external returns (bool success) {    
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L36


```solidity
File: tokenomics/contracts/Tokenomics.sol
384  function changeTokenomicsImplementation(address implementation) external {

404  function changeOwner(address newOwner) external {

423  function changeManagers(address _treasury, address _depository, address _dispenser) external {

450  function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {

447  function changeDonatorBlacklist(address _donatorBlacklist) external {

497  function changeTokenomicsParameters(

562  function changeIncentiveFractions(        
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L384


```solidity
File: tokenomics/contracts/Treasury.sol
137  function changeOwner(address newOwner) external {

156  function changeManagers(address _tokenomics, address _depository, address _dispenser) external {

182  function changeMinAcceptedETH(uint256 _minAcceptedETH) external {

313  function withdraw(address to, uint256 tokenAmount, address token) external returns (bool success) {

466  function drainServiceSlashedFunds() external returns (uint256 amount) {

487  function enableToken(address token) external {

507  function disableToken(address token) external {

531  function pause() external {

542  function unpause() external {                
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L137


## [G-26] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

There are 5 instance(s) of this issue:

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol
11  string public constant VERSION = "1.0.1";
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L11


```solidity
File: tokenomics/contracts/Depository.sol
77   string public constant VERSION = "1.0.1";
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L77


```solidity
File: registries/contracts/GenericRegistry.sol
33  string public constant CID_PREFIX = "f01701220";
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L33


```solidity
File: registries/contracts/ComponentRegistry.sol
10  string public constant VERSION = "1.0.0";
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L10


```solidity
File: registries/contracts/AgentRegistry.sol
13   string public constant VERSION = "1.0.0";
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L13

## [G-27] Use selfbalance() instead of address(this).balance
it's recommended to use the selfbalance() function instead of address(this).balance. The selfbalance() function is a built-in Solidity function that returns the balance of the current contract in Wei and is considered more gas-efficient and secure.

There are 4 instances of this issue:
```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol
147  if (value > address(this).balance) {

148  revert InsufficientBalance(value, address(this).balance);    
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147


```solidity
File: governance/contracts/bridges/HomeMediator.sol
147          if (value > address(this).balance) {
                revert InsufficientBalance(value, address(this).balance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147


```solidity
File: tokenomics/contracts/Treasury.sol
112   ETHOwned = uint96(address(this).balance);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L112


## [G-28] Pre-increment and pre-decrement are cheaper than +1 ,-1

There are 5 instances of this issue:
```solidity
File: registries/contracts/GenericRegistry.sol
98   unitId = id + 1;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L98


```solidity
File: tokenomics/contracts/Tokenomics.sol
586  uint256 eCounter = epochCounter + 1;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L586


```solidity
File: governance/contracts/veOLAS.sol
263  curNumPoint += 1;

556  maxPointNumber -= 1;

577  maxPointNumber = mid - 1;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L263

## [G-29] Ternary operation is cheaper than if-else statement

```solidity
File: governance/contracts/veOLAS.sol
240             if (tStep > block.timestamp) {
                    tStep = uint64(block.timestamp);
                } else {
                    dSlope = mapSlopeChanges[tStep];
                }

568         if (account == address(0)) {
                point = mapSupplyPoints[mid];
            } else {
                point = mapUserPoints[account][mid];
            }

578         if (point.blockNumber < (blockNumber + 1)) {
                minPointNumber = mid;
            } else {
                maxPointNumber = mid - 1;
            }   

582     if (account == address(0)) {
            point = mapSupplyPoints[minPointNumber];
        } else {
            point = mapUserPoints[account][minPointNumber];
        }   

699         if (tStep > ts) {
                tStep = ts;
            } else {
                dSlope = mapSlopeChanges[tStep];
            }                             
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L240-L244


```solidity
   (tStep > block.timestamp) ? tStep = uint64(block.timestamp) : dSlope = mapSlopeChanges[tStep];
```


```solidity
File: tokenomics/contracts/Tokenomics.sol
511     if (uint72(_devsPerCapital) > MIN_PARAM_VALUE) {
            devsPerCapital = uint72(_devsPerCapital);
        } else {
            // This is done in order not to pass incorrect parameters into the event
            _devsPerCapital = devsPerCapital;
        }

529     if (_epsilonRate > 0 && _epsilonRate <= 17e18) {
            epsilonRate = uint64(_epsilonRate);
        } else {
            _epsilonRate = epsilonRate;
        }

        // Check for the epochLen value to change
        if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) {
            nextEpochLen = uint32(_epochLen);
        } else {
            _epochLen = epochLen;
        }

        // Adjust veOLAS threshold for the next epoch
        if (uint96(_veOLASThreshold) > 0) {
            nextVeOLASThreshold = uint96(_veOLASThreshold);
        } else {
            _veOLASThreshold = veOLASThreshold;
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L511-L547


## [G-30] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup

There are 4 instance(s) of this issue:
```solidity
File: governance/contracts/veOLAS.sol
165  return mapUserPoints[account][idx];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L165


```solidity
File: registries/contracts/UnitRegistry.sol
153  unit = mapUnits[unitId];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L153


```solidity
File: tokenomics/contracts/DonatorBlacklist.sol
83   status = mapBlacklistedDonators[account];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L83


```solidity
File: tokenomics/contracts/Treasury.sol
527  enabled = mapEnabledTokens[token];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L527

## [G‑31] Unlimited gas consumption risk due to external call recipients

When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.


```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol
160   (bool success, ) = target.call{value: value}(payload);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L160


```solidity
File: governance/contracts/bridges/HomeMediator.sol
160  (bool success, ) = target.call{value: value}(payload);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L160


## [G-32] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

```solidity
File: registries/contracts/UnitRegistry.sol
81   Unit storage unit = mapUnits[unitId];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L81


```solidity
File: tokenomics/contracts/Tokenomics.sol
970  TokenomicsPoint storage nextEpochPoint = mapEpochTokenomics[eCounter + 1];
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L970


## [G-33] Initialize variables with no default value
If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

As an example:
```
for (uint256 i = 0; i < num.length; ++i) {};
```
should be replaced with
```
for (uint256 i; i < num.length; ++i) {};
```
Consider removing explicit initializations for default values.

[reference](https://gist.github.com/IllIllI000/e075d189c1b23dce256cd166e28f3397)


```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol
345         uint64 liquiditySum = 0;
346         uint32 numPositions = 0;
347         uint64 amountLeft = 0;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L345-L247



#### Note: <font color="red">This issue also exists in all 'for' loops within the scope of the files</font>

