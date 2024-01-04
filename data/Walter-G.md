## FxGovernorTunnel
1) ### Reduce variable size
Line 153 (https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxGovernorTunnel.sol#L153 ),125 (https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxGovernorTunnel.sol#L125 ) could be rewritten as an uint32
## TokenomicsConstants
1) line 59 (https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L59 ) should remove the return statement saving a bit of gas doing so
## Tokenomics
1) if statement at line 732 (https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L732 ) could be written without the curly brackets {} saving few gas in this way
## Treasury
1) zone around line 443 (https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L443 ) should be written like so:

```
            if (amountETHFromServices >= treasuryRewards) {

                // Update ETH from services value

                amountETHFromServices -= treasuryRewards;

                // Update treasury ETH owned values

                ETHOwned += uint96(treasuryRewards);

                ETHFromServices = uint96(amountETHFromServices);

                emit UpdateTreasuryBalances(ETHOwned, amountETHFromServices);

            } else {

                // There is not enough amount from services to allocate to the treasury

                success = false;

            }
```
And by doing so users will save a bit of gas every call of the function


