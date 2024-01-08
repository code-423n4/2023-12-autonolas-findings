# Gas Optimization

# Summary

| Number | Gas Optimization                                                                                          | Context |
| :----: | :-------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Pre-increment and pre-decrement are cheaper than +1 ,-1                                                   |   24    |
| [G-02] | <X> += <Y> costs more gas than  <X> = <X> + <Y> for state variables                                       |    3    |
| [G-03] | Make 3 Event parameters indexed when possible                                                             |   20    |
| [G-04] | Expressions for constant values such as a call to  KECCAK256(), should use IMMUTABLE rather than CONSTANT |    5    |
| [G-05] | abi.encode() is less efficient than abi.encodePacked()                                                    |    2    |
| [G-06] | Use selfbalance() instead of address(this).balance                                                        |    3    |
| [G-07] | Use Constants instead of type(uintx).max                                                                  |   13    |
| [G-08] | Shorten the array rather than copying to a new one                                                        |   26    |
| [G-09] | Functions Guaranteed to revert when called by normal users can be markd payable                           |   48    |
| [G-10] | Duplicated if() checks should be refactored to a modifier or function                                     |   24    |
| [G-11] | Use ASSEMBLY to perform efficient back-to-back calls                                                      |    9    |

## [G-01] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File:  registries/contracts/AgentRegistry.sol

41            if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L41

```solidity
File:  registries/contracts/ComponentRegistry.sol

30            if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L30

```solidity
File:  tokenomics/contracts/Depository.sol

234        productCounter = uint32(productId + 1);

334        bondCounter = uint32(bondId + 1);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L234

```solidity
File: registries/contracts/GenericRegistry.sol


73        return unitId > 0 && unitId < (totalSupply + 1);

98        unitId = id + 1;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#73

```solidity
File: tokenomics/contracts/Tokenomics.sol

551        emit TokenomicsParametersUpdateRequested(epochCounter + 1, _devsPerCapital, _codePerDev, _epsilonRate, _epochLen,

586        uint256 eCounter = epochCounter + 1;

970        TokenomicsPoint storage nextEpochPoint = mapEpochTokenomics[eCounter + 1];

975            emit IncentiveFractionsUpdated(eCounter + 1);

1002            emit TokenomicsParametersUpdated(eCounter + 1);

1067            epochCounter = uint32(eCounter + 1);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#551

```solidity
File: governance/contracts/veOLAS.sol

387        if (lockedBalance.endTime < (block.timestamp + 1)) {

441        if (unlockTime < (block.timestamp + 1)) {

469        if (lockedBalance.endTime < (block.timestamp + 1)) {

494        if (lockedBalance.endTime < (block.timestamp + 1)) {

498        if (unlockTime < (lockedBalance.endTime + 1)) {

562            if ((minPointNumber + 1) > maxPointNumber) {

565            uint256 mid = (minPointNumber + maxPointNumber + 1) / 2;

574            if (point.blockNumber < (blockNumber + 1)) {

626        if (uPoint.blockNumber < (blockNumber + 1)) {

655            PointVoting memory pointNext = mapSupplyPoints[minPointNumber + 1];

730        if (sPoint.blockNumber < (blockNumber + 1)) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L387

## [G-02] <X> += <Y> costs more gas than  <X> = <X> + <Y> for state variables

```solidity
File: tokenomics/contracts/Depository.sol

460                    payout += mapUserBonds[i].payout;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L460

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

189        totalLiquidity += positionLiquidity;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189

```solidity
File: tokenomics/contracts/Tokenomics.sol

1037        curMaxBond += effectiveBond;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037

## [G-03] Make 3 Event parameters indexed when possible

It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
File: tokenomics/contracts/Dispenser.sol

15    event IncentivesClaimed(address indexed owner, uint256 reward, uint256 topUp);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L15

```solidity
File: governance/contracts/multisigs/GuardCM.sol

90    event SetTargetSelectors(address[] indexed targets, bytes4[] indexed selectors, uint256[] chainIds, bool[] statuses);
91    event SetBridgeMediators(address[] indexed bridgeMediatorL1s, address[] indexed bridgeMediatorL2s, uint256[] chainIds);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L90

```solidity
File: tokenomics/contracts/Tokenomics.sol

123    event EpochLengthUpdated(uint256 epochLen);

124    event EffectiveBondUpdated(uint256 effectiveBond);

125    event IDFUpdated(uint256 idf);

126    event TokenomicsParametersUpdateRequested(uint256 indexed epochNumber, uint256 devsPerCapital, uint256 codePerDev,
        uint256 epsilonRate, uint256 epochLen, uint256 veOLASThreshold);

129    event IncentiveFractionsUpdateRequested(uint256 indexed epochNumber, uint256 rewardComponentFraction,
        uint256 rewardAgentFraction, uint256 maxBondFraction, uint256 topUpComponentFraction, uint256 topUpAgentFraction);


136    event EpochSettled(uint256 indexed epochCounter, uint256 treasuryRewards, uint256 accountRewards, uint256 accountTopUps);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L123

```solidity
File: tokenomics/contracts/Treasury.sol

44    event DepositTokenFromAccount(address indexed account, address indexed token, uint256 tokenAmount, uint256 olasAmount);

45    event DonateToServicesETH(address indexed sender, uint256[] serviceIds, uint256[] amounts, uint256 donation);

46    event Withdraw(address indexed token, address indexed to, uint256 tokenAmount);

50    event UpdateTreasuryBalances(uint256 ETHOwned, uint256 ETHFromServices);

53    event MinAcceptedETHUpdated(uint256 amount);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L44

```solidity
File: registries/contracts/UnitRegistry.sol


9    event CreateUnit(uint256 unitId, UnitType uType, bytes32 unitHash);

10    event UpdateUnitHash(uint256 unitId, UnitType uType, bytes32 unitHash);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L9

```solidity
File: governance/contracts/veOLAS.sol


94    event Deposit(address indexed account, uint256 amount, uint256 locktime, DepositType depositType, uint256 ts);

95    event Withdraw(address indexed account, uint256 amount, uint256 ts);

96    event Supply(uint256 previousSupply, uint256 currentSupply);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L94

## [G-04] Expressions for constant values such as a call to  KECCAK256(), should use IMMUTABLE rather than CONSTANT

```solidity
File: governance/contracts/multisigs/GuardCM.sol

97    bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));

99    bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));

101    bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));

103    bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));

105    bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L96-L105

## [G-05] abi.encode() is less efficient than abi.encodePacked()

```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol

97        bytes memory message = abi.encode(msg.sender, to, amount);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L97

```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol

93        bytes memory message = abi.encode(msg.sender, to, amount);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L93

## [G-06] Use selfbalance() instead of address(this).balance

```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

147            if (value > address(this).balance) {
148                revert InsufficientBalance(value, address(this).balance);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L147

```solidity
File: governance/contracts/HomeMediator.sol

147            if (value > address(this).balance) {
148                revert InsufficientBalance(value, address(this).balance);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L147

```solidity
File: tokenomics/contracts/Treasury.sol

112        ETHOwned = uint96(address(this).balance);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L112

## [G-07] Use Constants instead of type(uintx).max

```solidity
File: tokenomics/contracts/Depository.sol

195        if (priceLP > type(uint160).max) {

205        if (supply > type(uint96).max) {

216        if (maturity > type(uint32).max) {

311        if (maturity > type(uint32).max) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195

```solidity
File: tokenomics/contracts/GenericBondCalculator.sol

59        if (totalTokenValue > type(uint192).max) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

95        if (positionData.liquidity > type(uint64).max) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L95

```solidity
File: governance/contracts/OLAS.sol

131        if (spenderAllowance != type(uint256).max) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity
File: tokenomics/contracts/Treasury.sol

128        if (amount + ETHFromServices > type(uint96).max) {

194        if (_minAcceptedETH > type(uint96).max) {

291        if (donationETH + ETHOwned > type(uint96).max) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L128

```solidity
File: governance/contracts/veOLAS.sol

392        if (amount > type(uint96).max) {

449        if (amount > type(uint96).max) {

474        if (amount > type(uint96).max) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392

## [G-08] Shorten the array rather than copying to a new one

ASSEMBLY can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File:  tokenomics/contracts/Depository.sol

252        uint256[] memory ids = new uint256[](numProducts);

273        closedProductIds = new uint256[](numClosedProducts);

399        bool[] memory positions = new bool[](numProducts);

411        productIds = new uint256[](numSelectedProducts);

446        bool[] memory positions = new bool[](numBonds);

466        bondIds = new uint256[](numAccountBonds);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L252

```solidity
File: governance/contracts/multisigs/GuardCM.sol

348            targets = new address[](1);

349            callDatas = new bytes[](1);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L348

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

366        positionAddresses = new address[](numPositions);

367        positionAmounts = new uint64[](numPositions);

368        positionPdaAtas = new address[](numPositions);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L366

```solidity
File: tokenomics/contracts/Tokenomics.sol

689        address[] memory registries = new address[](2);

693        bool[] memory incentiveFlags = new bool[](4);

913        uint256[] memory incentives = new uint256[](7);

1099        address[] memory registries = new address[](2);

1103        uint256[] memory registriesSupply = new uint256[](2);

1109        uint256[] memory lastIds = new uint256[](2);

1169        address[] memory registries = new address[](2);

1173        uint256[] memory registriesSupply = new uint256[](2);

1179        uint256[] memory lastIds = new uint256[](2);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L689

```solidity
File: registries/contracts/UnitRegistry.sol


97            uint32[] memory addSubComponentIds = new uint32[](numSubComponents + 1);

205        uint32[] memory numComponents = new uint32[](numUnits);

207        uint32[][] memory components = new uint32[][](numUnits);

219        uint32[] memory allComponents = new uint32[](maxNumComponents);

211        uint32[] memory processedComponents = new uint32[](numUnits);

261        subComponentIds = new uint32[](counter);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L97

## [G-09] Functions Guaranteed to revert when called by normal users can be markd payable

```solidity
File: governance/contracts/BridgedERC20.sol

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
File: tokenomics/contracts/DonatorBlacklist.sol

36    function changeOwner(address newOwner) external {

56    function setDonatorsStatuses(address[] memory accounts, bool[] memory statuses) external returns (bool success) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L36

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

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L37

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

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#384

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

## [G-10] Duplicated if() checks should be refactored to a modifier or function

```solidity
File: contracts/bridges/BridgedERC20.sol

32        if (msg.sender != owner) {

50        if (msg.sender != owner) {

61        if (msg.sender != owner) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32

```solidity
File: tokenomics/contracts/Depository.sol

125        if (msg.sender != owner) {

145        if (msg.sender != owner) {

165        if (msg.sender != owner) {

185        if (msg.sender != owner) {

246        if (msg.sender != owner) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L125

```solidity
File: tokenomics/contracts/Dispenser.sol

48        if (msg.sender != owner) {

66        if (msg.sender != owner) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L48

```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol

80        if (!success) {
81            revert TransferFailed(childToken, address(this), to, amount);
82        }

103        if (!success) {
104            revert TransferFailed(childToken, msg.sender, address(this), amount);
105        }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L80

```solidity
File: registries/contracts/GenericManager.sol

22        if (msg.sender != owner) {

38        if (msg.sender != owner) {

49        if (msg.sender != owner) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L22

```solidity
File: governance/contracts/GuardCM.sol

155        if (msg.sender != owner) {

171        if (msg.sender != owner) {

448        if (msg.sender != owner) {

501        if (msg.sender != owner) {

562        if (msg.sender != owner) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155

```solidity
File: governance/contracts/OLAS.sol

44        if (msg.sender != owner) {

59        if (msg.sender != owner) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44

```solidity
File: tokenomics/contracts/Tokenomics.sol

386        if (msg.sender != owner) {

406        if (msg.sender != owner) {

425        if (msg.sender != owner) {

452        if (msg.sender != owner) {

476        if (msg.sender != owner) {

506        if (msg.sender != owner) {

571        if (msg.sender != owner) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L386

## [G-11] Use ASSEMBLY to perform efficient back-to-back calls

```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol

98        bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);

104        IERC20(rootToken).burn(amount);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98

```solidity
File: registries/contracts/RegistriesManager.sol

52            success = IRegistry(componentRegistry).updateHash(msg.sender, unitId, unitHash);

54            success = IRegistry(agentRegistry).updateHash(msg.sender, unitId, unitHash);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L52

```solidity
File:  governance/contracts/wveOLAS.sol

195        uint256 userNumPoints = IVEOLAS(ve).getNumUserPoints(account);

197            uPoint = IVEOLAS(ve).getUserPoint(account, idx);


265        uint256 numPoints = IVEOLAS(ve).totalNumPoints();

266        PointVoting memory sPoint = IVEOLAS(ve).mapSupplyPoints(numPoints);

269            vPower = IVEOLAS(ve).totalSupplyLockedAtT(ts);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L195