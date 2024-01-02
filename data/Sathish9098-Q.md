# QA Report

##

## [L-1] A year is not always 365 days

On leap years, the number of days is 366, so calculations during those years will return the wrong value

```solidity
FILE: 2023-12-autonolas/governance/contracts/OLAS.sol

21:  // One year interval
22:    uint256 public constant oneYear = 1 days * 365;

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L21-L22

```solidity
FILE: 2023-12-autonolas/tokenomics/contracts/TokenomicsConstants.sol

// One year in seconds
    uint256 public constant ONE_YEAR = 1 days * 365;

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L15-L16

## Recommended Mitigation
