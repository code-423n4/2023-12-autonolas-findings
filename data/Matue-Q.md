# Insecure conversion from uint256 to smaller types
Multiple unsafe conversions were identified in the *changeTokenomicsParameters* function in `autonolas/tokenomics/contracts/Tokenomics.sol`, variables are entered into the function as uint256 and converted to smaller types (such as uint96 and uint72) to assign values to global variables, however, the events emitted after successful execution, they are issued with the values of uin256 without conversion, leading the owner to believe that different values were stored in the global variables.

## Links to affected code

```solidity
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
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L497-L553

The variables _devsPerCapital, _codePerDev, _epochLen and _veOLASThreshold are vulnerable to insecure conversion.

## Recommended Mitigation Steps
Change the type of input variables or convert values before emitting events.