
|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[G-1](#g-1--use-assembly-to-write-address-storage-values)] | use assembly to write address storage values | Gas Optimization | Informational | 8 |
| [[G-2](#g-2--possible-optimizations-in-close-function)] | Possible Optimizations in `close` function | Gas Optimization | Informational | 1 |
| [[G-3](#g-3--possible-optimizations-in-redeem-function)] | Possible Optimizations in `redeem` function  | Gas Optimization | Informational | 1 |
| [[G-4](#g-4--with-assembly-call-bool-success-transfer-can-be-done-gas-optimized)] |  With assembly, .call (bool success) transfer can be done gas-optimized  | Gas Optimization | Informational | 1 |
| [[G-5](#g-5-arrayindex--amount-is-cheaper-than-arrayindex--arrayindex--amount)] |  array[index] += amount is cheaper than array[index] = array[index] + amount   | Gas Optimization | Informational | 1 |
| [[G-6](#g-6--possible-optimizations-in-depositservicedonationseth-function)] |  Possible Optimizations in `depositServiceDonationsETH` function   | Gas Optimization | Informational | 8 |
| [[G-7](#g-7--possible-optimizations-in-processmessagefromroot-function)] | Possible Optimizations in `processMessageFromRoot` function |Gas Optimization| Informational | 1 |
| [[G-8](#g-8--possible-optimizations-in-setfxchildtunnel-function)] | Possible Optimizations in `setFxChildTunnel` function  | Gas Optimization | Informational | 1 |
| [[G-9](#g-9--possible-optimizations-in-setfxroottunnel-function)] | Possible Optimizations in `setFxRootTunnel` function  | Gas Optimization | Informational | 1 |
| [[G-10](#g-10-use-assembly-to-write-address-storage-values-2)] | Use assembly to write address storage values *2*  | Gas Optimization | Informational | 4 |
| [[G-11](#g-11--possible-optimizations-in-inflationremainder-function)] | Possible Optimizations in `inflationRemainder` function  | Gas Optimization | Informational | 1 |

## **G-1** | Use assembly to write address storage values 

By using assembly, we can directly interact with the storage slot of the storedAddress variable, allowing us to efficiently write and read address values in storage.

```diff
diff --git a/Depository.sol b/aDepository.sol
index f9c7d64..1dd0350 100644
--- a/Depository.sol
+++ b/aDepository.sol
@@ -130,8 +130,10 @@ contract Depository is IErrorsTokenomics {
         if (newOwner == address(0)) {
             revert ZeroAddress();
         }
-
-        owner = newOwner;
+        assembly{
+            sstore(owner.slot,newOwner)
+        }
+        //owner = newOwner;
         emit OwnerUpdated(newOwner);
     }
 
@@ -148,16 +150,21 @@ contract Depository is IErrorsTokenomics {
 
         // Change Tokenomics contract address
         if (_tokenomics != address(0)) {
-            tokenomics = _tokenomics;
+            //tokenomics = _tokenomics;
+            assembly{
+                sstore(tokenomics.slot,_tokenomics)
+            }
             emit TokenomicsUpdated(_tokenomics);
         }
         // Change Treasury contract address
         if (_treasury != address(0)) {
-            treasury = _treasury;
+            //treasury = _treasury;
+            assembly{
+                sstore(treasury.slot,_treasury)
+            }
             emit TreasuryUpdated(_treasury);
         }
     }
-
     /// @dev Changes Bond Calculator contract address
     /// #if_succeeds {:msg "bondCalculator changed"} _bondCalculator != address(0) ==> bondCalculator == _bondCalculator;
     function changeBondCalculator(address _bondCalculator) external {
@@ -167,7 +174,10 @@ contract Depository is IErrorsTokenomics {
         }
 
         if (_bondCalculator != address(0)) {
-            bondCalculator = _bondCalculator;
+            //bondCalculator = _bondCalculator;
+            assembly{
+                sstore(bondCalculator.slot,_bondCalculator)
+            }
             emit BondCalculatorUpdated(_bondCalculator);
         }
     }

```

```diff
diff --git a/Treasury.sol b/aTreasury.sol
index 516c631..959651e 100644
--- a/Treasury.sol
+++ b/aTreasury.sol
@@ -161,17 +161,26 @@ contract Treasury is IErrorsTokenomics {
 
         // Change Tokenomics contract address
         if (_tokenomics != address(0)) {
-            tokenomics = _tokenomics;
+            assembly{
+                sstore(tokenomics.slot,_tokenomics)
+            }
+            //tokenomics = _tokenomics;
             emit TokenomicsUpdated(_tokenomics);
         }
         // Change Depository contract address
         if (_depository != address(0)) {
-            depository = _depository;
+            assembly{
+                sstore(depository.slot,_depository)
+            }
+            //depository = _depository;
             emit DepositoryUpdated(_depository);
         }
         // Change Dispenser contract address
         if (_dispenser != address(0)) {
-            dispenser = _dispenser;
+            assembly{
+                sstore(dispenser.slot,_dispenser)
+            }
+            //dispenser = _dispenser;
             emit DispenserUpdated(_dispenser);
         }
     }

```

```diff
diff --git a/DonatorBlacklist.sol b/aDonatorBlacklist.sol
index 00d84f4..74fcc01 100644
--- a/DonatorBlacklist.sol
+++ b/aDonatorBlacklist.sol
@@ -33,6 +33,7 @@ contract DonatorBlacklist {
 
     /// @dev Changes the owner address.
     /// @param newOwner Address of a new owner.
+    //@audit Gas use assembly
     function changeOwner(address newOwner) external {
         // Check for the contract ownership
         if (msg.sender != owner) {
@@ -43,8 +44,10 @@ contract DonatorBlacklist {
         if (newOwner == address(0)) {
             revert ZeroAddress();
         }
-
-        owner = newOwner;
+        assembly{
+                sstore(owner.slot,newOwner)
+            }
+        //owner = newOwner;
         emit OwnerUpdated(newOwner);
     }
 

```
> Gas diff 

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Depository.changeBondCalculator` | 26936 | 26928 | -8 | -0.03% |
| `Depository.changeManagers` | 30444 | 30429 | -15 | -0.05% |
| `Depository.changeOwner` | 30614 | 30555 | -59 | -0.19% |
| `DonatorBlacklist.changeOwner` | 28073 | 27955 | -118 | -0.42% |
| `Treasury.changeManagers` | 30996 | 30980 | -16 | -0.05% |
| `Depository` | 2057053 | 2024920 | -32133 | -1.56% |
| `DonatorBlacklist` | 505909 | 497715 | -8194 | -1.62% |
| `Treasury` | 2205060 | 2181128 | -23932 | -1.09% |


## **G-2** | Possible Optimizations in `close` function

```diff
diff --git a/Depository.sol b/aDepository.sol
index f9c7d64..44b0260 100644
--- a/Depository.sol
+++ b/aDepository.sol
@@ -251,8 +251,9 @@ contract Depository is IErrorsTokenomics {
         uint256 numProducts = productIds.length;
         uint256[] memory ids = new uint256[](numProducts);
         uint256 numClosedProducts;
+        uint256 i;
         // Traverse to close all possible products
-        for (uint256 i = 0; i < numProducts; ++i) {
+        do{//for (uint256 i = 0; i < numProducts; ++i) {
             uint256 productId = productIds[i];
             // Check if the product is still open by getting its supply amount
             uint256 supply = mapBondProducts[productId].supply;
@@ -267,12 +268,18 @@ contract Depository is IErrorsTokenomics {
                 ++numClosedProducts;
                 emit CloseProduct(token, productId, supply);
             }
-        }
+            unchecked {
+                ++i;
+            }
+        }while(i < numProducts);
 
         // Get the correct array size of closed product Ids
         closedProductIds = new uint256[](numClosedProducts);
-        for (uint256 i = 0; i < numClosedProducts; ++i) {
+        for (uint256 i = 0; i < numClosedProducts;) {
             closedProductIds[i] = ids[i];
+            unchecked{
+                ++i;
+            }
         }
     }
 

```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Depository.close` | 35777 | 35628 | -149 | -0.42% |
| `Depository` | 2057053 | 2049904 | -7149 | -0.35% |


## **G-3** | Possible Optimizations in `redeem` function


```diff
diff --git a/Depository.sol b/aDepository.sol
index f9c7d64..c35e206 100644
--- a/Depository.sol
+++ b/aDepository.sol
@@ -353,24 +353,31 @@ contract Depository is IErrorsTokenomics {
     /// #if_succeeds {:msg "accounts deleted"} forall (uint k in bondIds) mapUserBonds[bondIds[k]].account == address(0);
     /// #if_succeeds {:msg "payouts are zeroed"} forall (uint k in bondIds) mapUserBonds[bondIds[k]].payout == 0;
     /// #if_succeeds {:msg "maturities are zeroed"} forall (uint k in bondIds) mapUserBonds[bondIds[k]].maturity == 0;
-    function redeem(uint256[] memory bondIds) external returns (uint256 payout) {
-        for (uint256 i = 0; i < bondIds.length; ++i) {
+    function redeem(uint256[] calldata bondIds) external returns (uint256 payout) {
+        uint256 i;
+        do{//for (uint256 i = 0; i < bondIds.length; ++i) {
             // Get the amount to pay and the maturity status
             uint256 pay = mapUserBonds[bondIds[i]].payout;
             bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;
 
             // Revert if the bond does not exist or is not matured yet
-            if (pay == 0 || !matured) {
+            if (pay == 0) {
+                
+                revert BondNotRedeemable(bondIds[i]);
+            }
+            if(!matured){
                 revert BondNotRedeemable(bondIds[i]);
             }
-
             // Check that the msg.sender is the owner of the bond
             if (mapUserBonds[bondIds[i]].account != msg.sender) {
                 revert OwnerOnly(msg.sender, mapUserBonds[bondIds[i]].account);
             }
 
             // Increase the payout
-            payout += pay;
+            assembly{
+                payout := add(payout,pay)
+            }
+            //payout += pay;
 
             // Get the productId
             uint256 productId = mapUserBonds[bondIds[i]].productId;
@@ -378,7 +385,10 @@ contract Depository is IErrorsTokenomics {
             // Delete the Bond struct and release the gas
             delete mapUserBonds[bondIds[i]];
             emit RedeemBond(productId, msg.sender, bondIds[i]);
-        }
+            unchecked {
+                ++i;
+            }
+        }while(i < bondIds.length);
 
         // Check for the non-zero payout
         if (payout == 0) {

```

> Gas diff 

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Depository.redeem` | 54118 | 53673 | -445 | -0.82% |
| `Depository` | 2057053 | 2072168 | +15115 | +0.73% |

*note the incress on deployment*

## **G-4** | With assembly, .call (bool success) transfer can be done gas-optimized

`return` data `(bool OK,)` has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```diff
diff --git a/Treasury.sol b/aTreasury.sol
index 516c631..f6126ad 100644
--- a/Treasury.sol
+++ b/aTreasury.sol
@@ -336,7 +336,11 @@ contract Treasury is IErrorsTokenomics {
                 ETHOwned = uint96(amountOwned);
                 emit Withdraw(ETH_TOKEN_ADDRESS, to, tokenAmount);
                 // Send ETH to the specified address
-                (success, ) = to.call{value: tokenAmount}("");
+                //(success, ) = to.call{value: tokenAmount}("");
+                //bool success;
+                assembly {                                    
+                    success := call(gas(), to, tokenAmount, 0, 0, 0, 0)
+                }  
                 if (!success) {
                     revert TransferFailed(ETH_TOKEN_ADDRESS, address(this), to, tokenAmount);
                 }
```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Treasury.withdraw` | 39503 | 39457 | -46 | -0.12% |
| `Treasury` | 2205060 | 2189477 | -15583 | -0.71% |

## **G-5** array[index] += amount is cheaper than array[index] = array[index] + amount

```diff
-        uint256 reserves = mapTokenReserves[token] + tokenAmount;
+        mapTokenReserves[token] += tokenAmount;
```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Treasury.depositTokenForOLAS` | 138161 | 138095 | -66 | -0.05% |
| `Treasury` | 2205060 | 2202480 | -2580 | -0.12% |

## **G-6** | Possible Optimizations in `depositServiceDonationsETH` function

```diff
diff --git a/Treasury.sol b/aTreasury.sol
index 516c631..05dabf6 100644
--- a/Treasury.sol
+++ b/aTreasury.sol
@@ -254,7 +254,7 @@ contract Treasury is IErrorsTokenomics {
     /// #if_succeeds {:msg "updated ETHFromServices"} old(ETHFromServices) + msg.value + old(ETHOwned) <= type(uint96).max && ETHFromServices == old(ETHFromServices) + msg.value
     /// ==> address(this).balance == ETHFromServices + ETHOwned;
     /// #if_succeeds {:msg "any paused"} paused == 1 || paused == 2;
-    function depositServiceDonationsETH(uint256[] memory serviceIds, uint256[] memory amounts) external payable {
+    function depositServiceDonationsETH(uint256[] calldata serviceIds, uint256[] calldata amounts) external payable {
         // Reentrancy guard
         if (_locked > 1) {
             revert ReentrancyGuard();
@@ -273,12 +273,16 @@ contract Treasury is IErrorsTokenomics {
         }
 
         uint256 totalAmount;
-        for (uint256 i = 0; i < numServices; ++i) {
+        uint256 i;
+        do{//for (uint256 i = 0; i < serviceIds.length; ++i) {
             if (amounts[i] == 0) {
                 revert ZeroValue();
             }
             totalAmount += amounts[i];
-        }
+            unchecked {
+                ++i;
+            }
+        }while(i<serviceIds.length);
 
         // Check if the total transferred amount corresponds to the sum of amounts from services
         if (msg.value != totalAmount) {

```
> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Treasury.depositServiceDonationsETH` | 212135 | 210846 | -1289 | -0.61% |
| `Treasury` | 2205060 | 2164097 | -40963 | -1.86% |


## **G-7** | Possible Optimizations in `processMessageFromRoot` function

```diff
diff --git a/FxGovernorTunnel.sol b/aFxGovernorTunnel.sol
index 5b672e5..7ea4a63 100644
--- a/FxGovernorTunnel.sol
+++ b/aFxGovernorTunnel.sol
@@ -122,7 +122,8 @@ contract FxGovernorTunnel is IFxMessageProcessor {
         }
 
         // Unpack and process the data
-        for (uint256 i = 0; i < dataLength;) {
+        uint256 i;
+        do{//for (uint256 i = 0; i < dataLength;) {
             address target;
             uint96 value;
             uint32 payloadLength;
@@ -161,7 +162,7 @@ contract FxGovernorTunnel is IFxMessageProcessor {
             if (!success) {
                 revert TargetExecFailed(target, value, payload);
             }
-        }
+        }while(i < dataLength);
 
         // Emit received message
         emit MessageReceived(stateId, rootMessageSender, data);
```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `FxGovernorTunnel.processMessageFromRoot` | 77660 | 77500 | -160 | -0.21% |
| `FxGovernorTunnel` | 613115 | 611817 | -1298 | -0.21% |


## **G-8** | Possible Optimizations in `setFxChildTunnel` function

```diff
diff --git a/FxBaseRootTunnel.sol b/aFxBaseRootTunnel.sol
index 44c754a..50ea48b 100644
--- a/FxBaseRootTunnel.sol
+++ b/aFxBaseRootTunnel.sol
@@ -52,8 +52,22 @@ abstract contract FxBaseRootTunnel {
 
     // set fxChildTunnel if not set already
     function setFxChildTunnel(address _fxChildTunnel) public virtual {
-        require(fxChildTunnel == address(0x0), "FxBaseRootTunnel: CHILD_TUNNEL_ALREADY_SET");
-        fxChildTunnel = _fxChildTunnel;
+        //require(fxChildTunnel == address(0x0), "FxBaseRootTunnel: CHILD_TUNNEL_ALREADY_SET");
+        assembly{
+            // Custom error message,
+            let revertMessage := "CHILD_TUNNEL_ALREADY_SET"
+
+            // Get the size of the revert message
+            let size := mload(revertMessage)
+
+            // Revert with the custom error message
+            if iszero(fxChildTunnel.slot) {
+                revert(add(revertMessage, 0x20), size)
+                }
+                sstore(fxChildTunnel.slot,_fxChildTunnel)
+                
+        }
+        //fxChildTunnel = _fxChildTunnel;
     }
 
     /**

```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `FxERC20RootTunnel.setFxChildTunnel` | 44050 | 43873 | -177 | -0.40% |
| `FxERC20ChildTunnel` | 713697 | 663287 | -50410 | -7.06% |


## **G-9** | Possible Optimizations in `setFxRootTunnel` function

```diff
diff --git a/FxBaseChildTunnel.sol b/aFxBaseChildTunnel.sol
index 665e571..d09c915 100644
--- a/FxBaseChildTunnel.sol
+++ b/aFxBaseChildTunnel.sol
@@ -29,8 +29,22 @@ abstract contract FxBaseChildTunnel is IFxMessageProcessor {
 
     // set fxRootTunnel if not set already
     function setFxRootTunnel(address _fxRootTunnel) external virtual {
-        require(fxRootTunnel == address(0x0), "FxBaseChildTunnel: ROOT_TUNNEL_ALREADY_SET");
-        fxRootTunnel = _fxRootTunnel;
+        //require(fxRootTunnel == address(0x0), "FxBaseChildTunnel: ROOT_TUNNEL_ALREADY_SET");
+        assembly{
+            // Custom error message,
+            let revertMessage := "ROOT_TUNNEL_ALREADY_SET"
+
+            // Get the size of the revert message
+            let size := mload(revertMessage)
+
+            // Revert with the custom error message
+            if iszero(fxRootTunnel.slot) {
+                revert(add(revertMessage, 0x20), size)
+                }
+                sstore(fxRootTunnel.slot,_fxRootTunnel)
+                
+        }
+        //fxRootTunnel = _fxRootTunnel;
     }
 
     function processMessageFromRoot(uint256 stateId, address rootMessageSender, bytes calldata data) external override {

```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `FxERC20ChildTunnel.setFxRootTunnel` | 44049 | 43872 | -177 | -0.40% |
| `FxERC20RootTunnel` | 2212887 | 2162468 | -50419 | -2.28% |


## **G-10** Use assembly to write address storage values *2*

```diff
diff --git a/OLAS.sol b/aOLAS.sol
index a1700ed..9069715 100644
--- a/OLAS.sol
+++ b/aOLAS.sol
@@ -19,6 +19,8 @@ contract OLAS is ERC20 {
     event OwnerUpdated(address indexed owner);
 
     // One year interval
+    // @audit GAS 
+    // can just write as 365 days
     uint256 public constant oneYear = 1 days * 365;
     // Total supply cap for the first ten years (one billion OLAS tokens)
     uint256 public constant tenYearSupplyCap = 1_000_000_000e18;
@@ -31,10 +33,15 @@ contract OLAS is ERC20 {
     address public owner;
     // Minter address
     address public minter;
-
+    //@audit use assembly to
     constructor() ERC20("Autonolas", "OLAS", 18) {
-        owner = msg.sender;
-        minter = msg.sender;
+        assembly{
+            sstore(owner.slot, caller())
+            sstore(minter.slot, caller())
+
+        }
+        //owner = msg.sender;
+        //minter = msg.sender;
         timeLaunch = block.timestamp;
     }
 
@@ -48,8 +55,10 @@ contract OLAS is ERC20 {
         if (newOwner == address(0)) {
             revert ZeroAddress();
         }
-
-        owner = newOwner;
+        assembly{
+            sstore(owner.slot,newOwner)
+        }
+        // owner = newOwner;
         emit OwnerUpdated(newOwner);
     }
 
@@ -63,8 +72,10 @@ contract OLAS is ERC20 {
         if (newMinter == address(0)) {
             revert ZeroAddress();
         }
-
-        minter = newMinter;
+        assembly{
+            sstore(minter.slot,newMinter)
+        }
+        //minter = newMinter;
         emit MinterUpdated(newMinter);
     }
 
```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `OLAS.changeMinter` | 30099 | 30084 | -15 | -0.05% |
| `OLAS.changeOwner` | 28117 | 28002 | -115 | -0.41% |
| `OLAS` | 1309316 | 1284240 | -25076 | -1.92% |


## **G-11** | Possible Optimizations in `inflationRemainder` function

```diff
diff --git a/OLAS.sol b/aOLAS.sol
index a1700ed..9069715 100644
--- a/OLAS.sol
+++ b/aOLAS.sol
@@ -96,21 +107,38 @@ contract OLAS is ERC20 {
     /// @dev Gets the reminder of OLAS possible for the mint.
     /// @return remainder OLAS token remainder.
     function inflationRemainder() public view returns (uint256 remainder) {
-        uint256 _totalSupply = totalSupply;
+        //uint256 _totalSupply = totalSupply;
         // Current year
-        uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
+        uint256 n = timeLaunch;
+        uint256 numYears ;//= (block.timestamp - timeLaunch) / oneYear;
+        assembly{
+            numYears := sub(timestamp(),n)
+            numYears := div(numYears,oneYear)
+        }
         // Calculate maximum mint amount to date
         uint256 supplyCap = tenYearSupplyCap;
         // After 10 years, adjust supplyCap according to the yearly inflation % set in maxMintCapFraction
         if (numYears > 9) {
             // Number of years after ten years have passed (including ongoing ones)
-            numYears -= 9;
-            for (uint256 i = 0; i < numYears; ++i) {
-                supplyCap += (supplyCap * maxMintCapFraction) / 100;
+            //numYears -= 9;
+            assembly{
+                numYears := sub(numYears,9)
             }
+            //@audit use do while
+            uint256 i;
+            do{//for (uint256 i = 0; i < numYears; ++i) {
+                supplyCap += (supplyCap * 2) / 100;
+                unchecked {
+                    
+                    ++i;
+                }
+            }while(i < numYears);
         }
         // Check for the requested mint overflow
-        remainder = supplyCap - _totalSupply;
+        unchecked{
+            remainder = supplyCap - totalSupply;
+        }
+        
     }
 
     /// @dev Burns OLAS tokens.

```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `OLAS.approve` | 46201 | 46200 | -1 | -0.00% |
| `OLAS.mint` | 65728 | 65525 | -203 | -0.31% |
| `veOLAS.checkpoint` | 234706 | 234702 | -4 | -0.00% |
| `veOLAS.createLock` | 257018 | 257015 | -3 | -0.00% |
| `veOLAS.createLockFor` | 267370 | 267367 | -3 | -0.00% |
| `veOLAS.depositFor` | 163208 | 163211 | +3 | +0.00% |
| `veOLAS.increaseAmount` | 171860 | 171863 | +3 | +0.00% |
| `veOLAS.increaseUnlockTime` | 165857 | 165860 | +3 | +0.00% |
| `veOLAS.withdraw` | 180480 | 180479 | -1 | -0.00% |
| `OLAS` | 1309316 | 1300655 | -8661 | -0.66% |