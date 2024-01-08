
##  Gas Summary

| Number | Issue | Instances|
|--------|-------|----------|
|[G-01]| IF’s/require() statements that check input arguments should be at the top of the function  | 4 |
|[G-02]| Save loop calls  | 9 |
|[G-03]| Use hardcode address instead address(this)  | 31 |
|[G-04]| Using assembly to revert with an error message  | 91 |
|[G-05]| Use assembly to reuse memory space when making more than one external call  | 27 |
|[G-06]| Shorten arrays with inline assembly  | 10 |
|[G-07]| Use the existing Local variable/global variable when equal to a state variable to avoid reading from state  | 4 |
|[G-08]| Don’t make variables public unless necessary  | 74 |
|[G-09]| Assigning keccak operations to constant variables results in extra gas costs  | 5 |
|[G-10]| Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4  | 1 |
|[G-11]| Create immutable variable to avoid an external call  | 1 |
|[G-12]| Don’t cache state varible if  that are only used once  | 2 |
|[G-13]| Splitting if() statements that use && saves gas  | 15 |
|[G-14]| public functions not called by the contract should be declared external instead  | 10 |
|[G-15]| Pack struct variables  | 1 |
|[G-16]| Using storage instead of memory for structs/arrays saves gas   | 8 |
|[G-17]| When using storage instead of memory, we should cache any fields that need to be re-read in stack variables  | 2 |
|[G-18]| Use assembly for loops  | 22 |
|[G-19]| Use assembly in place of abi.decode to extract calldata values more efficiently  | 7 |
|[G-20]| Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient  | 9 |
|[G-21]| Use constants instead of type(uintX).max  | 27 |
|[G-22]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 14 |
|[G-23]| Unused Named Return Variables  | 57 |
|[G-24]| Using delete statement can save gas  | 2 |
|[G-25]| Default value initialization  | 59 |
|[G-26]| Don’t cache call if used only once  | 8 |
|[G-27]| State variables which are not modified within functions should be set as constant or immutable for values set at deployment  | 3 |


## [G-01] IF’s/require() statements that check input arguments should be at the top of the function

```solidity
file: main/governance/contracts/veOLAS.sol

379   if (amount == 0) {
            revert ZeroValue();
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L379-L381


```solidity
file: main/governance/contracts/veOLAS.sol

461   if (amount == 0) {
            revert ZeroValue();
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L461-L463


```solidity
file:  main/governance/contracts/veOLAS.sol

474   if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474-L476


```solidity
file:  main/governance/contracts/veOLAS.sol

502    if (unlockTime > block.timestamp + MAXTIME) {
            revert MaxUnlockTimeReached(msg.sender, block.timestamp + MAXTIME, unlockTime);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L502-L504

## [G-02] Save loop calls

Instead of calling mapUserBonds[bondIds[i]] 6 times in each loop for fetching data, it can be saved as a variable.


```solidity
file:  main/tokenomics/contracts/Depository.sol

357    for (uint256 i = 0; i < bondIds.length; ++i) {
            // Get the amount to pay and the maturity status
            uint256 pay = mapUserBonds[bondIds[i]].payout;
            bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;

            // Revert if the bond does not exist or is not matured yet
            if (pay == 0 || !matured) {
                revert BondNotRedeemable(bondIds[i]);
            }

            // Check that the msg.sender is the owner of the bond
            if (mapUserBonds[bondIds[i]].account != msg.sender) {
                revert OwnerOnly(msg.sender, mapUserBonds[bondIds[i]].account);
            }

            // Increase the payout
            payout += pay;

            // Get the productId
            uint256 productId = mapUserBonds[bondIds[i]].productId;

            // Delete the Bond struct and release the gas
            delete mapUserBonds[bondIds[i]];
            emit RedeemBond(productId, msg.sender, bondIds[i]);
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L375-L381

Instead of calling mapUserBonds[i] 3 times in each loop for fetching data, it can be saved as a variable.

```solidity
file:  main/tokenomics/contracts/Depository.sol

448    for (uint256 i = 0; i < numBonds; ++i) {
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
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L448-L462

Instead of calling unitPoints[i] 4 times in each loop for fetching data, it can be saved as a variable.

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

978   for (uint256 i = 0; i < 2; ++i) {
                nextEpochPoint.unitPoints[i].topUpUnitFraction = tp.unitPoints[i].topUpUnitFraction;
                nextEpochPoint.unitPoints[i].rewardUnitFraction = tp.unitPoints[i].rewardUnitFraction;
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L978-L981

Instead of calling serviceUnitIds[j] 3 times and mapNewOwners[unitOwner] 2 time in each loop for fetching data, it can be saved as a variable.

```sooidity
file: main/tokenomics/contracts/Tokenomics.sol

758   for (uint256 j = 0; j < numServiceUnits; ++j) {
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

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L785-L771

Instead of calling unitTypes[i] 6 times and unitIds[i] 3 time in each loop for fetching data, it can be saved as a variable.

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

1202    for (uint256 i = 0; i < unitIds.length; ++i) {
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

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1202-L1226


Instead of calling mapNewOwners[unitOwner] 2 time in each loop for fetching data, it can be saved as a variable.

```sooidity
file: main/tokenomics/contracts/Tokenomics.sol

758   for (uint256 j = 0; j < numServiceUnits; ++j) {
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

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L785-L771


Instead of calling mapUnitIncentives[unitType] 5 time in each loop for fetching data, it can be saved as a variable.

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

714    for (uint256 unitType = 0; unitType < 2; ++unitType) {
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
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L714-L755

Instead of calling mapUnitIncentives[unitTypes[i] 6 time in each loop for fetching data, it can be saved as a variable.

```solidity
file: main/tokenomics/contracts/Tokenomics.sol

1132   for (uint256 i = 0; i < unitIds.length; ++i) {
            // Get the last epoch number the incentives were accumulated for
            uint256 lastEpoch = mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch;
            // Finalize unit rewards and top-ups if there were pending ones from the previous epoch
            // The finalization is needed when the trackServiceDonations() function did not take care of it
            // since between last epoch the donations were received and this current epoch there were no more donations
            if (lastEpoch > 0 && lastEpoch < curEpoch) {
                _finalizeIncentivesForUnitId(lastEpoch, unitTypes[i], unitIds[i]);
                // Change the last epoch number
                mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch = 0;
            }

            // Accumulate total rewards and clear their balances
            reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;
            mapUnitIncentives[unitTypes[i]][unitIds[i]].reward = 0;
            // Accumulate total top-ups and clear their balances
            topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;
            mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp = 0;
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1132-L1150

Instead of calling mapUnitIncentives[unitTypes[i]][unitIds[i]] 5 time in each loop for fetching data, it can be saved as a variable.

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

1202    for (uint256 i = 0; i < unitIds.length; ++i) {
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
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1202-L1232

## [G-03] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References:

https://book.getfoundry.sh/reference/forge-std/compute-create-address https://twitter.com/transmissions11/status/1518507047943245824


```solidity
file: main/governance/contracts/veOLAS.sol

364   IERC20(token).transferFrom(msg.sender, address(this), amount);

768   revert NonTransferable(address(this));

773   revert NonTransferable(address(this));

778   revert NonTransferable(address(this));

784   revert NonTransferable(address(this));

790   revert NonDelegatable(address(this));

796   revert NonDelegatable(address(this));

803   revert NonDelegatable(address(this));

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364  

```solidity
file: main/governance/contracts/bridges/FxERC20ChildTunnel.sol

81   revert TransferFailed(childToken, address(this), to, amount);

102  bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

104  revert TransferFailed(childToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L81


```solidity
file: main/governance/contracts/bridges/FxERC20RootTunnel.sol

98   bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);

100  revert TransferFailed(rootToken, msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98

```solidity
file: main/governance/contracts/bridges/FxERC20RootTunnel.sol

84   if (msg.sender != address(this)) {

85   revert SelfCallOnly(msg.sender, address(this));

147  if (value > address(this).balance) {

148  revert InsufficientBalance(value, address(this).balance);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L84

```solidity
file: main/governance/contracts/bridges/HomeMediator.sol

84  if (msg.sender != address(this)) {

85   revert SelfCallOnly(msg.sender, address(this));

147  if (value > address(this).balance) {

148   revert InsufficientBalance(value, address(this).balance);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84

```solidity
file: blob/main/tokenomics/contracts/Treasury.sol

112    ETHOwned = uint96(address(this).balance);

228   if (IToken(token).allowance(account, address(this)) < tokenAmount) {

229   revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);

235   bool success = IToken(token).transferFrom(account, address(this), tokenAmount);

237    revert TransferFailed(token, account, address(this), tokenAmount)

320    if (to == address(this)) {

321    revert TransferFailed(token, address(this), to, tokenAmount);

341   revert TransferFailed(ETH_TOKEN_ADDRESS, address(this), to, tokenAmount);

364   revert TransferFailed(token, address(this), to, tokenAmount);

408    revert TransferFailed(address(0), address(this), account, accountRewards);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L112


## [G-04] Using assembly to revert with an error message

Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s an example:

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}
/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```
From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

Code Snippets:

```solidity
file: main/lockbox-solana/solidity/liquidity_lockbox.sol
 
72    revert("Invalid bump");

96    revert("Liquidity overflow");

101   revert("Wrong pool address");

106   revert("Wrong NFT address");

111   revert("Wrong ticks");

116   revert("Wrong PDA owner");

122   revert("Wrong PDA header");

128   revert("Wrong position PDA");

149   revert("Wrong user position ATA");

154   revert("Wrong bridged token mint account");

160   revert("Wrong PDA position owner");

213   revert("Wrong liquidity token account");

218   revert("Wrong position ATA");

224   revert("No liquidity on a provided token account");

229   revert("Amount exceeds a position liquidity");

234   revert("Wrong PDA bridged token ATA");

239   revert("Pool address is incorrect");

244   revert("Wrong bridged token mint account");

324   revert ("Requested amount is too big for the total available liquidity");

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L72


```solidity
file: main/tokenomics/contracts/TokenomicsProxy.sol
 
36    revert ZeroTokenomicsAddress();

41    revert ZeroTokenomicsData();

50    revert InitializationFailed();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsProxy.sol#L36

```solidity
file: main/governance/contracts/bridges/FxERC20ChildTunnel.sol

40   revert ZeroAddress();

60   revert ZeroAddress();

93   revert ZeroValue();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L40



```solidity
file: main/governance/contracts/bridges/FxERC20RootTunnel.sol

44   revert ZeroAddress();

64   revert ZeroAddress();

89   revert ZeroValue();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L44

```solidity
file: main/governance/contracts/bridges/FxGovernorTunnel.sol

65    revert ZeroAddress();

90    revert ZeroAddress();

144   revert ZeroAddress();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L65

```solidity
file: main/registries/contracts/GenericRegistry.sol

45   revert ZeroAddress();

61   revert ZeroAddress();

86   revert ZeroValue();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L45


```solidity
file: main/registries/contracts/UnitRegistry.sol

54   revert ReentrancyGuard();

65   revert ZeroAddress();

70   revert ZeroValue();

140  revert ZeroValue();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L54

```solidity
file: main/governance/contracts/multisigs/GuardCM.sol

145   revert ZeroAddress();

161   revert ZeroAddress();

177   revert ZeroValue();

227   revert ZeroAddress();

406   revert NoDelegateCall();

429   revert NoSelfCall();

461   revert ZeroAddress();

466   revert ZeroValue();

471   revert ZeroValue();

514   revert ZeroAddress();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L145

```solidity
file: main/governance/contracts/bridges/HomeMediator.sol

65     revert ZeroAddress();

90    revert ZeroAddress();

144   revert ZeroAddress();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L65

```solidity
file: main/governance/contracts/wveOLAS.sol

148    revert ZeroAddress();

271    revert WrongTimestamp(sPoint.ts, ts);

298    revert NonTransferable(ve);

303    revert NonTransferable(ve);

308    revert NonTransferable(ve);

313    revert NonTransferable(ve);

318    revert NonDelegatable(ve);

324    revert NonDelegatable(ve);

330    revert NonDelegatable(ve);

335    revert NonDelegatable(ve);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L148


```solidity
file: main/governance/contracts/bridges/BridgedERC20.sol

33   revert OwnerOnly(msg.sender, owner);

38   revert ZeroAddress();

51   revert OwnerOnly(msg.sender, owner);

62   revert OwnerOnly(msg.sender, owner);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L33

```solidity
file: main/governance/contracts/veOLAS.sol

380   revert ZeroValue();

384   revert NoValueLocked(account);

388   revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);

450   revert Overflow(amount, type(uint96).max);

462   revert ZeroValue();

466   revert NoValueLocked(msg.sender);

470   revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);

475   revert Overflow(amount, type(uint96).max);

491   revert NoValueLocked(msg.sender);

495   revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);

499   revert UnlockTimeIncorrect(msg.sender, lockedBalance.endTime, unlockTime);

503   revert MaxUnlockTimeReached(msg.sender, block.timestamp + MAXTIME, unlockTime);

513   revert LockNotExpired(msg.sender, lockedBalance.endTime, block.timestamp);

646   revert WrongBlockNumber(blockNumber, block.number);

768   revert NonTransferable(address(this));

773   revert NonTransferable(address(this));

778   revert NonTransferable(address(this));

784   revert NonTransferable(address(this));

790   revert NonDelegatable(address(this));

796   revert NonDelegatable(address(this));

803   revert NonDelegatable(address(this));

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L380

```solidity
file: main/governance/contracts/OLAS.sol

45    revert ManagerOnly(msg.sender, owner);

49    revert ZeroAddress();

60    revert ManagerOnly(msg.sender, owner);

64    revert ZeroAddress();

78    revert ManagerOnly(msg.sender, minter);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L45


## [G-05] Use assembly to reuse memory space when making more than one external call

Issue Description - When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not clear or reuse memory, so it expands memory each time.

Proposed Optimization - Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments without expanding memory further.

Estimated Gas Savings - Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000 gas. With larger arguments or return data, the savings would be even greater.


```solidity
See the example below;
contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}
contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);
        uint256 res = res1 + res2;
        return res;
    }
}
contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }
            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)
            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)
            // add results
            let res := add(res1, res2)
            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and its arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

Code Snippets:


```solidity
file: main/registries/contracts/RegistriesManager.sol

39    unitId = IRegistry(componentRegistry).create(unitOwner, unitHash, dependencies);

41    unitId = IRegistry(agentRegistry).create(unitOwner, unitHash, dependencies);

52    success = IRegistry(componentRegistry).updateHash(msg.sender, unitId, unitHash);

54    success = IRegistry(agentRegistry).updateHash(msg.sender, unitId, unitHash);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L39


```solidity
file: main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

125   address[] memory checkOwners = IGnosisSafe(multisig).getOwners();

126   uint256 checkThreshold = IGnosisSafe(multisig).getThreshold();

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L125

```solidity
file: main/governance/contracts/wveOLAS.sol

157   numPoints = IVEOLAS(ve).totalNumPoints();

164   sPoint = IVEOLAS(ve).mapSupplyPoints(idx);

171   slopeChange = IVEOLAS(ve).mapSlopeChanges(ts);

178   pv = IVEOLAS(ve).getLastUserPoint(account);

185   userNumPoints = IVEOLAS(ve).getNumUserPoints(account);

195   uint256 userNumPoints = IVEOLAS(ve).getNumUserPoints(account);

204   balance = IVEOLAS(ve).getVotes(account);

224   balance = IVEOLAS(ve).balanceOf(account);

236   balance = IVEOLAS(ve).balanceOfAt(account, blockNumber);

244   unlockTime = IVEOLAS(ve).lockedEnd(account);

250   supply = IVEOLAS(ve).totalSupply();

257   supplyAt = IVEOLAS(ve).totalSupplyAt(blockNumber);

265   uint256 numPoints = IVEOLAS(ve).totalNumPoints();

266   PointVoting memory sPoint = IVEOLAS(ve).mapSupplyPoints(numPoints);

278   Power = IVEOLAS(ve).totalSupplyLocked();

286   vPower = IVEOLAS(ve).getPastTotalSupply(blockNumber);

293   return IVEOLAS(ve).supportsInterface(interfaceId);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L157

```solidity
file: main/tokenomics/contracts/Dispenser.sol

99     (reward, topUp) = ITokenomics(tokenomics).accountOwnerIncentives(msg.sender, unitTypes, unitIds);

104    success = ITreasury(treasury).withdrawToAccount(msg.sender, reward, topUp);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L99


```solidity
file: main/governance/contracts/bridges/FxERC20ChildTunnel.sol

79   bool success = IERC20(childToken).transfer(to, amount);

102  bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79

This report discusses how using inline assembly can optimize gas costs when making multiple external calls by reusing memory space, rather than expanding memory separately for each call. This can save thousands of gas compared to the solidity compiler’s default behavior.


## [G-06] Shorten arrays with inline assembly

Issue Description - When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating storage.

Proposed Optimization - Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new array.

Estimated Gas Savings - Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending on original length.

```solidity
function shorten(uint[] storage array, uint newLen) internal {
  assembly {
    sstore(array_slot, newLen)
  }
}
// Rather than:
function shorten(uint[] storage array, uint newLen) internal {
  uint[] memory newArray = new uint[](newLen);
  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }
  delete array;
  array = newArray;
}
```
Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas savings.

Code Snippet:


```solidity
file: main/registries/contracts/UnitRegistry.sol

97    uint32[] memory addSubComponentIds = new uint32[](numSubComponents + 1);

219   uint32[] memory allComponents = new uint32[](maxNumComponents);

221   uint32[] memory processedComponents = new uint32[](numUnits);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L97

```solidity
file: main/tokenomics/contracts/Depository.sol

252   uint256[] memory ids = new uint256[](numProducts);

399   bool[] memory positions = new bool[](numProducts);

446   bool[] memory positions = new bool[](numBonds);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L252

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

913    uint256[] memory incentives = new uint256[](7);

1103   uint256[] memory registriesSupply = new uint256[](2);

1169   address[] memory registries = new address[](2);

1179   uint256[] memory lastIds = new uint256[](2);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L913

## [G-07] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### Local variable fromLiq should be used instead of reading mapLockedBalances[msg.sender]

```solidity
file: main/governance/contracts/veOLAS.sol

510  function withdraw() external {
        LockedBalance memory lockedBalance = mapLockedBalances[msg.sender];
        if (lockedBalance.endTime > block.timestamp) {
            revert LockNotExpired(msg.sender, lockedBalance.endTime, block.timestamp);
        }
        uint256 amount = uint256(lockedBalance.amount);

        mapLockedBalances[msg.sender] = LockedBalance(0, 0);
        uint256 supplyBefore = supply;
        uint256 supplyAfter;
        // The amount cannot be less than the total supply
        unchecked {
            supplyAfter = supplyBefore - amount;
            supply = supplyAfter;
        }
        // oldLocked can have either expired <= timestamp or zero end
        // lockedBalance has only 0 end
        // Both can have >= 0 amount
        _checkpoint(msg.sender, lockedBalance, LockedBalance(0, 0), uint128(supplyAfter));

        emit Withdraw(msg.sender, amount, block.timestamp);
        emit Supply(supplyBefore, supplyAfter);

        // OLAS is a solmate-based ERC20 token with optimized transfer() that either returns true or reverts
        IERC20(token).transfer(msg.sender, amount);
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L510-L535

### Local variable fromLiq should be used instead of reading ETHOwned

```solidity
file:  main/tokenomics/contracts/Treasury.sol

330   if (token == ETH_TOKEN_ADDRESS) {
            uint256 amountOwned = ETHOwned;
            // Check if treasury has enough amount of owned ETH
            if (amountOwned >= tokenAmount) {
                // This branch is used to transfer ETH to a specified address
                amountOwned -= tokenAmount;
                ETHOwned = uint96(amountOwned);
                emit Withdraw(ETH_TOKEN_ADDRESS, to, tokenAmount);
                // Send ETH to the specified address
                (success, ) = to.call{value: tokenAmount}("");
                if (!success) {
                    revert TransferFailed(ETH_TOKEN_ADDRESS, address(this), to, tokenAmount);
                }
            } else {
                // Insufficient amount of treasury owned ETH
                revert LowerThan(tokenAmount, amountOwned);
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L330-L346

### Local variable fromLiq should be used instead of reading ETHFromServices

```solidity
file: main/tokenomics/contracts/Treasury.sol

400          uint256 amountETHFromServices = ETHFromServices;
        // Send ETH rewards, if any
        if (accountRewards > 0 && amountETHFromServices >= accountRewards) {
            amountETHFromServices -= accountRewards;
            ETHFromServices = uint96(amountETHFromServices);
            emit Withdraw(ETH_TOKEN_ADDRESS, account, accountRewards);
            (success, ) = account.call{value: accountRewards}("");
            if (!success) {
                revert TransferFailed(address(0), address(this), account, accountRewards);
            }
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L400-L410

### Local variable fromLiq should be used instead of reading effectiveBond

```solidity
file: main/tokenomics/contracts/Tokenomics.sol

609   function reserveAmountForBondProgram(uint256 amount) external returns (bool success) {
        // Check for the depository access
        if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }

        // Effective bond must be bigger than the requested amount
        uint256 eBond = effectiveBond;
        if (eBond >= amount) {
            // The effective bond value is adjusted with the amount that is reserved for bonding
            // The unrealized part of the bonding amount will be returned when the bonding program is closed
            eBond -= amount;
            effectiveBond = uint96(eBond);
            success = true;
            emit EffectiveBondUpdated(eBond);
        }
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol609-L625

## [G-08] Don’t make variables public unless necessary

Issue Description - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

Proposed Optimization - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

Estimated Gas Savings - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don’t need to be public.

Code Snippets:

```solidity
file: main/governance/contracts/OLAS.sol

31   address public owner;
  
33   address public minter;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L31


```solidity
file: main/registries/contracts/GenericManager.sol

14  address public owner;

16  bool public paused;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L14

```solidity
file: main/registries/contracts/GenericRegistry.sol

15    address public owner;
    
17    address public manager;
    
18    string public baseURI;
    
19    uint256 public totalSupply;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L15

```solidity
file: main/tokenomics/contracts/Depository.sol

80    address public owner;

83    uint32 public bondCounter;

86    uint32 public productCounter;

89    address public immutable olas;

91    address public tokenomics;
   
93    address public treasury;
   
95    address public bondCalculator;

98    mapping(uint256 => Bond) public mapUserBonds;
  
100   mapping(uint256 => Product) public mapBondProducts;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L80

```solidity
file: main/tokenomics/contracts/Dispenser.sol

18    address public owner;
    
20    uint8 internal _locked;

23    address public tokenomics;
   
25    address public treasury;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L18

```solidity
file:

140  address public owner;

143  uint96 public maxBond;

148  uint96 public inflationPerSecond;

151  address public treasury;

154  uint96 public veOLASThreshold;

157  address public depository;

161  uint96 public effectiveBond;

164  address public dispenser;

167  uint72 public codePerDev;

175  uint8 public currentYear;

172  bytes1 public tokenomicsParametersUpdated;

177  address public componentRegistry;

181  uint64 public epsilonRate;

184  uint32 public epochLen;

187  address public agentRegistry;

190   uint96 public nextVeOLASThreshold;

193  address public serviceRegistry;

196  uint32 public epochCounter;

199  uint32 public timeLaunch;

202  uint32 public nextEpochLen;

205   address public ve;

208  uint72 public devsPerCapital;

211  address public donatorBlacklist;

214  uint32 public lastDonationBlockNumber;

```

```solidity
file: main/tokenomics/contracts/DonatorBlacklist.sol

25  address public owner;

27  mapping(address => bool) public mapBlacklistedDonators;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L25

```solidity
file: main/governance/contracts/bridges/BridgedERC20.sol

22  address public owner;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L22

```solidity
file: main/governance/contracts/bridges/FxGovernorTunnel.sol

57  address public rootGovernor;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L57

```solidity
file: main/governance/contracts/multisigs/GuardCM.sol
 
125  address public governor;

130  mapping(uint256 => bool) public mapAllowedTargetSelectorChainIds;

132  mapping(address => uint256) public mapBridgeMediatorL1L2ChainIds;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L125

```solidity
file: main/governance/contracts/veOLAS.sol

110  uint256 public supply;

112  mapping(address => LockedBalance) public mapLockedBalances;

115  uint256 public totalNumPoints;

117  mapping(uint256 => PointVoting) public mapSupplyPoints;

119  mapping(address => PointVoting[]) public mapUserPoints

121  mapping(uint64 => int128) public mapSlopeChanges;

124  string public name;

126  string public symbol;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L110

```solidity
file: main/governance/contracts/bridges/HomeMediator.sol

57  address public foreignGovernor;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L57

```solidity
file: tokenomics/contracts/Treasury.sol

59  address public owner;

62  uint96 public ETHFromServices;

65  address public olas;

68  uint96 public ETHOwned;

71  address public tokenomics;

73  uint96 public minAcceptedETH = 0.065 ether;

76  address public depository;

78  uint8 public paused = 1;

80   uint8 internal _locked;

83  address public dispenser;

86  mapping(address => uint256) public mapTokenReserves

88  mapping(address => bool) public mapEnabledTokens;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main//tokenomics/contracts/Treasury.sol#L59

## [G-09] Assigning keccak operations to constant variables results in extra gas costs

"constants" expressions are expressions. As such, keccak assigned to a constant variable are re-computed at each use of the variable, which will consume gas unnecessarily. To save gas, Changing the variable from constant to immutable will make the computation run only once and therefore save gas.

```solidity
file:  main/governance/contracts/multisigs/GuardCM.sol

97      bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));
        // scheduleBatch selector
99      bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));
        // requireToPassMessage selector (Gnosis chain)
100     bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));
        // processMessageFromForeign selector (Gnosis chain)
101     bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));
        // sendMessageToChild selector (Polygon)
102     bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L97

## [G-10] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

Issue Description - The abi.encodePacked() function is used to concatenate multiple arguments into a byte array prior to 0.8.4. However, since 0.8.4 the bytes.concat() function was introduced which performs the same role but is preferred since it avoids ABI encoding overhead.

Proposed Optimization - Replace uses of abi.encodePacked() with bytes.concat() where multiple arguments need to be concatenated into a byte array.

Estimated Gas Savings - Using bytes.concat() instead of abi.encodePacked() saves approximately 300-500 gas per concatenation by avoiding ABI encoding.

Code Snippet:


```solidity
file: main/registries/contracts/GenericRegistry.sol

139   return string(abi.encodePacked(baseURI, CID_PREFIX, _toHex16(bytes16(unitHash)),

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L139

## [G-11] Create immutable variable to avoid an external call

Instead of performing an external call to get the root address each time _enableNode is invoked, we can perform this external call once in the constructor and store the root as an immutable variable. Doing this will save 1 external call each time _enableNode is invoked.


```solidity
file: main/tokenomics/contracts/GenericBondCalculator.sol

72    IUniswapV2Pair pair = IUniswapV2Pair(token);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L72

## [G-12] Don’t cache state varible if  that are only used once

```solidity
file: main/governance/contracts/OLAS.sol

99   uint256 _totalSupply = totalSupply;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L99


```solidity
file: main/tokenomics/contracts/Treasury.sol

224    uint256 reserves = mapTokenReserves[token] + tokenAmount;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L224


## [G-13] Splitting if() statements that use && saves gas

```solidity
file: main/governance/contracts/veOLAS.sol

188  if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) {

192  if (newLocked.endTime > block.timestamp && newLocked.amount > 0) {

306  if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188


```solidity
file: main/governance/contracts/wveOLAS.sol

215  if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) {

235  if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215

```solidity
 
file:  main/governance/contracts/multisigs/GuardCM.sol
 
519   if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001) {\

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L519

```solidity
file: main/lockbox-solana/solidity/liquidity_lockbox.sol

376  if (numPositions > 0 && amountLeft > 0) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

```solidity
file: main/tokenomics/contracts/Depository.sol

404   if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

529    if (_epsilonRate > 0 && _epsilonRate <= 17e18) {

536    if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) {

750    if (topUpEligible && incentiveFlags[unitType + 2]) {

801   if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) {

1138  if (lastEpoch > 0 && lastEpoch < curEpoch) {

1206  if (lastEpoch > 0 && lastEpoch < curEpoch) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529

```solidity
file: main/tokenomics/contracts/Treasury.sol

402  if (accountRewards > 0 && amountETHFromServices >= accountRewards) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L402

## [G-14] public functions not called by the contract should be declared external instead

when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external.

```solidity
file: main/governance/contracts/veOLAS.sol

633   function getVotes(address account) public view override returns (uint256) {

672   function getPastVotes(address account, uint256 blockNumber) public view override returns (uint256 balance) {

719   function totalSupply() public view override returns (uint256) {

726   function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt) {

745   function totalSupplyLocked() public view returns (uint256) {

752   function getPastTotalSupply(uint256 blockNumber) public view override returns (uint256) {

761   function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L633

```solidity
file: main/registries/contracts/GenericRegistry.sol

135   function tokenURI(uint256 unitId) public view virtual override returns (string memory) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135


```solidity
file: main/tokenomics/contracts/TokenomicsConstants.sol

30  function getSupplyCapForYear(uint256 numYears) public pure returns (uint256 supplyCap) {

66  function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L30

## [G-15] Pack struct variables

Reordering the structure of the Position ensures that the struct variables are packed efficiently, minimizing storage costs.

```solidity
file:  main/lockbox-solana/solidity/liquidity_lockbox.sol

7      address whirlpool;
    // Position mint (liquidity NFT) address, 32 bytes
    address positionMint;
    // Position liquidity, 16 bytes
    uint128 liquidity;
    // Tick lower index, 4 bytes
    int32 tickLowerIndex;
    /// Tick upper index, 4 bytes
    int32 tickUpperIndex;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L7-L15



## [G-16] Using storage instead of memory for structs/arrays saves gas 

```solidity
file: main/governance/contracts/wveOLAS.sol

213   PointVoting memory uPoint = getUserPoint(account, 0);

233   PointVoting memory uPoint = getUserPoint(account, 0);

266   PointVoting memory sPoint = IVEOLAS(ve).mapSupplyPoints(numPoints);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L213

```solidity
file: main/governance/contracts/veOLAS.sol

210    PointVoting memory lastPoint;

220    PointVoting memory initialPoint = lastPoint;

596    PointVoting memory uPoint = mapUserPoints[account][pointNumber - 1];

655    PointVoting memory pointNext = mapSupplyPoints[minPointNumber + 1];

739    PointVoting memory lastPoint = mapSupplyPoints[totalNumPoints];

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L210

## [G-17] When using storage instead of memory, we should cache any fields that need to be re-read in stack variables


```solidity
file: main/tokenomics/contracts/Tokenomics.sol

970        TokenomicsPoint storage nextEpochPoint = mapEpochTokenomics[eCounter + 1];
        // Update incentive fractions for the next epoch if they were requested by the changeIncentiveFractions() function
        // Check if the second bit is set to one
        if (tokenomicsParametersUpdated & 0x02 == 0x02) {
            // Confirm the change of incentive fractions
            emit IncentiveFractionsUpdated(eCounter + 1);
        } else {
            // Copy current tokenomics point into the next one such that it has necessary tokenomics parameters
            for (uint256 i = 0; i < 2; ++i) {
                nextEpochPoint.unitPoints[i].topUpUnitFraction = tp.unitPoints[i].topUpUnitFraction;
                nextEpochPoint.unitPoints[i].rewardUnitFraction = tp.unitPoints[i].rewardUnitFraction;
            }
            nextEpochPoint.epochPoint.rewardTreasuryFraction = tp.epochPoint.rewardTreasuryFraction;
            nextEpochPoint.epochPoint.maxBondFraction = tp.epochPoint.maxBondFraction;
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L970-L984

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

338           TokenomicsPoint storage tp = mapEpochTokenomics[1];

        // Setting initial parameters and fractions
        devsPerCapital = 1e18;
        tp.epochPoint.idf = 1e18;

        // Reward fractions
        // 0 stands for components and 1 for agents
        // The initial target is to distribute around 2/3 of incentives reserved to fund owners of the code
        // for components royalties and 1/3 for agents royalties
        tp.unitPoints[0].rewardUnitFraction = 83;
        tp.unitPoints[1].rewardUnitFraction = 17;
        // tp.epochPoint.rewardTreasuryFraction is essentially equal to zero

        // We consider a unit of code as n agents or m components.
        // Initially we consider 1 unit of code as either 2 agents or 1 component.
        // E.g. if we have 2 profitable components and 2 profitable agents, this means there are (2 x 2.0 + 2 x 1.0) / 3 = 2
        // units of code.
        // We assume that during one epoch the developer can contribute with one piece of code (1 component or 2 agents)
        codePerDev = 1e18;

        // Top-up fractions
        uint256 _maxBondFraction = 50;
        tp.epochPoint.maxBondFraction = uint8(_maxBondFraction);
        tp.unitPoints[0].topUpUnitFraction = 41;
        tp.unitPoints[1].topUpUnitFraction = 9;

        // Calculate initial effectiveBond based on the maxBond during the first epoch
        // maxBond = inflationPerSecond * epochLen * maxBondFraction / 100
        uint256 _maxBond = (_inflationPerSecond * _epochLen * _maxBondFraction) / 100;
        maxBond = uint96(_maxBond);
        effectiveBond = uint96(_maxBond);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L338-L369

## [G-18] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

```solidity
file: main/governance/contracts/OLAS.sol

108  for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L108-L110

```solidity
file: main/governance/contracts/bridges/FxGovernorTunnel.sol

153  for (uint256 j = 0; j < payloadLength; ++j) {
                payload[j] = data[i + j];
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L153-L155


```solidity
file: main/governance/contracts/bridges/HomeMediator.sol

153  for (uint256 j = 0; j < payloadLength; ++j) {
                payload[j] = data[i + j];
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L153-L155

```solidity
file: main/governance/contracts/multisigs/GuardCM.sol

237  for (uint256 j = 0; j < payloadLength; ++j) {
                payload[j] = data[i + j];
            }
        
292  for (uint256 i = 0; i < bridgePayload.length; ++i) {
                bridgePayload[i] = mediatorPayload[i + SELECTOR_DATA_LENGTH];
            }

318    for (uint256 i = 0; i < payload.length; ++i) {
                payload[i] = data[i + SELECTOR_DATA_LENGTH];
            }

340   for (uint256 i = 0; i < payload.length; ++i) {
            payload[i] = data[i + 4];
        }


```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L137-L239

```solidity
file: main/lockbox-solana/solidity/liquidity_lockbox.sol

369  for (uint32 i = 0; i < numPositions; ++i) {
            positionAddresses[i] = positionAccounts[firstAvailablePositionAccountIndex + i];
            positionAmounts[i] = mapPositionAccountLiquidity[positionAddresses[i]];
            positionPdaAtas[i] = mapPositionAccountPdaAta[positionAddresses[i]];
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L369-L373


```solidity
file: main/registries/contracts/AgentRegistry.sol

40   for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
            if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) {
                revert ComponentNotFound(dependencies[iDep]);
            }
            lastId = dependencies[iDep];
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L40-L45

```solidity
file: main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

113    for (uint256 i = 0; i < payloadLength; ++i) {
                payload[i] = data[i + DEFAULT_DATA_LENGTH];
            }

139    for (uint256 i = 0; i < numOwners; ++i) {
            if (owners[i] != checkOwners[numOwners - i - 1]) {
                revert WrongOwner(owners[i]);
            }
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L113-L115


```solidity
file: main/tokenomics/contracts/Depository.sol

274  for (uint256 i = 0; i < numClosedProducts; ++i) {
            closedProductIds[i] = ids[i];
        }

402   for (uint256 i = 0; i < numProducts; ++i) {
            // Product is always active if its supply is not zero, and inactive otherwise
            if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) {
                positions[i] = true;
                ++numSelectedProducts;
            }
        }

413   for (uint256 i = 0; i < numProducts; ++i) {
            if (positions[i]) {
                productIds[numPos] = i;
                ++numPos;
            }
        }

468   for (uint256 i = 0; i < numBonds; ++i) {
            if (positions[i]) {
                bondIds[numPos] = i;
                ++numPos;
            }
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L274-L276


```solidity
file: main/tokenomics/contracts/Tokenomics.sol

808   for (uint256 i = 0; i < numServices; ++i) {
            // Check for the service Id existence
            if (!IServiceRegistry(serviceRegistry).exists(serviceIds[i])) {
                revert ServiceDoesNotExist(serviceIds[i]);
            }
        }

978   for (uint256 i = 0; i < 2; ++i) {
                nextEpochPoint.unitPoints[i].topUpUnitFraction = tp.unitPoints[i].topUpUnitFraction;
                nextEpochPoint.unitPoints[i].rewardUnitFraction = tp.unitPoints[i].rewardUnitFraction;
            }

1104   for (uint256 i = 0; i < 2; ++i) {
            registriesSupply[i] = IToken(registries[i]).totalSupply();
        }

1174   for (uint256 i = 0; i < 2; ++i) {
            registriesSupply[i] = IToken(registries[i]).totalSupply();
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L808-L813


```solidity
file: main/tokenomics/contracts/TokenomicsConstants.sol

55  for (uint256 i = 0; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }
92   for (uint256 i = 1; i < numYears; ++i) {
                supplyCap += (supplyCap * maxMintCapFraction) / 100;
            }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L55-L57


```solidity
file: main/tokenomics/contracts/Treasury.sol

276   for (uint256 i = 0; i < numServices; ++i) {
            if (amounts[i] == 0) {
                revert ZeroValue();
            }
            totalAmount += amounts[i];
        }

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L276-L281

## [G-19] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use. we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use

```solidity
file: main/governance/contracts/multisigs/GuardCM.sol

278   (address homeMediator, bytes memory mediatorPayload, ) = abi.decode(payload, (address, bytes, uint256));

297   (bytes memory l2Message) = abi.decode(bridgePayload, (bytes));

323   (address fxGovernorTunnel, bytes memory l2Message) = abi.decode(payload, (address, bytes));

352    abi.decode(payload, (address, uint256, bytes, bytes32, bytes32, uint256));

356    abi.decode(payload, (address[], uint256[], bytes[], bytes32, bytes32, uint256));

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L278

```solidity
file: main/governance/contracts/bridges/FxERC20ChildTunnel.sol

76   (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L76


```solidity
file: main/governance/contracts/bridges/FxERC20RootTunnel.sol

74   (address from, address to, uint256 amount) = abi.decode(message, (address, address, uint256));

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L74

## [G-20] Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient

SSTORE from 0 to 1 (or any non-zero value) costs 20000 gas. SSTORE from 1 to 2 (or any other non-zero value) costs 5000 gas.

By storing the original value once again, a refund is triggered (https://eips.ethereum.org/EIPS/eip-2200).

Since refunds are capped to a percentage of the total transaction’s gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.

Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.

See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/86bd4d73896afcb35a205456e361436701823c7a/contracts/security/ReentrancyGuard.sol#L29-L33


```solidity
file:  tokenomics/contracts/Depository.sol

405    positions[i] = true;

457    positions[i] = true;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main//tokenomics/contracts/Depository.sol#L405


```solidity
file: tokenomics/contracts/Treasury.sol

500   mapEnabledTokens[token] = true;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main//tokenomics/contracts/Treasury.sol#L500

```solidity
file: tokenomics/contracts/Tokenomics.sol

622   success = true;

761   mapNewUnits[unitType][serviceUnitIds[j]] = true;

767   mapNewOwners[unitOwner] = true;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main//tokenomics/contracts/Tokenomics.sol#L622

```solidity
file: registries/contracts/GenericManager.sol

42   paused = true;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main//registries/contracts/GenericManager.sol#L42


```solidity
file: main/registries/contracts/GenericManager.sol

53    paused = false;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L53

```solidity
file:  main/tokenomics/contracts/Treasury.sol

455   success = false;

518   mapEnabledTokens[token] = false;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L455

## [G-21] Use constants instead of type(uintX).max

Using constants instead of type(uintX).max saves gas in Solidity. This is because the type(uintX).max function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. Constants, on the other hand, are stored in the bytecode of your contract, so they do not have to be recalculated every time you need them. Saves 13 GAS.

```solidity
file: main/governance/contracts/OLAS.sol

131   if (spenderAllowance != type(uint256).max) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L131

```solidity
file: main/governance/contracts/veOLAS.sol

392   if (amount > type(uint96).max) {

393   revert Overflow(amount, type(uint96).max);

449   if (amount > type(uint96).max) {

450   revert Overflow(amount, type(uint96).max);

474   if (amount > type(uint96).max) {

475   revert Overflow(amount, type(uint96).max);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392

```solidity
file: main/lockbox-solana/solidity/liquidity_lockbox.sol

52    address[type(uint32).max] public positionAccounts;

95    if (positionData.liquidity > type(uint64).max) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L52

```solidity
file: main/tokenomics/contracts/Depository.sol

195   if (priceLP > type(uint160).max) {

196   revert Overflow(priceLP, type(uint160).max);

205   if (supply > type(uint96).max) {

206   revert Overflow(supply, type(uint96).max);

216   if (maturity > type(uint32).max) {

217   revert Overflow(maturity, type(uint32).max);

311   if (maturity > type(uint32).max) {

312   revert Overflow(maturity, type(uint32).max);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L195

```solidity
file: main/tokenomics/contracts/GenericBondCalculator.sol

59   if (totalTokenValue > type(uint192).max) {

60   revert Overflow(totalTokenValue, type(uint192).max);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L59


```solidity
file: main/tokenomics/contracts/Tokenomics.sol

639   if (eBond > type(uint96).max) {

640   revert Overflow(eBond, type(uint96).max);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L639

```solidity
file: main/tokenomics/contracts/Treasury.sol

128   if (amount + ETHFromServices > type(uint96).max) {

129   revert Overflow(amount, type(uint96).max);

194   if (_minAcceptedETH > type(uint96).max) {

195   revert Overflow(_minAcceptedETH, type(uint96).max);

291   if (donationETH + ETHOwned > type(uint96).max) {

292    revert Overflow(donationETH, type(uint96).max);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L128

## [G-22] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

```solidity
file: main/registries/contracts/GenericManager.sol

16    bool public paused;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L16

```solidity
file:  main/governance/contracts/multisigs/GuardCM.sol

130     mapping(uint256 => bool) public mapAllowedTargetSelectorChainIds;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L130 


```solidity
file: main/tokenomics/contracts/Tokenomics.sol

225   mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;

227   mapping(address => bool) public mapNewOwners;

693   bool[] memory incentiveFlags = new bool[](4);

706   bool topUpEligible;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L225

```solidity
file: main/tokenomics/contracts/Treasury.sol

88   mapping(address => bool) public mapEnabledTokens;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L88

```solidity
file: main/tokenomics/contracts/Depository.sol

360    bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;

399    bool[] memory positions = new bool[](numProducts);

446    bool[] memory positions = new bool[](numBonds);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L360

```solidity
file:  main/tokenomics/contracts/Dispenser.sol
 
101    bool success;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L101

```solidity
file:  main/governance/contracts/bridges/FxERC20ChildTunnel.sol

79     bool success = IERC20(childToken).transfer(to, amount);

102    bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79

```solidity
file: main/governance/contracts/bridges/FxERC20RootTunnel.sol

98    bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98


## [G-23] Unused Named Return Variables

Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.

```solidity
file:  main/governance/contracts/veOLAS.sol

155    function getNumUserPoints(address account) external view returns (uint256 accountNumPoints) {

614    function lockedEnd(address account) external view returns (uint256 unlockTime) {

622    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance) {

726    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L155


```solidity
file:

16    function totalNumPoints() external view returns (uint256 numPoints);

26    function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange);

36    function getNumUserPoints(address account) external view returns (uint256 userNumPoints);

49    function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance);

54    function balanceOf(address account) external view returns (uint256 balance);

60    function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance);

65    function lockedEnd(address account) external view returns (uint256 unlockTime);

70    function getVotes(address account) external view returns (uint256 balance);

74    function totalSupply() external view returns (uint256 supply);

79    function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt);

84    function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower);

88    function totalSupplyLocked() external view returns (uint256 vPower);

93    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower);

156   function totalNumPoints() external view returns (uint256 numPoints) {

170   function mapSlopeChanges(uint64 ts) external view returns (int128 slopeChange) {

184   function getNumUserPoints(address account) external view returns (uint256 userNumPoints) {

203   function getVotes(address account) external view returns (uint256 balance) {

211   function getPastVotes(address account, uint256 blockNumber) external view returns (uint256 balance) {

223   function balanceOf(address account) external view returns (uint256 balance) {

231   function balanceOfAt(address account, uint256 blockNumber) external view returns (uint256 balance) {

243   function lockedEnd(address account) external view returns (uint256 unlockTime) {

249   function totalSupply() external view returns (uint256 supply) {

256   function totalSupplyAt(uint256 blockNumber) external view returns (uint256 supplyAt) {

263   function totalSupplyLockedAtT(uint256 ts) external view returns (uint256 vPower) {

277   function totalSupplyLocked() external view returns (uint256 vPower) {

285    function getPastTotalSupply(uint256 blockNumber) external view returns (uint256 vPower) {

```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L16


```solidity
file: main/governance/contracts/multisigs/GuardCM.sol

579    function getTargetSelectorChainId(address target, bytes4 selector, uint256 chainId) external view
        returns (bool status)
    {

597   function getBridgeMediatorChainId(address bridgeMediatorL1) external view
        returns (address bridgeMediatorL2, uint256 chainId)
    {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L579-L581


```solidity
file:  main/lockbox-solana/solidity/liquidity_lockbox.sol

338    function getLiquidityAmountsAndPositions(uint64 amount)
        external view returns (uint64[] positionAmounts, address[] positionAddresses, address[] positionPdaAtas)
    {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L338-L340


```solidity
file: main/registries/contracts/interfaces/IRegistry.sol

27    function updateHash(address owner, uint256 unitId, bytes32 unitHash) external returns (bool success);

34    function getLocalSubComponents(uint256 unitId) external view returns (uint32[] memory subComponentIds, uint256 numSubComponents);

38    function calculateSubComponents(uint32[] memory unitIds) external view returns (uint32[] memory subComponentIds);

44    function getUpdatedHashes(uint256 unitId) external view returns (uint256 numHashes, bytes32[] memory unitHashes);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IRegistry.sol#L27


```solidity
file: main/tokenomics/contracts/Depository.sol

183   function create(address token, uint256 priceLP, uint256 supply, uint256 vesting) external returns (uint256 productId) {

244   function close(uint256[] memory productIds) external returns (uint256[] memory closedProductIds) {

291   function deposit(uint256 productId, uint256 tokenAmount) external
        returns (uint256 payout, uint256 maturity, uint256 bondId)
    {

356   function redeem(uint256[] memory bondIds) external returns (uint256 payout) {

396   function getProducts(bool active) external view returns (uint256[] memory productIds) {

424    function isActiveProduct(uint256 productId) external view returns (bool status) {

435    function getBonds(address account, bool matured) external view
        returns (uint256[] memory bondIds, uint256 payout)
    {

480   function getBondStatus(uint256 bondId) external view returns (uint256 payout, bool matured) {

491   function getCurrentPriceLP(address token) external view returns (uint256 priceLP) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L183


```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

374    function tokenomicsImplementation() external view returns (address implementation) {

609    unction reserveAmountForBondProgram(uint256 amount) external returns (bool success) {

1085   function accountOwnerIncentives(address account, uint256[] memory unitTypes, uint256[] memory unitIds) external
        returns (uint256 reward, uint256 topUp)
    {

1160   function getOwnerIncentives(address account, uint256[] memory unitTypes, uint256[] memory unitIds) external view
        returns (uint256 reward, uint256 topUp)
    {

1237   function getInflationPerEpoch() external view returns (uint256 inflationPerEpoch) {

1252  function getIDF(uint256 epoch) external view returns (uint256 idf)

1262   function getLastIDF() external view returns (uint256 idf)

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L374

```solidity
file:  main/tokenomics/contracts/Treasury.sol

387   function withdrawToAccount(address account, uint256 accountRewards, uint256 accountTopUps) external
        returns (bool success)
    {

428    function rebalanceTreasury(uint256 treasuryRewards) external returns (bool success) {

466   function drainServiceSlashedFunds() external returns (uint256 amount) {

526   function isEnabled(address token) external view returns (bool enabled) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L387-L389

## [G-24] Using delete statement can save gas

```solidity
file:  main/tokenomics/contracts/Tokenomics.sol

662    mapUnitIncentives[unitType][unitId].pendingRelativeReward = 0;

678    mapUnitIncentives[unitType][unitId].pendingRelativeTopUp = 0;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L662

## [G-25] Default value initialization

If a variable is not set/initialized, it is assumed to have the default value(0, false, 0x0 etc depending on the data type). Explicitly initializing it with its default value is an anti-pattern and wastes gas.


```solidity
file:  main/lockbox-solana/solidity/liquidity_lockbox.sol

345    uint64 liquiditySum = 0;

346    uint32 numPositions = 0;

347    uint64 amountLeft = 0;

369    for (uint32 i = 0; i < numPositions; ++i) {
    
```
https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L345

```solidity
file: main/governance/contracts/multisigs/GuardCM.sol

212   for (uint256 i = 0; i < data.length;) {

237   for (uint256 j = 0; j < payloadLength; ++j) {

292   for (uint256 i = 0; i < bridgePayload.length; ++i) {

318   for (uint256 i = 0; i < payload.length; ++i) {

340   for (uint256 i = 0; i < payload.length; ++i) {

360   for (uint i = 0; i < targets.length; ++i) {

458   for (uint256 i = 0; i < targets.length; ++i) {
    
511  for (uint256 i = 0; i < chainIds.length; ++i) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L212


```solidity
file: main/registries/contracts/UnitRegistry.sol

98   for (uint256 i = 0; i < numSubComponents; ++i) {

211  for (uint32 i = 0; i < numUnits; ++i) {

227  for (counter = 0; counter < maxNumComponents; ++counter) {

234  for (uint32 i = 0; i < numUnits; ++i) {

262  for (uint32 i = 0; i < counter; ++i) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L98

```solidity
file: contracts/multisigs/GnosisSafeSameAddressMultisig.sol

113   for (uint256 i = 0; i < payloadLength; ++i) {

139   for (uint256 i = 0; i < numOwners; ++i) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L113

```solidity
file: main/tokenomics/contracts/Depository.sol  

255   for (uint256 i = 0; i < numProducts; ++i) {

274   for (uint256 i = 0; i < numClosedProducts; ++i) {

357   for (uint256 i = 0; i < bondIds.length; ++i) {

402   for (uint256 i = 0; i < numProducts; ++i) {

413   for (uint256 i = 0; i < numProducts; ++i) {

448   for (uint256 i = 0; i < numBonds; ++i) {

468   for (uint256 i = 0; i < numBonds; ++i) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L255

```solidity
file: main/tokenomics/contracts/Tokenomics.sol

662   mapUnitIncentives[unitType][unitId].pendingRelativeReward = 0;

678   mapUnitIncentives[unitType][unitId].pendingRelativeTopUp = 0;

702   for (uint256 i = 0; i < numServices; ++i) {

714   for (uint256 unitType = 0; unitType < 2; ++unitType) {

728   for (uint256 j = 0; j < numServiceUnits; ++j) {

808   for (uint256 i = 0; i < numServices; ++i) {

978   for (uint256 i = 0; i < 2; ++i) {

992   nextEpochLen = 0;

998   nextVeOLASThreshold = 0;

1027  tokenomicsParametersUpdated = 0;

1034  tokenomicsParametersUpdated = 0;

1104  for (uint256 i = 0; i < 2; ++i) {

1110   for (uint256 i = 0; i < unitIds.length; ++i) {

1132   for (uint256 i = 0; i < unitIds.length; ++i) {

1141   mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch = 0;

1146    mapUnitIncentives[unitTypes[i]][unitIds[i]].reward = 0;

1149   mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp = 0;

1174   for (uint256 i = 0; i < 2; ++i) {

1180   for (uint256 i = 0; i < unitIds.length; ++i) {

1202   for (uint256 i = 0; i < unitIds.length; ++i) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L662

```solidity
file: /main/governance/contracts/veOLAS.sol

232   for (uint256 i = 0; i < 255; ++i) {

249   lastPoint.bias = 0;

253   lastPoint.slope = 0;

283   lastPoint.slope = 0;

286   lastPoint.bias = 0;

561   for (uint256 i = 0; i < 128; ++i) {

693   for (uint256 i = 0; i < 255; ++i) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L232


```solidity
file: governance/contracts/bridges/FxGovernorTunnel.sol

125   for (uint256 i = 0; i < dataLength;) {

153   for (uint256 j = 0; j < payloadLength; ++j) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L125


```solidity
file: main/governance/contracts/bridges/HomeMediator.sol

125   for (uint256 i = 0; i < dataLength;) {

153   for (uint256 j = 0; j < payloadLength; ++j) {

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L125

## [G-26] Don’t cache call if used only once

Caching here will simply increase gas cost, since it doesn’t affect readability, we should not cache since we only reference the cached call only once.

```solidity
file: main/tokenomics/contracts/Depository.sol

262    ITokenomics(tokenomics).refundFromBondProgram(supply);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L262

```solidity
file: main/tokenomics/contracts/Tokenomics.sol

708   address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);

709   topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||

710   IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;

716   (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).

717   getUnitIdsOfService(IServiceRegistry.UnitType(unitType), serviceIds[i]);

739   _finalizeIncentivesForUnitId(lastEpoch, unitType, serviceUnitIds[j]);

765   address unitOwner = IToken(registries[unitType]).ownerOf(serviceUnitIds[j]);

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L708


## [G-27] State variables which are not modified within functions should be set as constant or immutable for values set at deployment

Cache such variables and perform operations on them, if operations include modifications to the state variable(s) then remember to equate the state variable to it’s cached counterpart at the end

```solidity
file: main/registries/contracts/GenericRegistry.sol

23   uint256 internal _locked = 1;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L23

```solidity
file: main/governance/contracts/veOLAS.sol

124  string public name;

126  string public symbol;

```
https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L124