# L1: Silent fail of ETH transfer in `Treasury::withdrawToAccount` if `amountETHFromServices` is smaller than `accountRewards`
## related code: 
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L402-L410
## summary 
If a user claims his eth rewards and the rewards he wants to claim are for any reason bigger than the `amountETHFromServices`, the call would not transfer his ETH but set the claimable ETH rewards for the component/agent to 0 making it impossible to claim them at a later time.

To prevent this, the call should fail if the rewards a user wants to claim are bigger than the `amountETHFromServices`





# I1: Redunant Check
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
