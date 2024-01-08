## [G-01] Caching global variables is expensive than using the variable itself

Total Instances : (17)
```
  owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L36C7-L36C28

```
        minter = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L37

```
        timeLaunch = block.timestamp;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L38C1-L38C38

```
  dBlock = block.number - point.blockNumber;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L659C11-L659C55

```
 dt = block.timestamp - point.ts;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L660C12-L660C45

```
  owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L21C7-L21C28

```
 owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L26C8-L26C28

```
  owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L18C7-L18C28

```
 owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L108C8-L108C28

```
 uint256 maturity = block.timestamp + vesting;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L215C8-L215C54

```
  maturity = block.timestamp + product.vesting;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L309C7-L309C54

```
 owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L31C8-L31C28

```
    owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L32C5-L32C28

```
  owner = msg.sender;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L290C7-L290C28

```
  uint256 diffNumSeconds = block.timestamp - prevEpochTime;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L898C7-L898C66

```
  numYears = (block.timestamp + curEpochLen - timeLaunch) / ONE_YEAR;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1010C7-L1010C76

```
 inflationPerEpoch = (yearEndTime - block.timestamp) * curInflationPerSecond;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1017C12-L1017C89

## [G-02] x += y/x -= y costs more gas than x = x + y/x = x - y for state variables

Total Instances : (4)
```
       tStep += WEEK;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L237C14-L237C35

```
 tStep += WEEK;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L696C16-L696C31

```
   totalLiquidity += positionLiquidity;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L189C6-L189C45

```
  totalLiquidity -= amount;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L255C7-L255C34

## [G-03] Make for loop unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is
limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time
and gas to let the for loop overflow.

Total Instances : (38)
```
 for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L108C12-L110C14

```
  for (uint256 i = 0; i < 128; ++i) {
            if ((minPointNumber + 1) > maxPointNumber) {
                break;
            }
            uint256 mid = (minPointNumber + maxPointNumber + 1) / 2;

            // Choose the source of points
            if (account == address(0)) {
                point = mapSupplyPoints[mid];
            } else {
                point = mapUserPoints[account][mid];
            }

            if (point.blockNumber < (blockNumber + 1)) {
                minPointNumber = mid;
            } else {
                maxPointNumber = mid - 1;
            }
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L561C7-L579C10

```
  for (uint256 i = 0; i < data.length;) {
            address target;
            uint32 payloadLength;
            // solhint-disable-next-line no-inline-assembly
            assembly {
                // First 20 bytes is the address (160 bits)
                i := add(i, 20)
                target := mload(add(data, i))
                // Offset the data by 12 bytes of value (96 bits) and by 4 bytes of payload length (32 bits)
                i := add(i, 16)
                payloadLength := mload(add(data, i))
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L212C7-L224C1

```
 for (uint256 j = 0; j < payloadLength; ++j) {
                payload[j] = data[i + j];
            }
            // Offset the data by the payload number of bytes
            i += payloadLength;

            // Verify the scope of the data
            _verifyData(target, payload, chainId);
        }
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L212

```
 for (uint256 i = 0; i < payload.length; ++i) {
            payload[i] = data[i + 4];
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L340C8-L343C1

```
   for (uint i = 0; i < targets.length; ++i) {
            // Get the bridgeMediatorL2 and L2 chain Id, if any
            uint256 bridgeMediatorL2ChainId = mapBridgeMediatorL1L2ChainIds[targets[i]];
            // bridgeMediatorL2 occupies first 160 bits
            address bridgeMediatorL2 = address(uint160(bridgeMediatorL2ChainId));

            // Check if the data goes across the bridge
            if (bridgeMediatorL2 != address(0)) {
                // Get the chain Id
                // L2 chain Id occupies next 64 bits
                uint256 chainId = bridgeMediatorL2ChainId >> 160;

                // Process the bridge logic
                _processBridgeData(callDatas[i], bridgeMediatorL2, chainId);
            } else {
                // Verify the data right away as it is not the bridged one
                _verifyData(targets[i], callDatas[i], block.chainid);
            }
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L360C6-L378C10

```
  for (uint256 i = 0; i < dataLength;) {
            address target;
            uint96 value;
            uint32 payloadLength;
            // solhint-disable-next-line no-inline-assembly
            assembly {
                // First 20 bytes is the address (160 bits)
                i := add(i, 20)
                target := mload(add(data, i))
                // Offset the data by 12 bytes of value (96 bits)
                i := add(i, 12)
                value := mload(add(data, i))
                // Offset the data by 4 bytes of payload length (32 bits)
                i := add(i, 4)
                payloadLength := mload(add(data, i))
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L125C7-L140C14

```
  for (uint256 j = 0; j < payloadLength; ++j) {
                payload[j] = data[i + j];
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L153C11-L155C14

```
  for (uint256 i = 0; i < dataLength;) {
            address target;
            uint96 value;
            uint32 payloadLength;
            // solhint-disable-next-line no-inline-assembly
            assembly {
                // First 20 bytes is the address (160 bits)
                i := add(i, 20)
                target := mload(add(data, i))
                // Offset the data by 12 bytes of value (96 bits)
                i := add(i, 12)
                value := mload(add(data, i))
                // Offset the data by 4 bytes of payload length (32 bits)
                i := add(i, 4)
                payloadLength := mload(add(data, i))
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L125C7-L141C1

```
 for (uint256 j = 0; j < payloadLength; ++j) {
                payload[j] = data[i + j];
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L153C12-L155C14

```
  for (uint256 i = 0; i < numSubComponents; ++i) {
                addSubComponentIds[i] = subComponentIds[i];
            }
            // Adding self component Id
            addSubComponentIds[numSubComponents] = uint32(unitId);
            subComponentIds = addSubComponentIds;
        }
        mapSubComponents[unitId] = subComponentIds;

        // Set total supply to the unit Id number
        totalSupply = unitId;
        // Safe mint is needed since contracts can create units as well
        _safeMint(unitOwner, unitId);

        emit CreateUnit(unitId, unitType, unitHash);
        _locked = 1;
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L98C2-L114C6

```
 for (uint32 i = 0; i < numUnits; ++i) {
            // Get subcomponents for each unit Id based on the subcomponentsFromType
            components[i] = _getSubComponents(subcomponentsFromType, unitIds[i]);
            numComponents[i] = uint32(components[i].length);
            maxNumComponents += numComponents[i];
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L211C8-L216C10

```
  for (counter = 0; counter < maxNumComponents; ++counter) {
            // Index of a minimal component
            uint32 minIdxComponent;
            // Amount of components identified as the next minimal component number
            uint32 numComponentsCheck;
            uint32 tryMinComponent = type(uint32).max;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L227C7-L232C55

```
     for (uint32 i = 0; i < numUnits; ++i) {
                // Either get a component that has a higher id than the last one ore reach the end of the processed Ids
                for (; processedComponents[i] < numComponents[i]; ++processedComponents[i]) {
                    if (minComponent < components[i][processedComponents[i]]) {
                        // Out of those component Ids that are higher than the last one, pick the minimal one
                        if (components[i][processedComponents[i]] < tryMinComponent) {
                            tryMinComponent = components[i][processedComponents[i]];
                            minIdxComponent = i;
                        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L234C8-L242C26

```
for (uint32 i = 0; i < counter; ++i) {
            subComponentIds[i] = allComponents[i];
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L262C9-L264C10

```
  for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
            if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) {
                revert ComponentNotFound(dependencies[iDep]);
            }
            lastId = dependencies[iDep];
        }
    }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L40C7-L46C6

```
         for (uint256 i = 0; i < payloadLength; ++i) {
                payload[i] = data[i + DEFAULT_DATA_LENGTH];
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L113C4-L116C1

```
  for (uint256 i = 0; i < numOwners; ++i) {
            if (owners[i] != checkOwners[numOwners - i - 1]) {
                revert WrongOwner(owners[i]);
            }
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L139C7-L143C10

```
  for (uint256 i = 0; i < numClosedProducts; ++i) {
            closedProductIds[i] = ids[i];
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L274C7-L276C10

```
 for (uint256 i = 0; i < bondIds.length; ++i) {
            // Get the amount to pay and the maturity status
            uint256 pay = mapUserBonds[bondIds[i]].payout;
            bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L357C8-L361C1

```
   for (uint256 i = 0; i < numProducts; ++i) {
            // Product is always active if its supply is not zero, and inactive otherwise
            if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) {
                positions[i] = true;
                ++numSelectedProducts;
            }
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L402C6-L408C10

```
  for (uint256 i = 0; i < numProducts; ++i) {
            if (positions[i]) {
                productIds[numPos] = i;
                ++numPos;
            }
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L413C7-L418C10

```
   for (uint256 i = 0; i < numBonds; ++i) {
            // Check if the bond belongs to the account
            // If not and the address is zero, the bond was redeemed or never existed
            if (mapUserBonds[i].account == account) {
                // Check if requested bond is not matured but owned by the account address
                if (!matured ||
                    // Or if the requested bond is matured, i.e., the bond maturity timestamp passed
                    block.timestamp >= mapUserBonds[i].maturity)
                {
                    positions[i] = true;
                    ++numAccountBonds;
                    // The payout is always bigger than zero if the bond exists
                    payout += mapUserBonds[i].payout;
                }
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L448C6-L462C14

```
  for (uint256 i = 0; i < accounts.length; ++i) {
            // Check for the zero address
            if (accounts[i] == address(0)) {
                revert ZeroAddress();
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L67C7-L71C14

```
 for (uint256 i = 0; i < numServices; ++i) {
            // Check if the service owner or donator stakes enough OLAS for its components / agents to get a top-up
            // If both component and agent owner top-up fractions are zero, there is no need to call external contract
            // functions to check each service owner veOLAS balance
            bool topUpEligible;
            if (incentiveFlags[2] || incentiveFlags[3]) {
                address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
                    IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L702C8-L712C1

```
  for (uint256 unitType = 0; unitType < 2; ++unitType) {
                // Get the number and set of units in the service
                (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).
                    getUnitIdsOfService(IServiceRegistry.UnitType(unitType), serviceIds[i]);
                // Service has to be deployed at least once to be able to receive donations,
                // otherwise its components and agents are undefined
                if (numServiceUnits == 0) {
                    revert ServiceNeverDeployed(serviceIds[i]);
                }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L714C11-L722C18

```
  for (uint256 j = 0; j < numServiceUnits; ++j) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L728C19-L728C68

```
   for (uint256 j = 0; j < numServiceUnits; ++j) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L758C14-L758C64

```
    for (uint256 i = 0; i < numServices; ++i) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L808C5-L808C52

```
  for (uint256 i = 0; i < 2; ++i) {
            registriesSupply[i] = IToken(registries[i]).totalSupply();
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1104C7-L1106C10

```
     for (uint256 i = 0; i < unitIds.length; ++i) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1110C4-L1110C55

```
  for (uint256 i = 0; i < unitIds.length; ++i) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1132C7-L1132C55

```
  for (uint256 i = 0; i < 2; ++i) {
            registriesSupply[i] = IToken(registries[i]).totalSupply();
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1174C7-L1176C10

```
  for (uint256 i = 0; i < unitIds.length; ++i) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1180C7-L1180C55

```
  for (uint256 i = 0; i < unitIds.length; ++i) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1202C7-L1202C55

```
  for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L55C11-L57C14

```
 for (uint256 i = 1; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L92C12-L94C14

```
  for (uint32 i = 0; i < numPositions; ++i) {
            positionAddresses[i] = positionAccounts[firstAvailablePositionAccountIndex + i];
            positionAmounts[i] = mapPositionAccountLiquidity[positionAddresses[i]];
            positionPdaAtas[i] = mapPositionAccountPdaAta[positionAddresses[i]];
        }
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L369C7-L373C10


## [G-04] abi.encode() is less efficient than abi.encodePacked()

Total Instances : (8)
```
  (address homeMediator, bytes memory mediatorPayload, ) = abi.decode(payload, (address, bytes, uint256));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L278C11-L278C117

```
        (bytes memory l2Message) = abi.decode(bridgePayload, (bytes));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L297C5-L297C75

```
 (address fxGovernorTunnel, bytes memory l2Message) = abi.decode(payload, (address, bytes));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L323C12-L323C104

```
  (targets, , callDatas, , , ) =
            abi.decode(payload, (address[], uint256[], bytes[], bytes32, bytes32, uint256));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L355C11-L356C93
```
   (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L76C6-L76C103

```
 bytes memory message = abi.encode(msg.sender, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L97C8-L97C67

```
    (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L74C5-L74C103

```
  bytes memory message = abi.encode(msg.sender, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L93C7-L93C67


## [G-05] Pack struct variables
Reordering the structure of the PointVoting ensures that the struct variables are packed efficiently, minimizing storage costs.

Total Instances : (1)
```
struct PointVoting {
    int128 bias;
    int128 slope;
    uint64 ts;
    uint64 blockNumber;
    uint128 balance;
}
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L5C1-L11C2

## [GAS-06] Long revert strings

Total Instances : (4)
```
  revert("Wrong bridged token mint account");
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L154C11-L154C56

```
 revert("No liquidity on a provided token account");
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L224C12-L224C64

```
 revert("Wrong bridged token mint account");
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L244C12-L244C56

```
     revert ("Requested amount is too big for the total available liquidity");
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L342C8-L342C86

## [G-07] --maxPointNumber costs less gas compared to maxPointNumber -= 1
This one isn’t in a for-loop and isn’t covered by c4udit

Total Instances : (1)
```
 maxPointNumber -= 1;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L556C16-L556C37

## [G-08] Use constants for variables that don’t change (Save a storage SLOT: 2200 Gas)
The variable paused should be declared as a constant variable since it does not change, from the naming, it would seem the intention was to actually make it a constant.

Total Instances : (2)
```
  uint8 public paused = 1;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L127C3-L127C29

The variable _locked should be declared as a constant variable since it does not change, from the naming, it would seem the intention was to actually make it a constant.

```
 uint256 internal _locked = 1;
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L23C4-L23C34

## [G-09] Using assembly to revert with an error message
Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

Total Instances : (3)
```
  revert ZeroAddress();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L144C15-L144C38

```
          revert ZeroAddress();
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L38C3-L38C34

```
  revert("Wrong PDA header");
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L122C11-L122C40

## [G-10] Expressions for constant values such as a call to keccak256() ,
should use immutable rather than constant

Total Instances : (5)
```
    bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L97C2-L97C123

```
 bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L99C4-L99C140

```
 bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L101C4-L101C126

```
 bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L103C4-L103C120

```
 bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L105C4-L105C114