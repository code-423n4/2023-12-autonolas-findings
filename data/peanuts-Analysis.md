### Summary and Overview

OLAS protocol has four parts, bonding, registries and voting and bridging. Users will find themselves integrating with voting and bonding more than other mechanisms.

For voting, the mechanism is similar to veCRV voting.
- Firstly, users have to acquire some OLAS tokens
- Users can create a lock and deposit OLAs tokens in exchange for voting power
- Voting power depends on the amount of OLAS tokens locked and the duration of the lock

For Bonding, users will deposit LP tokens (OLAS-X, where X can be another common token like ETH or DAI at the protocol's discretion) to get OLAS token. Owner will specify the bonding rubrics.
- The owner will enable a certain LP token for bonding
- Users will acquire the LP token (eg OLAS-ETH LP token), and deposit the LP token into the treasury contract
- The protocol function will calculate the conversion rate of the LP token to OLAS.
- Users will need to wait for MIN_VESTING period (1 days) for their bond to mature.
- Once the bond matures, users can withdraw their OLAS tokens from the Depository contract 

### Codebase quality analysis and Mechanism review

##### veOLAS.sol

| Contract | Function           | Explanation                                                | Comments                                                         |
| -------- | ------------------ | ---------------------------------------------------------- | ---------------------------------------------------------------- |
| veOLAS   | depositFor         | Deposit amount of OLAS and adds amount to the current lock | Lock must already be created, another person can help to deposit |
| veOLAS   | createLockFor      | Creates a lock, deposit amount of OLAs and sets unlockTime | Another person can help to create the lock                       |
| veOLAS   | increaseAmount     | Deposit amount of OLAS and adds amount to the current lock | Similar to depositFor, but only for msg.sender                   |
| veOLAS   | increaseUnlockTime | Increase the current unlock time for the current lock      | Only for msg.sender, cannot increase past MAXTIME                |
| veOLAS   | withdraw           | Withdraw all tokens for msg.sender                         | Must withdraw total balance, lock must expire                    |
| veOLAS   | \_balanceOfLocked  | Gets the voting power of an account at a particular time   | view function, used only in getVotes                             |

##### Mechanism Review and Recommendation

- User can create a lock to deposit OLAS. User also specifies the duration of the lock.
- More OLAS deposited, more voting power. The longer the lock duration, the more voting power
- Users can create a lock for others and also increase the amount deposited for others
- Only the owners of the lock can increase the duration of the lock. Lock duration has a max limit
- This contract is extremely similar to the veCRV contract, so I did a comparison between both contracts
- veCRV does not allow contracts to create locks, but veOLAS does
- veCRV uses block epoch, but veOLAS uses block.number

- Recommend restricting contracts to have voting power, only addresses, to prevent any potential griefing/attacks
- Recommend not to allow users to create lock for other users to prevent griefing attacks (locking 1 wei for 4 years for an account)

##### Treasury.sol

| Contract | Function                   | Explanation                                                  | Comments                                                                             |
| -------- | -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| Treasury | receive                    | Must be above 0.065 ether (which can be changed by owner)    | Owner controls minAcceptedETH, also function does not need to check for overflow     |
| Treasury | changeOwner                | Changes the owner                                            | onlyOwner, Checks for 0 address, has modifier, but no two-step address change        |
| Treasury | changeManagers             | Changes the managers                                         | onlyOwner, Checks for 0 address, has modifier, but no two-step address change        |
| Treasury | changeMinAcceptedETH       | Changes minimum accepted ETH (currently 0.065)               | onlyOwner, Check for zero value, but no upper limit                                  |
| Treasury | depositTokenForOLAS        | Depository gets LP tokens and mints Olas                     | onlyDepository, requires account to give allowance                                   |
| Treasury | depositServiceDonationsETH | Contract gets service donations, transfer not in function    | increases the ETHFromServices variable, uses reentrancy guard                        |
| Treasury | withdraw                   | Withdraws ETH or LP token from contract to specified address | onlyOwner, updates ETHOwned / mapTokenReserves[token]                                |
| Treasury | withdrawToAccount          | Withdraws ETH from contract to specified address             | onlyDispenser, checks pause, withdraws ETHFromServices, accTopUps can be 0           |
| Treasury | rebalanceTreasury          | Increases ETHOwned and decreases ETHFromServices             | onlyTokenomics                                                                       |
| Treasury | drainServiceSlashedFunds   | Slash funds from service registry                            | onlyOwner, function calls serviceRegistry contract                                   |
| Treasury | enableToken                | Enable LP token address to be used by contract               | onlyOwner, changes mapEnabledTokens[token] = true, useful in withdraw and deposit LP |
| Treasury | disableToken               | Disable LP token address to be used by contract              | onlyOwner, checks whether reserve of token is 0, change mapEnabledTokens to false    |
| Treasury | pause                      | Pauses the contract                                          | onlyOwner, used in rebalance and withdrawToAccount                                   |
| Treasury | unpause                    | Unpause the contract                                         | onlyOwner, used in rebalance and withdrawToAccount                                   |

##### Mechanism Review and Recommendation

- There are two types of ETH that is tracked, ETHOwned and ETHFromServices
- ETHOwned is withdrawn through `withdraw()` and ETHFromServices is withdrawn through `withdrawToAccount()`
- There is mapEnabledTokens[token] and mapTokenReserves[token]. Both points to the LP token. One of them is a boolean, one is the amount of reserves
- mapTokenReserves[token] is increased through `depositTokenForOLAS()` and decreased through `withdraw()`
- OLAS is minted through `depositTokenForOLAS()` and potentially `withdrawToAccount()`

- Recommend creating a modifier for msg.sender == owner since it is repeated many times in the code
- `changeOwner()` should have a two-step transfer
- `changeManagers()` should have a two-step transfer
- `changeMinAcceptedETH()` should have a upper limit (eg max minEth accepted is 0.5 ETH)
- pause and unpause mechanism only used for 2 functions, consider using for more functions

##### Depository.sol

| Contract   | Function             | Explanation                                      | Comments                                                                               |
| ---------- | -------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------- |
| Depository | changeOwner          | Change the owner                                 | onlyOwner, Checks for 0 address, has modifier, but no two-step address change          |
| Depository | changeManagers       | Changes the managers                             | onlyOwner, Checks for 0 address, has modifier, but no two-step address change          |
| Depository | changeBondCalculator | Changes the bond calculator contract             | onlyOwner, Checks for 0 address, has modifier, but no two-step address change          |
| Depository | create               | Create a bond                                    | onlyOwner, increases productCounter variable and saves mapBondProducts                 |
| Depository | close                | Close a bond, uses productId                     | onlyOwner, increases numClosedProducts, saves closedProductIds, delete mapBondProducts |
| Depository | deposit              | User calls this function to deposit LP tokens    | calls calculatePayoutOLAS, updates supply, creates mapUserBonds                        |
| Depository | redeem               | User calls this function to withdraw OLAS tokens | withdraws OLAS after maturity (which is block.timestamp + vesting time)                |
| Depository | getProducts          | Get active / inactive products                   | Checks how many LP tokens are created and active                                       |
| Depository | isActiveProduct      | Check if a productId is active                   | Check whether supply of the productId > 0                                              |
| Depository | getBonds             | Get bond info for an account                     | Can check only for matured bonds, or for both matured and not yet ready                |
| Depository | getBondStatus        | Get maturity and payout info for a bondId        | Payout can be zero, bond has been redeemed or never existed                            |
| Depository | getCurrentPriceLP    | Get the current price of the LP                  | Called from BondCalculator contract                                                    |

##### Mechanism Review and Recommendation

- MIN_VESTING is one day, cannot be changed
- productId refers to the bond, and the productId is monotonically increasing
- productCounter will never decrease, only increase
- `create()` refers to how much OLAS can be supplied for a given LP amount
- every time `create()` is called, a new Product struct is created and saved in the mapBondProducts setting
- For the Product struct, PriceLP refers to the price of the LP, vesting refers to the number of seconds (not end time), token refers to the LP token address, supply refers to the OLAs token amount
- In deposit(), payout is calculated with amount and priceLP using the calculatePayoutOLAS function, supply is decreased, mapUserBonds is created, LP token is deposited, Olas token is minted
- In deposit(), mapBondProducts is auto deleted if supply reaches 0.
- `getBonds()`, if set to true, then will get only matured bonds from an account
- `getBonds()`, if set to false, then will get all bonds from an account.
- `getBonds()` loops all bonds, may cause out of gas error

- Recommend making MIN_VESTING flexible, changed with owner's discretion
- Checking whether msg.sender == owner should be a modifier since it is repeated many times in the code
- productCounter only increasing will affect getProducts(), might reach out of gas error
- bondCounter only increasing will affect getBonds(), might reach out of gas error
- payout may truncate to zero with extremely small tokenAmount

##### Other Contracts

| Contract           | Function             | Explanation                     | Comments                                                                      |
| ------------------ | -------------------- | ------------------------------- | ----------------------------------------------------------------------------- |
| TokenomicsConstant | getSupplyCapForYear  | Get supply cap for year         | Checked using Remix, numYears == 1000 still works, pure function              |
| TokenomicsConstant | getInflationForYear  | Get inflation for year          | Checked using Remix, numYears == 1000 still works, pure function              |
| DonatorBlacklist   | changeOwner          | Change the owner                | onlyOwner, Checks for 0 address, has modifier, but no two-step address change |
| DonatorBlacklist   | setDonatorsStatuses  | Set blacklist status            | onlyOwner, used only in Tokenomics.trackServiceDonations()                    |
| DonatorBlacklist   | isDonatorBlacklisted | Check if donator is blacklisted | onlyOwner, used only in Tokenomics.trackServiceDonations()                    |
| Dispenser          | changeOwner          | Change the owner                | onlyOwner, Checks for 0 address, has modifier, but no two-step address change |
| Dispenser          | changeManagers       | Changes the managers            | onlyOwner, Checks for 0 address, has modifier, but no two-step address change |
| Dispenser          | claimOwnerIncentives | Claims incentives for owner     | calls accountOwnerIncentives, withdrawToAccount, affects only msg.sender      |

### Centralization risks

| Contract         | Role  | Risk | Explanation                                                                                         |
| ---------------- | ----- | ---- | --------------------------------------------------------------------------------------------------- |
| Depository       | Owner | High | Controls bonding of LP tokens and supply, changeOwner and changeManagers                            |
| Treasury         | Owner | High | Controls draining of services, LP token for bonding, minAcceptedETH, changeOwner and changeManagers |
| DonatorBlacklist | Owner | High | Controls donators statuses, changeOwner                                                             |
| Tokenomics       | Owner | High | Controls changeTokenomicsParameters, changeOnwer, changeManagers, changeRegistries                  |

- The owner controls most of the flow of the protocol.
- Owner can set all addresses, have capabilities like changeOwner and changeManager
- Owner also controls the amount of supply for a given LP bond, and controls the tokenomicsParameters
- Owner cannot withdraw funds from the user, which is a good thing, but since the owner controls the protocol, the protocol still have a high centralization risk

- Centralization Risk - High

### Architecture Review

##### `Design Patterns and Best Practices:`

There is good usage of established design patterns, like pausing mechanism and reentrancy guard. There is also consistent checks for zero values and overflows. This shows that the protocol is well versed in security.

One good example is the usage of allowance check in Treasury.sol. Even though the allowance check is not needed because transferFrom inherently check for allowance, the function still checks because Uniswap allowance implementation does not revert with the accurate message. This shows that the protocol understands how external functionality works well.

##### `Code Readability and Maintainability:`

Functions are pretty difficult to understand, but the NATSPECs does help in facilitating understanding. Code can sometimes be repetitive, such as the repeated checking of owner. Some overflow checks are also redundant because of 0.8.0 where overflows will automatically be reverted.

##### `Error Handling and Input Validation:`

Events and Error messages are easy to understand. Error messages are also written neatly at the top of the contract, with sufficient NATSPEC explaining the reasoning for the error.

##### `Interoperability and Standards Compliance:`

Good knowledge of the ERC20 standard when creating the OLAS tokens. This is seen from protocol checking for success value after transferFrom, and creating a soft max cap with 2% inflation rate for their token. 

##### `Testing and Documentation:`

Extensive tests (unit, integration tests) done, reaching an overall coverage of almost 100%. Documentation (whitepaper, github) is plentiful and includes quality reasoning at every juncture of the protocol. **One improvement could be having more diagrams and a summarized version of how the protocol works from the top down, with a end-to-end scenario of how a user can interact with the protocol  (what function should the user call first, what can a user accomplish, why is the user incentivized to use the protocol)**

##### `Upgradeability:`

Most of the protocol is not intended to be upgradeable, but contracts can be redeployed and rerouted easily through the changeManagers and changeRegistries function.

##### `Dependency Management:`

Protocol relies on external libraries like OpenZeppelin, and also uses code from other contracts (veCRV, FXPortal, Arbitrary Message Bridge). If those code are not well secured, it will affect the protocol. Protocol should keep an eye on vulnerabilities that affects those external integrations, and make changes where necessary.

Overall, great architecture from the protocol, slight changes would be to the written code itself (using modifiers for repeated code, checking zero values, checking overflows etc) and more explicit documentation

### Comments and Context

- Due to time constraints, the audit only focused on the Bonding and Voting Escrow portion of the protocol. The contracts being focused on are `OLAS.sol`, `veOLAS.sol`, `wveOLAS.sol`, `GovernorOLAS.sol`, `Depository.sol`, `Dispenser.sol`, `DonatorBlacklist.sol`, `GenericBondCalculator.sol`, `Tokenomics.sol`, `TokenomicsConstants.sol`, `TokenomicsProxy.sol`, `Treasury.sol`. For the rest of the contracts, I have looked only looked briefly just to contextualize the whole protocol.

### Time spent:
025 hours