# OLAS.inflationRemainder is against the design
File: https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L98-L114
According to [TokenomicsConstants.getSupplyCapForYear](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L30-L61), the first ten year's cap should be **supplyCaps**
```solidity
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

But in [OLAS.inflationRemainder](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L98-L113), the [supply](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L103C17-L103C26) is [tenYearSupplyCap](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L24), which is against the design
```solidity
22    uint256 public constant tenYearSupplyCap = 1_000_000_000e18;
```
```solidity
    function inflationRemainder() public view returns (uint256 remainder) {
        uint256 _totalSupply = totalSupply;
        // Current year
        uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
        // Calculate maximum mint amount to date
        uint256 supplyCap = tenYearSupplyCap;
        // After 10 years, adjust supplyCap according to the yearly inflation % set in maxMintCapFraction
        if (numYears > 9) {
            // Number of years after ten years have passed (including ongoing ones)
            numYears -= 9;
            for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
        }
        // Check for the requested mint overflow
        remainder = supplyCap - _totalSupply;
    }
```

