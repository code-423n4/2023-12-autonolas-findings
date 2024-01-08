# Gas Optimization

## [G-01] Expensive operation inside a for loop

```solidity
File: tokenomics/contracts/Tokenomics.sol

714            for (uint256 unitType = 0; unitType < 2; ++unitType) {
                // Get the number and set of units in the service
                (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).
                    getUnitIdsOfService(IServiceRegistry.UnitType(unitType), serviceIds[i]);
                // Service has to be deployed at least once to be able to receive donations,
                // otherwise its components and agents are undefined
                if (numServiceUnits == 0) {
                    revert ServiceNeverDeployed(serviceIds[i]);
                }
                // Record amounts data only if at least one incentive unit fraction is not zero
                if (incentiveFlags[unitType] || incentiveFlags[unitType + 2]) {
                    // The amount has to be adjusted for the number of units in the service
                    uint96 amount = uint96(amounts[i] / numServiceUnits);
                    // Accumulate amounts for each unit Id
                    for (uint256 j = 0; j < numServiceUnits; ++j) {
                        // Get the last epoch number the incentives were accumulated for
                        uint256 lastEpoch = mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch;
                        // Check if there were no donations in previous epochs and set the current epoch
                        if (lastEpoch == 0) {
                            mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);
                        } else if (lastEpoch < curEpoch) {
                            // Finalize unit rewards and top-ups if there were pending ones from the previous epoch
                            // Pending incentives are getting finalized during the next epoch the component / agent
                            // receives donations. If this is not the case before claiming incentives, the finalization
                            // happens in the accountOwnerIncentives() where the incentives are issued
                            _finalizeIncentivesForUnitId(lastEpoch, unitType, serviceUnitIds[j]);
                            // Change the last epoch number
                            mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch = uint32(curEpoch);
                        }
                        // Sum the relative amounts for the corresponding components / agents
                        if (incentiveFlags[unitType]) {
                            mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;
                        }
                        // If eligible, add relative top-up weights in the form of donation amounts.
                        // These weights will represent the fraction of top-ups for each component / agent relative
                        // to the overall amount of top-ups that must be allocated
                        if (topUpEligible && incentiveFlags[unitType + 2]) {
                            mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;
                            mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;
                        }
                    }
                }

                // Record new units and new unit owners
                for (uint256 j = 0; j < numServiceUnits; ++j) {
                    // Check if the component / agent is used for the first time
                    if (!mapNewUnits[unitType][serviceUnitIds[j]]) {
                        mapNewUnits[unitType][serviceUnitIds[j]] = true;
                        mapEpochTokenomics[curEpoch].unitPoints[unitType].numNewUnits++;
                        // Check if the owner has introduced component / agent for the first time
                        // This is done together with the new unit check, otherwise it could be just a new unit owner
                        address unitOwner = IToken(registries[unitType]).ownerOf(serviceUnitIds[j]);
                        if (!mapNewOwners[unitOwner]) {
                            mapNewOwners[unitOwner] = true;
                            mapEpochTokenomics[curEpoch].epochPoint.numNewOwners++;
                        }
                    }
                }
            }












1202        for (uint256 i = 0; i < unitIds.length; ++i) {
            // Get the last epoch number the incentives were accumulated for
            uint256 lastEpoch = mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch;
            // Calculate rewards and top-ups if there were pending ones from the previous epoch
            if (lastEpoch > 0 && lastEpoch < curEpoch) {
                // Get the overall amount of unit rewards for the component's last epoch
                // reward = (pendingRelativeReward * rewardUnitFraction) / 100
                uint256 totalIncentives = mapUnitIncentives[unitTypes[i]][unitIds[i]].pendingRelativeReward;
                if (totalIncentives > 0) {
                    totalIncentives *= mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].rewardUnitFraction;
                    // Accumulate to the final reward for the last epoch
                    reward += totalIncentives / 100;
                }
                // Add the final top-up for the last epoch
                totalIncentives = mapUnitIncentives[unitTypes[i]][unitIds[i]].pendingRelativeTopUp;
                if (totalIncentives > 0) {
                    // Summation of all the unit top-ups and total amount of top-ups per epoch
                    // topUp = (pendingRelativeTopUp * totalTopUpsOLAS * topUpUnitFraction) / (100 * sumUnitTopUpsOLAS)
                    totalIncentives *= mapEpochTokenomics[lastEpoch].epochPoint.totalTopUpsOLAS;
                    totalIncentives *= mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].topUpUnitFraction;
                    uint256 sumUnitIncentives = uint256(mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].sumUnitTopUpsOLAS) * 100;
                    // Accumulate to the final top-up for the last epoch
                    topUp += totalIncentives / sumUnitIncentives;
                }
            }

            // Accumulate total rewards to finalized ones
            reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            // Accumulate total top-ups to finalized ones
            topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;
        }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L714

```solidity
File: registries/contracts/UnitRegistry.sol

227        for (counter = 0; counter < maxNumComponents; ++counter) {
            // Index of a minimal component
            uint32 minIdxComponent;
            // Amount of components identified as the next minimal component number
            uint32 numComponentsCheck;
            uint32 tryMinComponent = type(uint32).max;
            // Assemble an array of all first components from each component array
            for (uint32 i = 0; i < numUnits; ++i) {
                // Either get a component that has a higher id than the last one ore reach the end of the processed Ids
                for (; processedComponents[i] < numComponents[i]; ++processedComponents[i]) {
                    if (minComponent < components[i][processedComponents[i]]) {
                        // Out of those component Ids that are higher than the last one, pick the minimal one
                        if (components[i][processedComponents[i]] < tryMinComponent) {
                            tryMinComponent = components[i][processedComponents[i]];
                            minIdxComponent = i;
                        }
                        // If we found a minimal component Id, we increase the counter and break to start the search again
                        numComponentsCheck++;
                        break;
                    }
                }
            }
            minComponent = tryMinComponent;

            // If minimal component Id is greater than the last one, it should be added, otherwise we reached the end
            if (numComponentsCheck > 0) {
                allComponents[counter] = minComponent;
                processedComponents[minIdxComponent]++;
            } else {
                break;
            }
        }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L227

## [G-02] Make 3 Event parameters indexed when possible

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

## [G-03] Pre-increment and pre-decrement are cheaper than +1 ,-1

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

## [G-04] Use selfbalance() instead of address(this).balance

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

## [G-05] Use ASSEMBLY to perform efficient back-to-back calls

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

## [G-06] Use Constants instead of type(uintx).max

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

## [G-07] abi.encode() is less efficient than abi.encodePacked()

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

## [G-08] Don’t make variables public unless it is necessary to do so

```solidity
File:  tokenomics/contracts/Depository.sol


74    // Minimum bond vesting value
    uint256 public constant MIN_VESTING = 1 days;
    // Depository version number
    string public constant VERSION = "1.0.1";

    // Owner address
    address public owner;
    // Individual bond counter
    // We assume that the number of bonds will not be bigger than the number of seconds
    uint32 public bondCounter;
    // Bond product counter
    // We assume that the number of products will not be bigger than the number of seconds
    uint32 public productCounter;

    // OLAS token address
    address public immutable olas;
    // Tkenomics contract address
    address public tokenomics;
    // Treasury contract address
    address public treasury;
    // Bond Calculator contract address
    address public bondCalculator;

    // Mapping of bond Id => account bond instance
    mapping(uint256 => Bond) public mapUserBonds;
    // Mapping of product Id => bond product instance
    mapping(uint256 => Product) public mapBondProducts;

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L74

```solidity
File: tokenomics/contracts/Dispenser.sol


18    address public owner;
    // Reentrancy lock

22    // Tokenomics contract address
    address public tokenomics;
    // Treasury contract address
    address public treasury;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L18

```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol


53    uint256 public constant DEFAULT_DATA_LENGTH = 36;
    // FX child address on L2 that receives the message across the bridge from the root L1 network
    address public immutable fxChild;
    // Root governor address on L1 that is authorized to propagate the transaction execution across the bridge
    address public rootGovernor;

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L53

```solidity
File: tokenomics/contracts/GenericBondCalculator.sol

22    address public immutable olas;
    // Tokenomics contract address
    address public immutable tokenomics;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L22

```solidity
File: registries/contracts/GenericManager.sol

14    address public owner;
    // Pause switch
    bool public paused;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L14

```solidity
File: registries/contracts/GenericRegistry.sol

14    // Owner address
    address public owner;
    // Unit manager
    address public manager;
    // Base URI
    string public baseURI;
    // Unit counter
    uint256 public totalSupply;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L14

```solidity
File: registries/contracts/multisigs/GnosisSafeMultisig.sol

26    bytes4 public constant GNOSIS_SAFE_SETUP_SELECTOR = 0xb63e800d;
    // Default data size to be parsed and passed to the Gnosis Safe Factory without payload
    uint256 public constant DEFAULT_DATA_LENGTH = 144;
    // Gnosis Safe
    address payable public immutable gnosisSafe;
    // Gnosis Safe Factory
    address public immutable gnosisSafeProxyFactory;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol#L26

```solidity
File: governance/contracts/multisigs/GuardCM.sol

96    // schedule selector
    bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));
    // scheduleBatch selector
    bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));
    // requireToPassMessage selector (Gnosis chain)
    bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));
    // processMessageFromForeign selector (Gnosis chain)
    bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));
    // sendMessageToChild selector (Polygon)
    bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));
    // Initial check governance proposal Id
    // Calculated from the proposalHash function of the GovernorOLAS
    uint256 public governorCheckProposalId = 88250008686885504216650933897987879122244685460173810624866685274624741477673;
    // Minimum data length that is encoded for the schedule function,
    // plus at least 4 bytes or 32 bits for the selector from the payload
    uint256 public constant MIN_SCHEDULE_DATA_LENGTH = 260;
    // Minimum data length that contains at least a selector (4 bytes or 32 bits)
    uint256 public constant SELECTOR_DATA_LENGTH = 4;
    // Minimum payload length for message on Gnosis accounting for all required encoding and at least one selector
    uint256 public constant MIN_GNOSIS_PAYLOAD_LENGTH = 292;
    // Minimum payload length for message on Polygon accounting for all required encoding and at least one selector
    uint256 public constant MIN_POLYGON_PAYLOAD_LENGTH = 164;

    // Owner address
    address public immutable owner;
    // Multisig address
    address public immutable multisig;

    // Governor address
    address public governor;
    // Guard pausing possibility
    uint8 public paused = 1;

    // Mapping of (target address | bytes4 selector | uint64 chain Id) => enabled / disabled
    mapping(uint256 => bool) public mapAllowedTargetSelectorChainIds;
    // Mapping of bridge mediator address L1 => (bridge mediator L2 address | uint64 supported L2 chain Id)
    mapping(address => uint256) public mapBridgeMediatorL1L2ChainIds;

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L96

```solidity
File: governance/contracts/bridges/HomeMediator.sol


53    uint256 public constant DEFAULT_DATA_LENGTH = 36;
    // AMB Contract Proxy (Home) address on L2 that receives the message across the bridge from the foreign L1 network
    address public immutable AMBContractProxyHome;
    // Foreign governor address on L1 that is authorized to propagate the transaction execution across the bridge
    address public foreignGovernor;

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L53

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

24    address public constant orca = address"whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc";
    // Whirlpool (LP) pool address
    address public pool;
    // Current program owned PDA account address
    address public pdaProgram;
    // Bridged token mint address
    address public bridgedTokenMint;
    // PDA bridged token account address
    address public pdaBridgedTokenAccount;
    // PDA header for position account
    uint64 public pdaHeader = 0xd0f7407ae48fbcaa;
    // Program PDA seed
    bytes public constant pdaProgramSeed = "pdaProgram";
    // Program PDA bump
    bytes1 public pdaBump;
    int32 public constant minTickLowerIndex = -443632;
    int32 public constant maxTickLowerIndex = 443632;

    // Total number of token accounts (even those that hold no positions anymore)
    uint32 public numPositionAccounts;
    // First available account index in the set of accounts;
    uint32 public firstAvailablePositionAccountIndex;
    // Total liquidity in a lockbox
    uint64 public totalLiquidity;

    //
    mapping(address => uint64) public mapPositionAccountLiquidity;
    mapping(address => address) public mapPositionAccountPdaAta;
    address[type(uint32).max] public positionAccounts;

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L24

```solidity
File: governance/contracts/OLAS.sol

22    uint256 public constant oneYear = 1 days * 365;
    // Total supply cap for the first ten years (one billion OLAS tokens)
    uint256 public constant tenYearSupplyCap = 1_000_000_000e18;
    // Maximum annual inflation after first ten years
    uint256 public constant maxMintCapFraction = 2;
    // Initial timestamp of the token deployment
    uint256 public immutable timeLaunch;

    // Owner address
    address public owner;
    // Minter address
    address public minter;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L22

```solidity
File: tokenomics/contracts/Tokenomics.sol

139    // Owner address
    address public owner;
    // Max bond per epoch: calculated as a fraction from the OLAS inflation parameter
    // After 10 years, the OLAS inflation rate is 2% per year. It would take 220+ years to reach 2^96 - 1
    uint96 public maxBond;

    // OLAS token address
    address public olas;
    // Inflation amount per second
    uint96 public inflationPerSecond;

    // Treasury contract address
    address public treasury;
    // veOLAS threshold for top-ups
    // This number cannot be practically bigger than the number of OLAS tokens
    uint96 public veOLASThreshold;

    // Depository contract address
    address public depository;
    // effectiveBond = sum(MaxBond(e)) - sum(BondingProgram) over all epochs: accumulates leftovers from previous epochs
    // Effective bond is updated before the start of the next epoch such that the bonding limits are accounted for
    // This number cannot be practically bigger than the inflation remainder of OLAS
    uint96 public effectiveBond;

    // Dispenser contract address
    address public dispenser;
    // Number of units of useful code that can be built by a developer during one epoch
    // We assume this number will not be practically bigger than 4,722 of its integer-part (with 18 digits of fractional-part)
    uint72 public codePerDev;
    // Current year number
    // This number is enough for the next 255 years
    uint8 public currentYear;
    // Tokenomics parameters change request flag
    bytes1 public tokenomicsParametersUpdated;
    // Reentrancy lock
    uint8 internal _locked;

    // Component Registry
    address public componentRegistry;
    // Default epsilon rate that contributes to the interest rate: 10% or 0.1
    // We assume that for the IDF calculation epsilonRate must be lower than 17 (with 18 decimals)
    // (2^64 - 1) / 10^18 > 18, however IDF = 1 + epsilonRate, thus we limit epsilonRate by 17 with 18 decimals at most
    uint64 public epsilonRate;
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
    uint32 public epochCounter;
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
    uint72 public devsPerCapital;

    // Blacklist contract address
    address public donatorBlacklist;
    // Last donation block number to prevent the flash loan attack
    // This number cannot be practically bigger than the number of seconds
    uint32 public lastDonationBlockNumber;

    // Map of service Ids and their amounts in current epoch
    mapping(uint256 => uint256) public mapServiceAmounts;
    // Mapping of owner of component / agent address => reward amount (in ETH)
    mapping(address => uint256) public mapOwnerRewards;
    // Mapping of owner of component / agent address => top-up amount (in OLAS)
    mapping(address => uint256) public mapOwnerTopUps;
    // Mapping of epoch => tokenomics point
    mapping(uint256 => TokenomicsPoint) public mapEpochTokenomics;
    // Map of new component / agent Ids that contribute to protocol owned services
    mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;
    // Mapping of new owner of component / agent addresses that create them
    mapping(address => bool) public mapNewOwners;
    // Mapping of component / agent Id => incentive balances
    mapping(uint256 => mapping(uint256 => IncentiveBalances)) public mapUnitIncentives;

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L139

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

11    string public constant VERSION = "1.0.1";
    // Tokenomics proxy address slot
    // keccak256("PROXY_TOKENOMICS") = "0xbd5523e7c3b6a94aa0e3b24d1120addc2f95c7029e097b466b2bedc8d4b4362f"
    bytes32 public constant PROXY_TOKENOMICS = 0xbd5523e7c3b6a94aa0e3b24d1120addc2f95c7029e097b466b2bedc8d4b4362f;
    // One year in seconds
    uint256 public constant ONE_YEAR = 1 days * 365;
    // Minimum epoch length
    uint256 public constant MIN_EPOCH_LENGTH = 1 weeks;
    // Minimum fixed point tokenomics parameters
    uint256 public constant MIN_PARAM_VALUE = 1e14;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L11

```solidity
File: tokenomics/contracts/Treasury.sol

55    // A well-known representation of an ETH as address
    address public constant ETH_TOKEN_ADDRESS = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

    // Owner address
    address public owner;
    // ETH received from services
    // Even if the ETH inflation rate is 5% per year, it would take 130+ years to reach 2^96 - 1 of ETH total supply
    uint96 public ETHFromServices;

    // OLAS token address
    address public olas;
    // ETH owned by treasury
    // Even if the ETH inflation rate is 5% per year, it would take 130+ years to reach 2^96 - 1 of ETH total supply
    uint96 public ETHOwned;

    // Tkenomics contract address
    address public tokenomics;
    // Minimum accepted donation value
    uint96 public minAcceptedETH = 0.065 ether;

    // Depository contract address
    address public depository;
    // Contract pausing
    uint8 public paused = 1;
    // Reentrancy lock
    uint8 internal _locked;

    // Dispenser contract address
    address public dispenser;

    // Token address => token reserves
    mapping(address => uint256) public mapTokenReserves;
    // Token address => enabled / disabled status
    mapping(address => bool) public mapEnabledTokens;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L55

```solidity
File: governance/contracts/veOLAS.sol


105    uint8 public constant decimals = 18;

107    address public immutable token;

109    uint256 public supply;

111    mapping(address => LockedBalance) public mapLockedBalances;

114    uint256 public totalNumPoints;

116    mapping(uint256 => PointVoting) public mapSupplyPoints;

118    mapping(address => PointVoting[]) public mapUserPoints;

120    mapping(uint64 => int128) public mapSlopeChanges;

123    string public name;

125    string public symbol;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L105

```solidity
File: governance/contracts/wveOLAS.sol

132    address public immutable ve;

134    address public immutable token;

136    string public constant name = "Voting Escrow OLAS";

138    string public constant symbol = "veOLAS";

140    uint8 public constant decimals = 18;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L132

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

## [G-10] <X> += <Y> costs more gas than  <X> = <X> + <Y> for state variables

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

## [G-11] Duplicated if() checks should be refactored to a modifier or function

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

## [G-12] Expressions for constant values such as a call to  KECCAK256(), should use IMMUTABLE rather than CONSTANT

```solidity
File: governance/contracts/multisigs/GuardCM.sol

97    bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));

99    bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));

101    bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));

103    bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));

105    bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L96-L105

## [G-13] Shorten the array rather than copying to a new one

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