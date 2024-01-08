# Codebase Optimization Report

## Table Of Contents

- [Codebase Optimization Report](#codebase-optimization-report)

  - [Table Of Contents](#table-of-contents)
  - [\[G-01\]Using immutable on variables that are only set in the constructor and never after (Save 10500 Gas)](#g-01using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after-save-10500-gas)
  - [\[G-02\]STATE VARIABLES can be packed into fewer STORAGE SLOTS](#g-02state-variables-can-be-packed-into-fewer-storage-slots)
    - [\[G-02-01\]We can save one slot by optimize the order of variable declaration (Save 2k gas)](#g-02-01we-can-save-one-slot-by-optimize-the-order-of-variable-declaration-save-2k-gas)
  - [\[G-03\]Avoid zero to non zero storage writes](#g-03avoid-zero-to-non-zero-storage-writes)
  - [\[G-04\]STATE VARIABLES which are not modified within functions should be set as constants or immutable for values set at deployment](#g-04state-variables-which-are-not-modified-within-functions-should-be-set-as-constants-or-immutable-for-values-set-at-deployment)
    - [\[G-04-01\]The `pdaHeader` variable is not modified in `liquidity_lockbox` contract in any function so should be set as constant to save 1 STORAGE SLOT](#g-04-01the-pdaheader-variable-is-not-modified-in-liquidity_lockbox-contract-in-any-function-so-should-be-set-as-constant-to-save-1-storage-slot)
    - [\[G-04-02\]The `_locked` variable is not modified in `GenericRegistry` contract in any function so should be set as constant to save 1 STORAGE SLOT](#g-04-02the-_locked-variable-is-not-modified-in-genericregistry-contract-in-any-function-so-should-be-set-as-constant-to-save-1-storage-slot)
  - [\[G-05\]Use custom errors rather than `REVERT` to save gas](#g-05use-custom-errors-rather-than-revert-to-save-gas)
  - [\[G-06\]Admin functions can be payable](#g-06admin-functions-can-be-payable)
  - [\[G-07\]`<X> += <Y>` costs more gas than  `<X> = <X> + <Y>` for state variables](#g-07<x>-+=-<y>+costs-more-gas-than-<x>-=-<x>-+-<y>-for-state-variables)
  - [\[G-08\]Refactor Event to avoid emitting empty data](#g-08refactor-event-to-avoid-emitting-empty-data)

**The bot did an amazing job on this audit,but still missed some findings**

## [G-01]Using immutable on variables that are only set in the constructor and never after (Save 10500 Gas)

**This instance were missed by the bot**

**Gas per instance: 2.1K**
**Total Instances: 5**
Total Gas Saved: `2.1 * 5 = 10500 Gas`

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

26    address public pool;

28    address public pdaProgram;

30    address public bridgedTokenMint;

32    address public pdaBridgedTokenAccount;

38    bytes1 public pdaBump;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L26C1-L32C43

```diff
-    address public pool;
+    address public immutable pool;
    // Current program owned PDA account address
-    address public pdaProgram;
+    address public immutable pdaProgram;
    // Bridged token mint address
-    address public bridgedTokenMint;
+    address public immutable bridgedTokenMint;
    // PDA bridged token account address
-    address public pdaBridgedTokenAccount;
+    address public immutable pdaBridgedTokenAccount;
```

## [G-02]STATE VARIABLES can be packed into fewer STORAGE SLOTS

### [G-02-01]We can save one slot by optimize the order of variable declaration (Save 2k gas)

This modification shifts `totalLiquidity` and `firstAvailablePositionAccountIndex` to be next to the `pool` variable in `slot1`, and `numPositionAccounts` and `pdaBump` next to the `pdaProgram` variable in `slot2` and we save one `STORAGE SLOT`.

```solidity
File:

26   address public pool;
27 // Current program owned PDA account address
28 address public pdaProgram;
29 // Bridged token mint address
30 address public bridgedTokenMint;
31 // PDA bridged token account address
32 address public pdaBridgedTokenAccount;
33 // PDA header for position account
34 uint64 public pdaHeader = 0xd0f7407ae48fbcaa;
35 // Program PDA seed
36 bytes public constant pdaProgramSeed = "pdaProgram";
37 // Program PDA bump
38 bytes1 public pdaBump;
39 int32 public constant minTickLowerIndex = -443632;
41 int32 public constant maxTickLowerIndex = 443632;
42 // Total number of token accounts (even those that hold no positions anymore)
43 uint32 public numPositionAccounts;
44 // First available account index in the set of accounts;
45 uint32 public firstAvailablePositionAccountIndex;
46 // Total liquidity in a lockbox
47 uint64 public totalLiquidity;
```

```diff
26   address public pool;
+  // Total liquidity in a lockbox
+    uint64 public totalLiquidity;
+  // First available account index in the set of accounts;
+    uint32 public firstAvailablePositionAccountIndex;
27 // Current program owned PDA account address
28 address public pdaProgram;
+  // Total number of token accounts (even those that hold no positions anymore)
+    uint32 public numPositionAccounts;
+  // Program PDA bump
+    bytes1 public pdaBump;
29 // Bridged token mint address
30 address public bridgedTokenMint;
31 // PDA bridged token account address
32 address public pdaBridgedTokenAccount;
33 // PDA header for position account
34 uint64 public pdaHeader = 0xd0f7407ae48fbcaa;
35 // Program PDA seed
36 bytes public constant pdaProgramSeed = "pdaProgram";
-37 // Program PDA bump
-38 bytes1 public pdaBump;
39 int32 public constant minTickLowerIndex = -443632;
41 int32 public constant maxTickLowerIndex = 443632;
-42 // Total number of token accounts (even those that hold no positions anymore)
-43 uint32 public numPositionAccounts;
-44 // First available account index in the set of accounts;
-45 uint32 public firstAvailablePositionAccountIndex;
-46 // Total liquidity in a lockbox
-47 uint64 public totalLiquidity;
```

## [G-03]Avoid zero to non zero storage writes

When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L291-L293

```solidity
File: tokenomics/contracts/Tokenomics.sol


//@audit  Default  value is assigned `zero`


142    // After 10 years, the OLAS inflation rate is 2% per year. It would take 220+ years to reach 2^96 - 1
143    uint96 public maxBond;


153    // This number cannot be practically bigger than the number of OLAS tokens
154    uint96 public veOLASThreshold;


160    // This number cannot be practically bigger than the inflation remainder of OLAS
161    uint96 public effectiveBond;

166    // We assume this number will not be practically bigger than 4,722 of its integer-part (with 18 digits of fractional-part)
167    uint72 public codePerDev;

173    // Reentrancy lock
174    uint8 internal _locked;

181    uint64 public epsilonRate;

195    // This number cannot be practically bigger than the number of blocks
196    uint32 public epochCounter;


207    // We assume this number will not be practically bigger than 4,722 of its integer-part (with 18 digits of fractional-part)
208    uint72 public devsPerCapital;






264    function initializeTokenomics(
265     address _olas,
266     address _treasury,
267     address _depository,
268     address _dispenser,
269     address _ve,
270     uint256 _epochLen,
271     address _componentRegistry,
272     address _agentRegistry,
273     address _serviceRegistry,
274        address _donatorBlacklist
275   ) external
276    {
277        // Check if the contract is already initialized
278        if (owner != address(0)) {
279            revert AlreadyInitialized();
280        }
281
282        // Check for at least one zero contract address
283        if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
284            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
285            _serviceRegistry == address(0)) {
286            revert ZeroAddress();
287        }
288
289        // Initialize storage variables
290        owner = msg.sender;
291        _locked = 1;                     >>>>>>>>> Gas_saving
292        epsilonRate = 1e17;
293        veOLASThreshold = 10_000e18;



337        epochCounter = 1;


        // Setting initial parameters and fractions
341        devsPerCapital = 1e18;

357        codePerDev = 1e18;


368        maxBond = uint96(_maxBond);
369        effectiveBond = uint96(_maxBond);
```

When declaring the variables `_locked`, `_locked`, `veOLASThreshold`, `epochCounter`, `devsPerCapital`, `codePerDev`, `maxBond` and `effectiveBond` the default value is applied since we do not assign anything, the default value 0.

We then call the function `initializeTokenomics()` and here we assign the value 1 for `_locked = 1;`, value 1e17 for `epsilonRate = 1e17;`, value 10_000e18 for `veOLASThreshold = 10_000e18;`, value 1 for `epochCounter = 1;` , value 1e18 for `devsPerCapital = 1e18;` , value 1e18 for `codePerDev = 1e18;`, value uint96(\_maxBond) for `maxBond = uint96(_maxBond);` and value uint96(\_maxBond) for `effectiveBond = uint96(_maxBond);` . This means our value is updated from 0 to `non zero`.

Since the `initializeTokenomics()` function will always be called first, we can initialize our variables `_locked`, `_locked`, `veOLASThreshold`, `epochCounter`, `devsPerCapital`, `codePerDev`, `maxBond` and `effectiveBond` with `1`, `1e17`, `10_000e18`, `1e18` and `uint96(_maxBond)` during declaration which should avoid the zero to non zero writes.

```diff
File: tokenomics/contracts/Tokenomics.sol

    // After 10 years, the OLAS inflation rate is 2% per year. It would take 220+ years to reach 2^96 - 1
-    uint96 public maxBond;
+    uint96 public maxBond = uint96(_maxBond);
    // OLAS token address
    address public olas;
    // Inflation amount per second
    uint96 public inflationPerSecond;

    // Treasury contract address
    address public treasury;
    // veOLAS threshold for top-ups
    // This number cannot be practically bigger than the number of OLAS tokens
-    uint96 public veOLASThreshold;
+    uint96 public veOLASThreshold = 10_000e18;

    // Depository contract address
    address public depository;
    // effectiveBond = sum(MaxBond(e)) - sum(BondingProgram) over all epochs: accumulates leftovers from previous epochs
    // Effective bond is updated before the start of the next epoch such that the bonding limits are accounted for
    // This number cannot be practically bigger than the inflation remainder of OLAS
-    uint96 public effectiveBond;
+    uint96 public effectiveBond = uint96(_maxBond);

    // Dispenser contract address
    address public dispenser;
    // Number of units of useful code that can be built by a developer during one epoch
    // We assume this number will not be practically bigger than 4,722 of its integer-part (with 18 digits of fractional-part)
-    uint72 public codePerDev;
+    uint72 public codePerDev = 1e18;
    // Current year number
    // This number is enough for the next 255 years
    uint8 public currentYear;
    // Tokenomics parameters change request flag
    bytes1 public tokenomicsParametersUpdated;
    // Reentrancy lock
-    uint8 internal _locked;
+    uint8 internal _locked = 1;

    // Component Registry
    address public componentRegistry;
    // Default epsilon rate that contributes to the interest rate: 10% or 0.1
    // We assume that for the IDF calculation epsilonRate must be lower than 17 (with 18 decimals)
    // (2^64 - 1) / 10^18 > 18, however IDF = 1 + epsilonRate, thus we limit epsilonRate by 17 with 18 decimals at most
-    uint64 public epsilonRate;
+    uint64 public epsilonRate = 1e17;

    // Epoch length in seconds
    // By design, the epoch length cannot be practically bigger than one year, or 31_536_000 seconds
    uint32 public epochLen;

    // Agent Registry
    address public agentRegistry;
    // veOLAS threshold for top-ups that will be set in the next epoch
    // This number cannot be practically bigger than the number of OLAS tokens
    uint96 public nextVeOLASThreshold;

    // Service Registry
    address public serviceRegistry;
    // Global epoch counter
    // This number cannot be practically bigger than the number of blocks
-    uint32 public epochCounter;
+    uint32 public epochCounter = 1;

    // Time launch of the OLAS contract
    // 2^32 - 1 gives 136+ years counted in seconds starting from the year 1970, which is safe until the year of 2106
    uint32 public timeLaunch;
    // Epoch length in seconds that will be set in the next epoch
    // By design, the epoch length cannot be practically bigger than one year, or 31_536_000 seconds
    uint32 public nextEpochLen;

    // Voting Escrow address
    address public ve;
    // Number of valuable devs that can be paid per units of capital per epoch in fixed point format
    // We assume this number will not be practically bigger than 4,722 of its integer-part (with 18 digits of fractional-part)
-    uint72 public devsPerCapital;
+    uint72 public devsPerCapital = 1e18;
```

## [G-04]STATE VARIABLES which are not modified within functions should be set as constants or immutable for values set at deployment

### [G-04-01]The `pdaHeader` variable is not modified in `liquidity_lockbox` contract in any function so should be set as constant to save 1 STORAGE SLOT

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

34    uint64 public pdaHeader = 0xd0f7407ae48fbcaa;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L34

### [G-04-02]The `_locked` variable is not modified in `GenericRegistry` contract in any function so should be set as constant to save 1 STORAGE SLOT

```solidity
File: registries/contracts/GenericRegistry.sol

23    uint256 internal _locked = 1;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L23

## [G-05]Use custom errors rather than REVERT() to save gas

Custom errors are cheaper than require or revert statements with strings because of how custom errors are handled. Solidity stores only the first 4 bytes of the hash of the error signature and returns only that. This means during reverting, only 4 bytes needs to be stored in memory. In the case of string messages in require statements, Solidity has to store(in memory) and revert with at least 64 bytes.

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

72            revert("Invalid bump");

96            revert("Liquidity overflow");

101            revert("Wrong pool address");

106            revert("Wrong NFT address");

111            revert("Wrong ticks");

116            revert("Wrong PDA owner");

122            revert("Wrong PDA header");

128            revert("Wrong position PDA");

149            revert("Wrong user position ATA");

154            revert("Wrong bridged token mint account");

160            revert("Wrong PDA position owner");

213            revert("Wrong liquidity token account");

218            revert("Wrong position ATA");

224            revert("No liquidity on a provided token account");

229            revert("Amount exceeds a position liquidity");

234            revert("Wrong PDA bridged token ATA");

239            revert("Pool address is incorrect");

244            revert("Wrong bridged token mint account");

342            revert ("Requested amount is too big for the total available liquidity");
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L72

## [G-06]Admin functions can be payable

We can make admin specific functions payable to save gas, because the compiler won’t be checking the callvalue of the function.

This will also make the contract smaller and cheaper to deploy as there will be fewer opcodes in the creation and runtime code.

so this functions because of this

```solidity
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
```

check admin functions that change the Owner so we can make these functions payable to `save gas`

```solidity
File: governance/contracts/bridges/BridgedERC20.sol

30    function changeOwner(address newOwner) external {

48    function mint(address account, uint256 amount) external {

59    function burn(uint256 amount) external {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L30

```solidity
File: tokenomics/contracts/Depository.sol


123    function changeOwner(address newOwner) external {

143    function changeManagers(address _tokenomics, address _treasury) external {

163    function changeBondCalculator(address _bondCalculator) external {

183    function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {

244    function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L123

```solidity
File: tokenomics/contracts/Dispenser.sol

46    function changeOwner(address newOwner) external {

64    function changeManagers(address _tokenomics, address _treasury) external {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L46

```solidity
File: tokenomics/contracts/interfaces/IDonatorBlacklist.sol

36    function changeOwner(address newOwner) external {

56    function setDonatorsStatuses(address[] memory accounts, bool[] memory statuses) external returns (bool success) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IDonatorBlacklist.sol#L36

```solidity
File: registries/contracts/GenericManager.sol

20    function changeOwner(address newOwner) external virtual {

36    function pause() external virtual {

47    function unpause() external virtual {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L20

```solidity
File: registries/contracts/GenericRegistry.sol

37    function changeOwner(address newOwner) external virtual {

54    function changeManager(address newManager) external virtual {

78    function setBaseURI(string memory bURI) external virtual {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#37

```solidity
File: governance/contracts/multisigs/GuardCM.sol

154    function changeGovernor(address newGovernor) exte8rnal {

170    function changeGovernorCheckProposalId(uint256 proposalId) external {

441    function setTargetSelectorChainIds(

495    function setBridgeMediatorChainIds(

539    function pause() external {

560    function unpause() external {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L154

```solidity
File: tokenomics/contracts/Tokenomics.sol

384    function changeTokenomicsImplementation(address implementation) external {

404    function changeOwner(address newOwner) external {

423    function changeManagers(address _treasury, address _depository, address _dispenser) external {

450    function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {

474    function changeDonatorBlacklist(address _donatorBlacklist) external {

497    function changeTokenomicsParameters(

562    function changeIncentiveFractions(

609    function reserveAmountForBondProgram(uint256 amount) external returns (bool success) {

630    function refundFromBondProgram(uint256 amount) external {

788    function trackServiceDonations(

1085    function accountOwnerIncentives(address account, uint256[] memory unitTypes, uint256[] memory unitIds) external
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L384

```solidity
File: tokenomics/contracts/Treasury.sol

137    function changeOwner(address newOwner) external {

156    function changeManagers(address _tokenomics, address _depository, address _dispenser) external {

182    function changeMinAcceptedETH(uint256 _minAcceptedETH) external {

212    function depositTokenForOLAS(address account, uint256 tokenAmount, address token, uint256 olasMintAmount) external {

313    function withdraw(address to, uint256 tokenAmount, address token) external returns (bool success) {

466    function drainServiceSlashedFunds() external returns (uint256 amount) {

487    function enableToken(address token) external {

507    function disableToken(address token) external {

531    function pause() external {

542    function unpause() external {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L137

```solidity
File: registries/contracts/UnitRegistry.sol

49    function create(address unitOwner, bytes32 unitHash, uint32[] memory dependencies)

121    function updateHash(address unitOwner, uint256 unitId, bytes32 unitHash) external virtual
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L49

## [G-07]`<X> += <Y>` costs more gas than  `<X> = <X> + <Y>` for state variables

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

189        totalLiquidity += positionLiquidity;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#189

```solidity
File: tokenomics/contracts/Tokenomics.sol

1037        curMaxBond += effectiveBond;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037

## [G-08]Refactor Event to avoid emitting empty data

```solidity
File: governance/contracts/multisigs/GuardCM.sol

568        emit GuardUnpaused();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L568

```solidity
File: tokenomics/contracts/Treasury.sol


538        emit PauseTreasury();

549        emit UnpauseTreasury();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L538
