# Redunant Check
## related code: 
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L441-L451
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1062-L1064
## summary 
There is no need to check `treasuryRewards` is great than zero at `Treasury:rebalanceTreasury`. Because it is checked before calling at here
```javascript
    if (
            incentives[1] == 0 ||
            ITreasury(treasury).rebalanceTreasury(incentives[1])
        )
```
## Recommended Mitigation Steps
```diff
--if (treasuryRewards > 0) {
```
