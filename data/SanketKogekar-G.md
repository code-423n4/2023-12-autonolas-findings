- To save caller's gas on calling `mint()`, simply add 2_000_000_0e18 to `supplyCap` on each iteration. Since both `supplyCap` (tenYearSupplyCap) and maxMintCapFraction are constants, we are always sure about following calculation to take place: `supplyCap += (1_000_000_000e18 * 2) / 100;` i.e. `supplyCap += 2_000_000_0e18`

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L108C1-L110C14

- In following calculations, slope and bias will always be 0 if either of these condition is true `(int128(oldLocked.amount) < IMAXTIME)` OR `(int128(newLocked.amount) < IMAXTIME)`. In case that is expected: both `slope` and `bias` could be set to 0 directly instead of calculating for each call of function `depositFor()`

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L188C1-L195C14