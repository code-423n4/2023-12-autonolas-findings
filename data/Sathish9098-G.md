# GAS OPTIMIZATIONS

##

## [G-1] ``manager`` and ``_locked`` state variables can be packed within same slot : Saves ``2000 GAS`` , ``1 SLOT``

_locked can be a uint8 instead of a uint256. It only stores the values 1 and 2 because the _locked variable is used solely for the Reentrancy lock. Therefore, using uint8 is perfectly safe and will not affect the protocol.

```diff
FILE: 2023-12-autonolas/registries/contracts/GenericRegistry.sol

// Owner address
    address public owner;
    // Unit manager
    address public manager;
+    // Reentrancy lock
+    uint8 internal _locked = 1;
    // Base URI
    string public baseURI;
    // Unit counter
    uint256 public totalSupply;
    // Reentrancy lock
-    uint256 internal _locked = 1;

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L14-L23

##

## [G-2] ``owner`` storage variable should be cached with stack variable 

Caching the owner in a state variable is more gas-efficient compared to making recursive calls. This approach saves 300 gas and 3 SLOAD

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Depository.sol

123: function changeOwner(address newOwner) external {
        // Check for the contract ownership
+    address owner_ =  owner ; 
- if (msg.sender != owner) {
+ if (msg.sender != owner_) {
-            revert OwnerOnly(msg.sender, owner);
+            revert OwnerOnly(msg.sender, owner_);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }
    
143:    function changeManagers(address _tokenomics, address _treasury) external {
        // Check for the contract ownership
+    address owner_ =  owner ;         
-        if (msg.sender != owner) {
+        if (msg.sender != owner_) {
-            revert OwnerOnly(msg.sender, owner);
+            revert OwnerOnly(msg.sender, owner_);
        }
        
163:         function changeBondCalculator(address _bondCalculator) external {
        // Check for the contract ownership
+    address owner_ =  owner ;         
-        if (msg.sender != owner) {
+        if (msg.sender != owner_) {
-            revert OwnerOnly(msg.sender, owner);
+            revert OwnerOnly(msg.sender, owner_);
        }

183: function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {
        // Check for the contract ownership
+    address owner_ =  owner ;         
-        if (msg.sender != owner) {
+        if (msg.sender != owner_) {
-            revert OwnerOnly(msg.sender, owner);
+            revert OwnerOnly(msg.sender, owner_);
        }
        
244:         function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {
        // Check for the contract ownership
+    address owner_ =  owner ;         
-        if (msg.sender != owner) {
+        if (msg.sender != owner_) {
-            revert OwnerOnly(msg.sender, owner);
+            revert OwnerOnly(msg.sender, owner_);
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L125-L127

##

## [G-3] ``tokenomics`` storage variable should be cached with stack variable 

Caching the tokenomics in a state variable is more gas-efficient compared to making recursive calls. This approach saves 100 gas and 1 SLOAD

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Depository.sol

225: // Check if the bond amount is beyond the limits
+   address tokenomics_ = tokenomics ;
-        if (!ITokenomics(tokenomics).reserveAmountForBondProgram(supply)) {
+        if (!ITokenomics(tokenomics_).reserveAmountForBondProgram(supply)) {
-            revert LowerThan(ITokenomics(tokenomics).effectiveBond(), supply);
+            revert LowerThan(ITokenomics(tokenomics_).effectiveBond(), supply);
        }



```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L226

##

## [G-4] ``epsilonRate`` and ``timeLaunch`` storage variable should be cached with stack variable 

Caching the epsilonRate in a state variable is more gas-efficient compared to making recursive calls. This approach saves 100 gas and 1 SLOAD

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Tokenomics.sol

+  uint64 epsilonRate_ = epsilonRate ; 
 // Compare with epsilon rate and choose the smallest one
-        if (fKD > epsilonRate) {
+        if (fKD > epsilonRate_) {
-            fKD = epsilonRate;
+            fKD = epsilonRate_;
        }
       
924:  // Current year
+    uint32  timeLaunch_ = timeLaunch ;
-        uint256 numYears = (block.timestamp - timeLaunch) / ONE_YEAR;
+        uint256 numYears = (block.timestamp - timeLaunch_) / ONE_YEAR;
        // Amounts for the yearly inflation change from year to year, so if the year changes in the middle
        // of the epoch, it is necessary to adjust epoch inflation numbers to account for the year change
        if (numYears > currentYear) {
            // Calculate remainder of inflation for the passing year
            // End of the year timestamp
 -           uint256 yearEndTime = timeLaunch + numYears * ONE_YEAR;
 +           uint256 yearEndTime = timeLaunch_ + numYears * ONE_YEAR;
        
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L856-L858





