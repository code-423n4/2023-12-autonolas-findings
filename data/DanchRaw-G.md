
# _locked variable optimization

## tokenomics\contracts\Dispenser.sol and tokenomics\contracts\Treasury.sol

if (_locked > 1) replace with _locked == 2, more gas efficient
Remove the initialization of _locked in the constructor, it is not needed

## registries\contracts\UnitRegistry.sol

if (_locked > 1) replace with _locked == 2, more gas efficient

## tokenomics\contracts\Tokenomics.sol 

remove unused _locked variable and initialization


