## 1. Refactor the `_checkpoint()` function to save gas.

 Combine ideal if-else statements in one blocks to improve Optimisation
 
```solidity

 // Now mapSupplyPoints is filled until current time
        if (account != address(0)) {
            // If last point was in this block, the slope change has been already applied. In such case we have 0 slope(s)
            lastPoint.slope += (uNew.slope - uOld.slope);
            lastPoint.bias += (uNew.bias - uOld.bias);
            if (lastPoint.slope < 0) {
                lastPoint.slope = 0;
            }
            if (lastPoint.bias < 0) {
                lastPoint.bias = 0;
            }
        }

        // Record the last updated point
        mapSupplyPoints[curNumPoint] = lastPoint;

        if (account != address(0)) {
            // Schedule the slope changes (slope is going down)
            // We subtract new_user_slope from [newLocked.endTime]
            // and add old_user_slope to [oldLocked.endTime]
            if (oldLocked.endTime > block.timestamp) {
                // oldDSlope was <something> - uOld.slope, so we cancel that
                oldDSlope += uOld.slope;
                if (newLocked.endTime == oldLocked.endTime) {
                    oldDSlope -= uNew.slope; // It was a new deposit, not extension
                }


```
In above code snippet we can notice both if-else statement expecting same condition so we can combine both to reduce code size

Recommendation :-

```solidity

        if (account != address(0)) {

             // If last point was in this block, the slope change has been already applied. In such case we have 0 slope(s)
            lastPoint.slope += (uNew.slope - uOld.slope);
            lastPoint.bias += (uNew.bias - uOld.bias);
            if (lastPoint.slope < 0) {
                lastPoint.slope = 0;
            }
            if (lastPoint.bias < 0) {
                lastPoint.bias = 0;
            }

            // Schedule the slope changes (slope is going down)
            // We subtract new_user_slope from [newLocked.endTime]
            // and add old_user_slope to [oldLocked.endTime]
            if (oldLocked.endTime > block.timestamp) {


```
We can notice that ideal if-else statements combined and optimized and update the last point on end of the if-else statement.

Deployment gas saved

Before

| Contract  | Min  | Max  | Avg  |
|---|---|---|---|
| veOLAS   |  3044493 |  3044673  | 3044590  |

After 

| Contract  | Min  | Max  | Avg  |
|---|---|---|---|
| veOLAS   | 3037763  | 3037943  | 3037860  |


Code snippet:-
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L278C1-L293C37



## 2. Cache the global mapping in local variable to save the gas 

In below code snippet we can notice that `mapEpochTokenomics[curEpoch]` is a global mapping called 4 times for assigning boolean value through the check which could expensive in nature 

```solidity
        // Check all the unit fractions and identify those that need accounting of incentives
        bool[] memory incentiveFlags = new bool[](4);
        incentiveFlags[0] = (mapEpochTokenomics[curEpoch].unitPoints[0].rewardUnitFraction > 0);
        incentiveFlags[1] = (mapEpochTokenomics[curEpoch].unitPoints[1].rewardUnitFraction > 0);
        incentiveFlags[2] = (mapEpochTokenomics[curEpoch].unitPoints[0].topUpUnitFraction > 0);
        incentiveFlags[3] = (mapEpochTokenomics[curEpoch].unitPoints[1].topUpUnitFraction > 0);

```

Recommendation 
Our recommendation is to cache the global mapping `mapEpochTokenomics[curEpoch]` in local storage variable and calls the nested mapping through it which could be more optimizable than before.

```solidity

        TokenomicsPoint storage LocalEpoch = mapEpochTokenomics[curEpoch];
        // Check all the unit fractions and identify those that need accounting of incentives
        bool[] memory incentiveFlags = new bool[](4);
        incentiveFlags[0] = (LocalEpoch.unitPoints[0].rewardUnitFraction > 0);
        incentiveFlags[1] = (LocalEpoch.unitPoints[1].rewardUnitFraction > 0);
        incentiveFlags[2] = (LocalEpoch.unitPoints[0].topUpUnitFraction > 0);
        incentiveFlags[3] = (LocalEpoch.unitPoints[1].topUpUnitFraction > 0);


```

Code snippet:- 
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L692C1-L698C1


**Total gas Saved**

Before

| Contract  | Min  | Max  | Avg  |
|---|---|---|---|
| Tokenomics   |  -- |  -- | 5270217  |

| Contract  | Method  | Min  | Max  | Avg  |
|---|---|---|---|---|
| Tokenomics  | trackServiceDonations  | 67583  |  290686  | 154731 |




After

| Contract  | Min  | Max  | Avg  |
|---|---|---|---|
| Tokenomics   |  -- |  -- | 5266258  |

| Contract  | Method  | Min  | Max  | Avg  |
|---|---|---|---|---|
| Tokenomics  | trackServiceDonations  |   67420   |  290523  | 154568 |


**Comparison Table**

| Method  | Time (seconds)  |
|---|---|
| Using mapEpochTokenomics | 0.14195574000041233 |
| Using LocalEpoch	  | 0.12857630500002415 |

**Highlights of the Report**

1. This report consist of exact gas benchmark which could saved by our findings.

2. And comparison table.