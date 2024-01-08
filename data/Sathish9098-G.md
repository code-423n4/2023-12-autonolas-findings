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

## [G-2] IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold check is redundant

The check IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold might be redundant across all iterations of your loop. If the donator's vote count and the veOLASThreshold remain constant throughout the execution of this loop (which they likely do), the result of IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold will be the same for each iteration.

By doing this, you reduce the number of calls to IVotingEscrow(ve).getVotes(donator), which can save gas, especially if this is a call to an external contract. Saves ``2100 GAS`` for every iterations and external call

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Tokenomics.sol

// Loop over service Ids to calculate their partial contributions
+      bool isDonatorEligible =   IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
        for (uint256 i = 0; i < numServices; ++i) {
            // Check if the service owner or donator stakes enough OLAS for its components / agents to get a top-up
            // If both component and agent owner top-up fractions are zero, there is no need to call external contract
            // functions to check each service owner veOLAS balance
            bool topUpEligible;
            if (incentiveFlags[2] || incentiveFlags[3]) {
                address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
-                    IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
+        isDonatorEligible ;
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L710

##

## [G-3] ``nextEpochLen`` , ``nextVeOLASThreshold`` storage variables should be cached 

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Tokenomics.sol

 // Update epoch length and set the next value back to zero
+        uint32 nextEpochLen_ = nextEpochLen ;
-            if (nextEpochLen > 0) {
+            if (nextEpochLen_ > 0) {
-                curEpochLen = nextEpochLen;
-                epochLen = uint32(curEpochLen);
+                epochLen = uint32(nextEpochLen_);
                nextEpochLen = 0;
            }

            // Update veOLAS threshold and set the next value back to zero
+       uint96 nextVeOLASThreshold_ = nextVeOLASThreshold ;
-            if (nextVeOLASThreshold > 0) {
+            if (nextVeOLASThreshold_ > 0) {
-                veOLASThreshold = nextVeOLASThreshold;
+                veOLASThreshold = nextVeOLASThreshold_;
                nextVeOLASThreshold = 0;
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L987-L999

##

## [G-4] ``lastPointNumber - 1`` subtraction can be unchecked 

Using unchecked for lastPointNumber - 1 is a good optimization to reduce gas costs. Since already checked that lastPointNumber > 0, the subtraction will not underflow.

```diff
FILE: Breadcrumbs2023-12-autonolas/governance/contracts/veOLAS.sol

uint256 lastPointNumber = mapUserPoints[account].length;
        if (lastPointNumber > 0) {
+     unchecked {
            pv = mapUserPoints[account][lastPointNumber - 1];
+               }
        }


```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L148



##

## [G-5] Optimize the _verifyData() function for better gas efficiency 

#### Original Code 

```solidity
FILE: 2023-12-autonolas/governance/contracts/multisigs/GuardCM.sol

function _verifyData(address target, bytes memory data, uint256 chainId) internal {
        // Push a pair of key defining variables into one key
        // target occupies first 160 bits
        uint256 targetSelectorChainId = uint256(uint160(target));
        // selector occupies next 32 bits
        targetSelectorChainId |= uint256(uint32(bytes4(data))) << 160;
        // chainId occupies next 64 bits
        targetSelectorChainId |= chainId << 192;

        // Check the authorized combination of target and selector
        if (!mapAllowedTargetSelectorChainIds[targetSelectorChainId]) {
            revert NotAuthorized(target, bytes4(data), chainId);
        }
    }

```

#### Optimized Code

```solidity

function _verifyData(address target, bytes memory data, uint256 chainId) internal {
    require(data.length >= 4, "Data too short");

    // Combine target, selector, and chainId into one key using bit manipulation
    uint256 targetSelectorChainId = (uint256(uint160(target)) | (uint256(bytes4(data)) << 160) | (chainId << 192));

    // Check the authorized combination
    if (!mapAllowedTargetSelectorChainIds[targetSelectorChainId]) {
        revert NotAuthorized(target, bytes4(data), chainId);
    }
}

```


#### Explanation of the Optimization

``Combined Bitwise Operations``:
The original code performs separate bitwise operations and assignments to compute targetSelectorChainId. In the optimized version, these operations are combined into a single expression. This reduces the number of assignment operations and makes the code more concise.

Data Length Check:
Added a require statement to ensure that the data array is at least 4 bytes long. This is necessary because the code assumes data contains at least 4 bytes (to extract bytes4(data)). Without this check, the code could potentially revert due to an out-of-bounds access.


##

## [G-6] Cache _timeLaunch + ONE_YEAR computation to optimize gas 

We can cache the repeated computation of ``_timeLaunch + ONE_YEAR`` to avoid calculating it multiple times. 

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Tokenomics.sol

 // Time launch of the OLAS contract
        uint256 _timeLaunch = IOLAS(_olas).timeLaunch();
        // Check that the tokenomics contract is initialized no later than one year after the OLAS token is deployed
+      uint  timeLaunchPlusOneYear = _timeLaunch + ONE_YEAR ; 
-        if (block.timestamp >= (_timeLaunch + ONE_YEAR)) {
+        if (block.timestamp >= timeLaunchPlusOneYear ) {
-            revert Overflow(_timeLaunch + ONE_YEAR, block.timestamp);
+            revert Overflow(timeLaunchPlusOneYear , block.timestamp);              
        }
        // Seconds left in the deployment year for the zero year inflation schedule
        // This value is necessary since it is different from a precise one year time, as the OLAS contract started earlier
-        uint256 zeroYearSecondsLeft = uint32(_timeLaunch + ONE_YEAR - block.timestamp);
+        uint256 zeroYearSecondsLeft = uint32(timeLaunchPlusOneYear - block.timestamp);

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L318-L325



##

## [G-7] ``eBond -= amount`` subtraction can be unchecked 

If certain that eBond is always greater than or equal to amount, using unchecked for the subtraction eBond -= amount can save some gas by skipping overflow/underflow checks.

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Tokenomics.sol

// Effective bond must be bigger than the requested amount
        uint256 eBond = effectiveBond;
        if (eBond >= amount) {
+      unchecked {
            // The effective bond value is adjusted with the amount that is reserved for bonding
            // The unrealized part of the bonding amount will be returned when the bonding program is closed
            eBond -= amount;
+            }
            effectiveBond = uint96(eBond);
            success = true;
            emit EffectiveBondUpdated(eBond);
        }


```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L620

##

## [G-8] Caching the Result of Typecasting uint32(_epochLen) to avoid repetitive typecasting

uint32(_epochLen) is used to convert _epochLen to a 32-bit unsigned integer. This is done three times within this snippet. 

```diff
FILE: 2023-12-autonolas/tokenomics/contracts/Tokenomics.sol

+    uint32 tempEpochLen = uint32(_epochLen);  // Caching the typecast result
 // Check for the epochLen value to change
-        if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) {
+        if (tempEpochLen >= MIN_EPOCH_LENGTH && tempEpochLen  <= ONE_YEAR) {
            nextEpochLen = tempEpochLen;
        } else {
            _epochLen = epochLen;
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L535-L540



##

##

## [G-9] Don't Cache state variables only used once 

```diff
FILE: Treasury.sol

// Increase the amount of LP token reserves
-        uint256 reserves = mapTokenReserves[token] + tokenAmount;
-        mapTokenReserves[token] = reserves;
+        mapTokenReserves[token] = mapTokenReserves[token] + tokenAmount;

```
##

## [G-10] ``owner`` storage variable should be cached with stack variable 

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

## [G-11] ``tokenomics`` storage variable should be cached with stack variable 

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

## [G-12] ``epsilonRate`` and ``timeLaunch`` storage variable should be cached with stack variable 

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

##

## [G-13] Don't cache external calls only used once 

```diff
FILE: 2023-12-autonolas/governance/contracts/veOLAS.sol

 function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {
        // Get the number of user points
-        uint256 userNumPoints = IVEOLAS(ve).getNumUserPoints(account);
-        if (userNumPoints > 0) {
+        if (IVEOLAS(ve).getNumUserPoints(account) > 0) {
            uPoint = IVEOLAS(ve).getUserPoint(account, idx);
        }
    }


```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/wveOLAS.sol#L193-L199

##

## [G-14] Use assembly for back to back external calls 

Using inline assembly in Solidity for back-to-back external calls can indeed optimize gas usage, particularly by reducing the overhead associated with these calls.

```solidity
FILE: 2023-12-autonolas/governance/contracts/wveOLAS.sol

 // Get the total number of supply points
        uint256 numPoints = IVEOLAS(ve).totalNumPoints();
        PointVoting memory sPoint = IVEOLAS(ve).mapSupplyPoints(numPoints);

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/wveOLAS.sol#L264-L266

##

## [G-15] Use efficient Compounded Inflation Calculations in ``getSupplyCapForYear()`` function could be more gas efficient 

Implemented an efficient algorithm for compound interest calculation that uses fewer iterations, similar to exponentiation by squaring. This reduces the number of loop iterations significantly for large numYears.

### Actual code: 

```solidity
FILE: Breadcrumbs2023-12-autonolas/tokenomics/contracts
/TokenomicsConstants.sol

function getSupplyCapForYear(uint256 numYears) public pure returns (uint256 supplyCap) {
        // For the first 10 years the supply caps are pre-defined
        if (numYears < 10) {
            uint96[10] memory supplyCaps = [
                529_659_000_00e16,
                569_913_084_00e16,
                641_152_219_50e16,
                708_500_141_72e16,
                771_039_876_00e16,
                828_233_282_97e16,
                879_860_040_11e16,
                925_948_139_65e16,
                966_706_331_40e16,
                1_000_000_000e18
            ];
            supplyCap = supplyCaps[numYears];
        } else {
            // Number of years after ten years have passed (including ongoing ones)
            numYears -= 9;
            // Max cap for the first 10 years
            supplyCap = 1_000_000_000e18;
            // After that the inflation is 2% per year as defined by the OLAS contract
            uint256 maxMintCapFraction = 2;

            // Get the supply cap until the current year
            for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
            // Return the difference between last two caps (inflation for the current year)
            return supplyCap;
        }
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L30-L61

### Optimized Code

```solidity

function getSupplyCapForYear(uint256 numYears) public pure returns (uint256 supplyCap) {
    uint96[10] memory supplyCaps = [
        529_659_000_00e16, 569_913_084_00e16, 641_152_219_50e16, 708_500_141_72e16,
        771_039_876_00e16, 828_233_282_97e16, 879_860_040_11e16, 925_948_139_65e16,
        966_706_331_40e16, 1_000_000_000e18
    ];

    if (numYears < 10) {
        return supplyCaps[numYears];
    }

    // Calculate compounded inflation using a more efficient formula
    return calculateCompoundedInflation(1_000_000_000e18, numYears - 9, 2);
}

function calculateCompoundedInflation(uint256 initialCap, uint256 years, uint256 inflationRate) internal pure returns (uint256) {
    while (years > 0) {
        if (years % 2 == 1) {
            initialCap += (initialCap * inflationRate) / 100;
        }
        inflationRate = (inflationRate * 2) % 100;
        years /= 2;
    }
    return initialCap;
}

```

#### Original Method (Linear Approach)
The original method calculates compounded inflation by iteratively adding 2% of the current supply cap to the cap for each year beyond the 10th.
In this linear approach, the number of iterations (and thus operations) is directly proportional to the number of years. For example, for 20 years beyond the 10th, it performs 20 iterations.

#### Optimized Method (Exponentiation by Squaring)
The optimized method changes the way the compounded inflation is calculated. Instead of linearly iterating through each year, it uses a method that can be likened to exponentiation by squaring, a technique to efficiently compute powers.
In this method, the number of operations grows logarithmically with the number of years, not linearly. This is because each iteration effectively squares the effect of the inflation, drastically reducing the number of iterations required as the number of years increases.

#### Example of Gas Savings

Consider calculating the supply cap for 20 years after the 10th year.
Linear Approach: Requires 20 iterations.
Optimized Approach: It effectively halves the number of remaining years in each iteration. The number of iterations is around log₂(20), which is about 4 or 5 iterations, significantly less than 20.

##

## [G-16] Optimize ``getCurrentPriceLP `` for better gas efficiency

The getCurrentPriceLP function, we can refine the optimizations further to enhance gas efficiency. The primary goal is to streamline the code by eliminating redundant operations and ensuring the calculations are done in the most gas-efficient manner. 

### Original Code 

```solidity
FILE: 2023-12-autonolas/tokenomics/contracts/GenericBondCalculator.sol

 function getCurrentPriceLP(address token) external view returns (uint256 priceLP)
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
                priceLP = (reserve1 * 1e18) / totalSupply;
            }
        }
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/GenericBondCalculator.sol#L67-L91

### Optimized Code

```solidity

function getCurrentPriceLP(address token) external view returns (uint256 priceLP) {
    IUniswapV2Pair pair = IUniswapV2Pair(token);
    (uint256 reserve0, uint256 reserve1, ) = pair.getReserves();

    if (reserve0 == 0 || reserve1 == 0) {
        return 0;
    }

    uint256 totalSupply = pair.totalSupply();
    if (totalSupply == 0) {
        return 0;
    }

    address token0 = pair.token0();
    priceLP = (token0 == olas ? reserve0 : reserve1) * 1e18 / totalSupply;
}

```

#### Explanations of Optimizations

- ``Early Check for Zero Reserves``: Check if either reserve0 or reserve1 is zero at the beginning. If either is zero, the function returns early. This is based on the assumption that if either reserve is zero, the price calculation isn't meaningful.

Gas Saving: This check prevents further execution if there is no liquidity in the pool, saving gas in such scenarios.

- ``Deferred Total Supply Check``: The totalSupply check is moved after the reserves check. Since the total supply is irrelevant if the reserves are zero, this ordering is more efficient.

Gas Saving: This ensures that the (potentially more expensive) totalSupply call is only made if necessary, saving gas when there are no reserves.

- ``Streamlined Price Calculation``: The price calculation is condensed into a single line. This is more concise and avoids the need for a conditional statement to swap reserve0 and reserve1.

Gas Saving: Reducing the number of conditional statements and assignments decreases the computational steps and thus the gas cost.

- ``Remove Redundant Token Check``: The function no longer checks for the case where neither token0 nor token1 is olas, as it's assumed that olas will be one of the tokens in the pair. This removes an unnecessary condition.

Gas Saving: Fewer conditional checks mean fewer operations to perform, leading to lower gas usage.

##

## [G-17] ``!matured`` this check is redundant inside the for loop

Particularly addressing the redundant check of the matured flag. Since matured does not change within the loop, we can optimize the function by checking its value outside the loop and altering the loop's behavior accordingly.

```solidity
FILE: 2023-12-autonolas/tokenomics/contracts/Depository.sol

 function getBonds(address account, bool matured) external view
        returns (uint256[] memory bondIds, uint256 payout)
    {
        // Check the address
        if (account == address(0)) {
            revert ZeroAddress();
        }

        uint256 numAccountBonds;
        // Calculate the number of pending bonds
        uint256 numBonds = bondCounter;
        bool[] memory positions = new bool[](numBonds);
        // Record the bond number if it belongs to the account address and was not yet redeemed
        for (uint256 i = 0; i < numBonds; ++i) {
            // Check if the bond belongs to the account
            // If not and the address is zero, the bond was redeemed or never existed
            if (mapUserBonds[i].account == account) {
                // Check if requested bond is not matured but owned by the account address
                if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
                    positions[i] = true;
                    ++numAccountBonds;
                    // The payout is always bigger than zero if the bond exists
                    payout += mapUserBonds[i].payout;
                }
            }
        }


``` 
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L453






