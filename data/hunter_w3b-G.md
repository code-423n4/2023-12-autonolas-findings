# Gas-Optmization

## [G-01] Use constants instead of type(uintx).max

**Issue Description**\
Using type(uint256).max to represent the maximum approvable amount for ERC20 tokens uses more gas in the distribution process
and also for each transaction than using a constant.

**Proposed Optimization**\
Define a constant such as MAX_UINT256 = type(uint256).max and use that constant instead.

**Estimated Gas Savings**\
Using a constant avoids the overhead of calling the type(uint256) method each time. This could save ~100 gas per transaction.
For contracts with many transactions, this can add up to significant savings over time.

**Attachments**

- **Code Snippets**

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

## [G-02] Using assembly to revert with an error message

**Issue Description**\
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings**\
Here’s an example;

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

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message
with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks
the solidity compiler does under the hood.

**Attachments**

- **Code Snippets**

```solidity
File: governance/contracts/bridges/BridgedERC20.sol

33            revert OwnerOnly(msg.sender, owner);

38            revert ZeroAddress();

51            revert OwnerOnly(msg.sender, owner);

62            revert OwnerOnly(msg.sender, owner);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L33

```solidity
File: tokenomics/contracts/Depository.sol

112            revert ZeroAddress();

126            revert OwnerOnly(msg.sender, owner);

131            revert ZeroAddress();

146            revert OwnerOnly(msg.sender, owner);

166            revert OwnerOnly(msg.sender, owner);

186            revert OwnerOnly(msg.sender, owner);

191            revert ZeroValue();

196            revert Overflow(priceLP, type(uint160).max);

201            revert ZeroValue();

206            revert Overflow(supply, type(uint96).max);

211            revert LowerThan(vesting, MIN_VESTING);

217            revert Overflow(maturity, type(uint32).max);

222            revert UnauthorizedToken(token);

227            revert LowerThan(ITokenomics(tokenomics).effectiveBond(), supply);

247            revert OwnerOnly(msg.sender, owner);

296            revert ZeroValue();

305            revert ProductClosed(productId);


312            revert Overflow(maturity, type(uint32).max);

324            revert ProductSupplyLow(token, productId, payout, supply);

369                revert OwnerOnly(msg.sender, mapUserBonds[bondIds[i]].account);

385            revert ZeroValue();

440            revert ZeroAddress();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L112

```solidity
File: tokenomics/contracts/Dispenser.sol



37            revert ZeroAddress();

49            revert OwnerOnly(msg.sender, owner);

54            revert ZeroAddress();

67            revert OwnerOnly(msg.sender, owner);

94            revert ReentrancyGuard();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L15

```solidity
File: /home/x0w3r/progress-ON/olas/scope/contracts/DonatorBlacklist.sol

39            revert OwnerOnly(msg.sender, owner);

44            revert ZeroAddress();

59            revert OwnerOnly(msg.sender, owner);

64            revert WrongArrayLength(accounts.length, statuses.length);

70                revert ZeroAddress();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L39

```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol


40            revert ZeroAddress();

60            revert ZeroAddress();

81            revert TransferFailed(childToken, address(this), to, amount);

93            revert ZeroValue();

104            revert TransferFailed(childToken, msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L40

```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol

44            revert ZeroAddress();

64            revert ZeroAddress();

89            revert ZeroValue();

100            revert TransferFailed(rootToken, msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L44

```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol


65            revert ZeroAddress();

85            revert SelfCallOnly(msg.sender, address(this));

90            revert ZeroAddress();

110            revert FxChildOnly(msg.sender, fxChild);

115            revert RootGovernorOnly(rootMessageSender, rootGovernor);

121            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);

144                revert ZeroAddress();

148                revert InsufficientBalance(value, address(this).balance);

162                revert TargetExecFailed(target, value, payload);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L65

```solidity
File: registries/contracts/GenericManager.sol


23            revert OwnerOnly(msg.sender, owner);

28            revert ZeroAddress();

39            revert OwnerOnly(msg.sender, owner);

50            revert OwnerOnly(msg.sender, owner);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L23

```solidity
File: registries/contracts/GenericRegistry.sol


40            revert OwnerOnly(msg.sender, owner);

45            revert ZeroAddress();

56            revert OwnerOnly(msg.sender, owner);

61            revert ZeroAddress();

81            revert OwnerOnly(msg.sender, owner);

86            revert ZeroValue();

100            revert Overflow(unitId, totalSupply);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L40

```solidity
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

62            revert ZeroValue();

94            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);

105            revert UnauthorizedMultisig(multisig);

120                revert MultisigExecFailed(multisig);

130            revert WrongThreshold(checkThreshold, threshold);

134            revert WrongNumOwners(checkOwners.length, numOwners);

141                revert WrongOwner(owners[i]);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L62

```solidity
File: governance/contracts/multisigs/GuardCM.sol


145            revert ZeroAddress();

156            revert OwnerOnly(msg.sender, owner);

161            revert ZeroAddress();

172            revert OwnerOnly(msg.sender, owner);

177            revert ZeroValue();

200            revert NotAuthorized(target, bytes4(data), chainId);

227                revert ZeroAddress();

232                revert IncorrectDataLength(payloadLength, SELECTOR_DATA_LENGTH);

263                revert WrongSelector(functionSig, chainId);

268                revert IncorrectDataLength(data.length, MIN_GNOSIS_PAYLOAD_LENGTH);

281                revert WrongL2BridgeMediator(homeMediator, bridgeMediatorL2);

287                revert WrongSelector(functionSig, chainId);

308                revert WrongSelector(functionSig, chainId);

313                revert IncorrectDataLength(data.length, MIN_POLYGON_PAYLOAD_LENGTH);

326                revert WrongL2BridgeMediator(fxGovernorTunnel, bridgeMediatorL2);

411                    revert IncorrectDataLength(data.length, SELECTOR_DATA_LENGTH);

422                        revert IncorrectDataLength(data.length, MIN_SCHEDULE_DATA_LENGTH);

454            revert WrongArrayLength(targets.length, selectors.length, statuses.length, chainIds.length);

461                revert ZeroAddress();

471                revert ZeroValue();

507            revert WrongArrayLength(bridgeMediatorL1s.length, bridgeMediatorL2s.length, chainIds.length, chainIds.length);

514                revert ZeroAddress();

520                revert L2ChainIdNotSupported(chainId);


549                revert NotDefeated(governorCheckProposalId, state);

563            revert OwnerOnly(msg.sender, owner);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L145

```solidity
File: governance/contracts/bridges/HomeMediator.sol

65            revert ZeroAddress();

85            revert SelfCallOnly(msg.sender, address(this));

90            revert ZeroAddress();

115            revert ForeignGovernorOnly(bridgeGovernor, governor);

121            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);

148                revert InsufficientBalance(value, address(this).balance);

162                revert TargetExecFailed(target, value, payload);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L65

```solidity
File: governance/contracts/OLAS.sol

45            revert ManagerOnly(msg.sender, owner);

49            revert ZeroAddress();

60            revert ManagerOnly(msg.sender, owner);

64            revert ZeroAddress();

78            revert ManagerOnly(msg.sender, minter);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L45

```solidity
File: tokenomics/contracts/Tokenomics.sol


279            revert AlreadyInitialized();

286            revert ZeroAddress();

297            revert LowerThan(_epochLen, MIN_EPOCH_LENGTH);

302            revert Overflow(_epochLen, ONE_YEAR);

321            revert Overflow(_timeLaunch + ONE_YEAR, block.timestamp);

387            revert OwnerOnly(msg.sender, owner);

392            revert ZeroAddress();

407            revert OwnerOnly(msg.sender, owner);

412            revert ZeroAddress();

453            revert OwnerOnly(msg.sender, owner);

477            revert OwnerOnly(msg.sender, owner);

507            revert OwnerOnly(msg.sender, owner);

572            revert OwnerOnly(msg.sender, owner);

577            revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);

582            revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);

612            revert ManagerOnly(msg.sender, depository);

633            revert ManagerOnly(msg.sender, depository);

640            revert Overflow(eBond, type(uint96).max);

721                    revert ServiceNeverDeployed(serviceIds[i]);

802            revert DonatorBlacklisted(donator);

811                revert ServiceDoesNotExist(serviceIds[i]);

888            revert DelegatecallOnly();

893            revert SameBlockNumberViolation();

1070            revert TreasuryRebalanceFailed(eCounter);

1090            revert ManagerOnly(msg.sender, dispenser);

1095            revert WrongArrayLength(unitTypes.length, unitIds.length);

1113                revert Overflow(unitTypes[i], 1);

1118                revert WrongUnitId(unitIds[i], unitTypes[i]);

1125                revert OwnerOnly(unitOwner, account);

1165            revert WrongArrayLength(unitTypes.length, unitIds.length);

1183                revert Overflow(unitTypes[i], 1);

1188                revert WrongUnitId(unitIds[i], unitTypes[i]);

1195                revert OwnerOnly(unitOwner, account);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L279

```solidity
File: tokenomics/contracts/TokenomicsProxy.sol


36            revert ZeroTokenomicsAddress();

41            revert ZeroTokenomicsData();

50            revert InitializationFailed();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsProxy.sol#L36

```solidity
File: tokenomics/contracts/Treasury.sol


101            revert ZeroAddress();

129            revert Overflow(amount, type(uint96).max);

140            revert OwnerOnly(msg.sender, owner);

145            revert ZeroAddress();

159            revert OwnerOnly(msg.sender, owner);

185            revert OwnerOnly(msg.sender, owner);

190            revert ZeroValue();

195            revert Overflow(_minAcceptedETH, type(uint96).max);

215            revert ManagerOnly(msg.sender, depository);

220            revert UnauthorizedToken(token);

229            revert InsufficientAllowance(IToken(token).allowance((account), address(this)), tokenAmount);

237            revert TransferFailed(token, account, address(this), tokenAmount);

266            revert LowerThan(msg.value, minAcceptedETH);

272            revert WrongArrayLength(numServices, amounts.length);

285            revert WrongAmount(msg.value, totalAmount);

292            revert Overflow(donationETH, type(uint96).max);

321            revert TransferFailed(token, address(this), to, tokenAmount);

341                    revert TransferFailed(ETH_TOKEN_ADDRESS, address(this), to, tokenAmount);

345                revert LowerThan(tokenAmount, amountOwned);

350                revert UnauthorizedToken(token);

364                    revert TransferFailed(token, address(this), to, tokenAmount);

397            revert ManagerOnly(msg.sender, dispenser);

436            revert ManagerOnly(msg.sender, tokenomics);

469            revert OwnerOnly(msg.sender, owner);

478            revert LowerThan(slashedFunds, minAcceptedETH);

490            revert OwnerOnly(msg.sender, owner);

510            revert OwnerOnly(msg.sender, owner);

534            revert OwnerOnly(msg.sender, owner);
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L101

```solidity
File: registries/contracts/UnitRegistry.sol


60            revert ManagerOnly(msg.sender, manager);

65            revert ZeroAddress();

70            revert ZeroValue();

126            revert ManagerOnly(msg.sender, manager);

132                revert ComponentNotFound(unitId);

140            revert ZeroValue();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L60

```solidity
File: governance/contracts/veOLAS.sol

380            revert ZeroValue();

384            revert NoValueLocked(account);

388            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);

393            revert Overflow(amount, type(uint96).max);

414            revert ZeroAddress();

438            revert LockedValueNotZero(account, uint256(lockedBalance.amount));

442            revert UnlockTimeIncorrect(account, block.timestamp, unlockTime);

446            revert MaxUnlockTimeReached(account, block.timestamp + MAXTIME, unlockTime);

450            revert Overflow(amount, type(uint96).max);

462            revert ZeroValue();

466            revert NoValueLocked(msg.sender);

470            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);

475            revert Overflow(amount, type(uint96).max);

491            revert NoValueLocked(msg.sender);

495            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);

499            revert UnlockTimeIncorrect(msg.sender, lockedBalance.endTime, unlockTime);

503            revert MaxUnlockTimeReached(msg.sender, block.timestamp + MAXTIME, unlockTime);

513            revert LockNotExpired(msg.sender, lockedBalance.endTime, block.timestamp);

646            revert WrongBlockNumber(blockNumber, block.number);

768        revert NonTransferable(address(this));

773        revert NonTransferable(address(this));

778        revert NonTransferable(address(this));

796        revert NonDelegatable(address(this));
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L380

## [G-03] Use assembly to reuse memory space when making more than one external call

**Issue Description**\
When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not
clear or reuse memory, so it expands memory each time.

**Proposed Optimization**\
Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments
without expanding memory further.

**Estimated Gas Savings**\
Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000
gas. With larger arguments or return data, the savings would be even greater.

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

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also
reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding
memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it
wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas
by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity
overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within
that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00
before exiting the assembly block.

**Attachments**

- **Code Snippets**

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

This report discusses how using inline assembly can optimize gas costs when making multiple external calls by reusing
memory space, rather than expanding memory separately for each call. This can save thousands of gas compared to the
solidity compiler's default behavior.

## [G-04] Don't make variables public unless necessary

**Issue Description**\
Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates
a public getter function of the same name, increasing the contract size.

**Proposed Optimization**\
Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private
or internal visibility.

**Estimated Gas Savings**\
The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up
over many transactions targeting a contract with public state variables that don't need to be public.

**Attachments**

- **Code Snippets**

```solidity
File:  tokenomics/contracts/Depository.sol

    uint256 public constant MIN_VESTING = 1 days;

    string public constant VERSION = "1.0.1";

    address public owner;

    uint32 public bondCounter;

    uint32 public productCounter;

    address public immutable olas;

    address public tokenomics;

    address public treasury;

    address public bondCalculator;

    mapping(uint256 => Bond) public mapUserBonds;

    mapping(uint256 => Product) public mapBondProducts;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L74

```solidity
File: tokenomics/contracts/Dispenser.sol

    address public owner;

    address public tokenomics;

    address public treasury;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L18

```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol


    uint256 public constant DEFAULT_DATA_LENGTH = 36;

    address public immutable fxChild;

    address public rootGovernor;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L53

```solidity
File: tokenomics/contracts/GenericBondCalculator.sol

    address public immutable olas;

    address public immutable tokenomics;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L22

```solidity
File: registries/contracts/GenericManager.sol

    address public owner;

    bool public paused;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L14

```solidity
File: registries/contracts/GenericRegistry.sol

    address public owner;

    address public manager;

    string public baseURI;

    uint256 public totalSupply;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L14

```solidity
File: registries/contracts/multisigs/GnosisSafeMultisig.sol

    bytes4 public constant GNOSIS_SAFE_SETUP_SELECTOR = 0xb63e800d;

    uint256 public constant DEFAULT_DATA_LENGTH = 144;

    address payable public immutable gnosisSafe;

    address public immutable gnosisSafeProxyFactory;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol#L26

```solidity
File: governance/contracts/multisigs/GuardCM.sol

    bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));

    bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));

    bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));

    bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));

    bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));

    uint256 public governorCheckProposalId = 88250008686885504216650933897987879122244685460173810624866685274624741477673;

    uint256 public constant MIN_SCHEDULE_DATA_LENGTH = 260;

    uint256 public constant SELECTOR_DATA_LENGTH = 4;

    uint256 public constant MIN_GNOSIS_PAYLOAD_LENGTH = 292;

    uint256 public constant MIN_POLYGON_PAYLOAD_LENGTH = 164;

    address public immutable owner;

    address public immutable multisig;

    address public governor;

    uint8 public paused = 1;

    mapping(uint256 => bool) public mapAllowedTargetSelectorChainIds;

    mapping(address => uint256) public mapBridgeMediatorL1L2ChainIds;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L96

```solidity
File: governance/contracts/bridges/HomeMediator.sol


    uint256 public constant DEFAULT_DATA_LENGTH = 36;

    address public immutable AMBContractProxyHome;

    address public foreignGovernor;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L53

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

    address public constant orca = address"whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc";

    address public pool;

    address public pdaProgram;

    address public bridgedTokenMint;

    address public pdaBridgedTokenAccount;

    uint64 public pdaHeader = 0xd0f7407ae48fbcaa;

    bytes public constant pdaProgramSeed = "pdaProgram";

    bytes1 public pdaBump;

    int32 public constant minTickLowerIndex = -443632;

    int32 public constant maxTickLowerIndex = 443632;

    uint32 public numPositionAccounts;

    uint32 public firstAvailablePositionAccountIndex;

    uint64 public totalLiquidity;

    mapping(address => uint64) public mapPositionAccountLiquidity;

    mapping(address => address) public mapPositionAccountPdaAta;

    address[type(uint32).max] public positionAccounts;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L24

```solidity
File: governance/contracts/OLAS.sol

    uint256 public constant oneYear = 1 days * 365;

    uint256 public constant tenYearSupplyCap = 1_000_000_000e18;

    uint256 public constant maxMintCapFraction = 2;

    uint256 public immutable timeLaunch;

    address public owner;

    address public minter;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L22

```solidity
File: tokenomics/contracts/Tokenomics.sol

    address public owner;

    uint96 public maxBond;

    address public olas;

    uint96 public inflationPerSecond;

    address public treasury;

    uint96 public veOLASThreshold;

    address public depository;

    uint96 public effectiveBond;

    address public dispenser;

    uint72 public codePerDev;

    uint8 public currentYear;

    bytes1 public tokenomicsParametersUpdated;

    uint8 internal _locked;

    address public componentRegistry;

    uint64 public epsilonRate;

    uint32 public epochLen;

    address public agentRegistry;

    uint96 public nextVeOLASThreshold;

    address public serviceRegistry;

    uint32 public epochCounter;

    uint32 public timeLaunch;

    uint32 public nextEpochLen;

    address public ve;

    uint72 public devsPerCapital;

    address public donatorBlacklist;

    uint32 public lastDonationBlockNumber;

    mapping(uint256 => uint256) public mapServiceAmounts;

    mapping(address => uint256) public mapOwnerRewards;

    mapping(address => uint256) public mapOwnerTopUps;

    mapping(uint256 => TokenomicsPoint) public mapEpochTokenomics;

    mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;

    mapping(address => bool) public mapNewOwners;

    mapping(uint256 => mapping(uint256 => IncentiveBalances)) public mapUnitIncentives;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L139

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

    string public constant VERSION = "1.0.1";

    bytes32 public constant PROXY_TOKENOMICS = 0xbd5523e7c3b6a94aa0e3b24d1120addc2f95c7029e097b466b2bedc8d4b4362f;

    uint256 public constant ONE_YEAR = 1 days * 365;

    uint256 public constant MIN_EPOCH_LENGTH = 1 weeks;

    uint256 public constant MIN_PARAM_VALUE = 1e14;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L11

```solidity
File: tokenomics/contracts/Treasury.sol

    address public constant ETH_TOKEN_ADDRESS = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

    address public owner;

    uint96 public ETHFromServices;

    address public olas;

    uint96 public ETHOwned;

    address public tokenomics;

    uint96 public minAcceptedETH = 0.065 ether;

    address public depository;

    uint8 public paused = 1;

    uint8 internal _locked;

    address public dispenser;

    mapping(address => uint256) public mapTokenReserves;

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

## [G-05] Shorten arrays with inline assembly

**Issue Description**\
When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating
storage.

**Proposed Optimization**\
Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new
array.

**Estimated Gas Savings**\
Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending
on original length.

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

Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas
savings.

**Attachments**

- **Code Snippets**

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

## [G-06] Make 3 event parameters indexed when possible

**Issue Description**\
Events allow logging information on-chain for analytics/tracing purposes. However, emitting unindexed parameters is inefficient
as it increases the size of the transaction logs linearly with the data size.

**Proposed Optimization**\
Review all events and make the first 3 non-address parameters indexed if possible, to optimize for gas efficiency of log
encoding. If an event has fewer than 3 parameters, make them all indexed.

**Estimated Gas Savings**\
Each indexed parameter saves approximately 200 gas vs an unindexed parameter by reducing log encoding size. With many events,
this optimization can add up to significant gas savings.

**Attachments**

- **Code Snippets**

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

## [G-07] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are Solmate and Solady.

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

## [G-08] Expressions for constatn values such as a call to KECCAK256(), should use immutable rather than constant

**Issue Description**\
The variable SCHEDULE is declared as constant but its value is computed from a call to keccak256(). Since this value does not change, it should be declared as immutable to properly indicate that its value is computed once at contract deployment rather than being a true constant.

**Proposed Optimization**\
Change the declaration of SCHEDULE from constant to immutable:

```solidity
bytes4 public immutable SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));
```

**Estimated Gas Savings**\
Changing constant to immutable helps properly indicate that the value of SCHEDULE is computed at contract deployment rather than being a true constant. This improves clarity without impacting gas costs.

**Attachments**

- **Code Snippets**

```solidity
File: governance/contracts/multisigs/GuardCM.sol

97  bytes4 public immutable SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));


99    bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));

101    bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));

103    bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));

105    bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L96-L105

## [G-09] Use nested if and, avoid multiple check combinations &&

**Issue Description**\
The code snippets are checking multiple conditions using && operators. This can be optimized by replacing with nested if statements to avoid unnecessary checks.

**Proposed Optimization**\

1. For MaxHeap.sol L102:

   ```solidity
   if (pos >= (size / 2)) {
   if (pos <= size) return;
   }
   ```

2. For ERC20TokenEmitter.sol L201:

   ```solidity
   if (totalTokensForCreators > 0) {
   if (creatorsAddress != address(0)) {
       // code
   }
   }
   ```

3. For AuctionHouse.sol L383:

   ```solidity
   if (creatorsShare > 0) {
   if (entropyRateBps > 0) {
   // code
   }
   }
   ```

**Estimated Gas Savings**\
Replacing && with nested ifs avoids evaluating the second condition unnecessarily if the first fails, which can provide marginal gas savings.

**Attachments**

- **Code Snippets**

```solidity
File: tokenomics/contracts/Depository.sol

404            if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L404

```solidity
File: governance/contracts/multisigs/GuardCM.sol

519            if (chainId != 100 && chainId != 137 && chainId != 10200 && chainId != 80001) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

376        if (numPositions > 0 && amountLeft > 0) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L376

```solidity
File: tokenomics/contracts/Tokenomics.sol

529        if (_epsilonRate > 0 && _epsilonRate <= 17e18) {

536        if (uint32(_epochLen) >= MIN_EPOCH_LENGTH && uint32(_epochLen) <= ONE_YEAR) {

750                        if (topUpEligible && incentiveFlags[unitType + 2]) {

801        if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) {

1138            if (lastEpoch > 0 && lastEpoch < curEpoch) {

1206            if (lastEpoch > 0 && lastEpoch < curEpoch) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L529

```solidity
File: governance/contracts/veOLAS.sol

188            if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) {

192            if (newLocked.endTime > block.timestamp && newLocked.amount > 0) {

306            if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188

```solidity
File: governance/contracts/wveOLAS.sol

215        if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) {

235        if (uPoint.blockNumber > 0 && blockNumber >= uPoint.blockNumber) {
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L215

## [G-10] Use selfbalance() instead of address(this).balance for contract balance checks

**Issue Description**\
The code is directly using address(this).balance to check the contract's ether balance, which can be inefficient in some cases.

**Proposed Optimization**\
Replace address(this).balance with selfbalance() where possible. The selfbalance() function from the Yul layer provides a direct reference to the contract's balance without having to perform an external call.

**Estimated Gas Savings**\
selfbalance() can save 200-400 gas compared to address(this).balance depending on the scenario. The savings comes from avoiding the external call.

**Attachments**

- **Code Snippets**

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

## [G-11] Use pre-increment and pre-decrement instead of +1/-1

**Issue Description**\
The code is using +1 and -1 to increment and decrement variables.

**Proposed Optimization**\
Replace instances of +1 with ++ for pre-decrement, and -1 with -- for pre-increment.

**Estimated Gas Savings**\
pre-increment/decrement can save 2 gas per operation compared to adding/subtracting 1.

**Attachments**

- **Code Snippets**

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

## [G-12] Expensive operation inside a for loop

**Proposed Optimization**\
Move expensive external calls out of the loop by:

- Computing splits & recipients arrays first
- Making a single bulk ETH transfer to the ERC20 contract
- Having ERC20 contract distribute tokens to recipients

**Estimated Gas Savings**\
Reduces external calls from O(n) to O(1). Can save >10k gas for larger creator lists.

**Attachments**

- **Code Snippets**

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

## [G-13] REVERT() strings longer than 32 bytes cost extra Gas

**Issue Description**\
The EVM has a 32 byte limit for data inside transactions. Require and revert strings longer than this will be truncated and wrapped in a keccak256 hash, increasing gas costs.

**Proposed Optimization**\
For require and revert messages, keep strings less than or equal to 32 bytes to avoid the extra hashing overhead. Alternatively, omit the messages altogether to save even more gas.

**Estimated Gas Savings**\
Each string hashed due to length costs an additional ~200 gas. For contracts with many validation checks, trimming require/revert strings could save thousands of gas.

**Attachments**

- **Code Snippets**

```solidity
File: lockbox-solana/solidity/liquidity_lockbox.sol

154            revert("Wrong bridged token mint account");

224            revert("No liquidity on a provided token account");

244            revert("Wrong bridged token mint account");

342            revert ("Requested amount is too big for the total available liquidity");
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L154

## [G-14] Cache external calls outside of loop to avoid re-calling function on each iteration

**Issue Description**\
Calling external functions like STATICCALL inside a loop incurs a gas cost for each iteration that can be optimized.

**Proposed Optimization**\
Cache results of external calls made prior to loop in local variables, and reference the cache inside the loop body.

**Estimated Gas Savings**\
Avoiding a re-call of an external function saves 100 gas per loop iteration. Significant savings for medium/large loops.

```solidity
// Without caching:

for (uint i = 0; i < 10; i++) {
  // Call external func
}

// With caching:

uint cache = externalFunc();

for (uint i = 0; i < 10; i++) {
  // Use cache
}
```

**Attachments**

- **Code Snippets**

```solidity
File: tokenomics/contracts/Depository.sol

255        for (uint256 i = 0; i < numProducts; ++i) {
            uint256 productId = productIds[i];
            // Check if the product is still open by getting its supply amount
            uint256 supply = mapBondProducts[productId].supply;
            // The supply is greater than zero only if the product is active, otherwise it is already closed
            if (supply > 0) {
                // Refund unused OLAS supply from the product if it was not used by the product completely
                ITokenomics(tokenomics).refundFromBondProgram(supply);  >>>>>>>>>>>//@audit  cache external call
                address token = mapBondProducts[productId].token;
                delete mapBondProducts[productId];

                ids[numClosedProducts] = productIds[i];
                ++numClosedProducts;
                emit CloseProduct(token, productId, supply);
            }
        }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L255

```solidity
File: tokenomics/contracts/Tokenomics.sol

702        for (uint256 i = 0; i < numServices; ++i) {
            // Check if the service owner or donator stakes enough OLAS for its components / agents to get a top-up
            // If both component and agent owner top-up fractions are zero, there is no need to call external contract
            // functions to check each service owner veOLAS balance
            bool topUpEligible;
            if (incentiveFlags[2] || incentiveFlags[3]) {
                address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
                    IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
            }

            // Loop over component and agent Ids
            for (uint256 unitType = 0; unitType < 2; ++unitType) {
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
        }


808        for (uint256 i = 0; i < numServices; ++i) {
            // Check for the service Id existence
            if (!IServiceRegistry(serviceRegistry).exists(serviceIds[i])) {
                revert ServiceDoesNotExist(serviceIds[i]);
            }


1104        for (uint256 i = 0; i < 2; ++i) {
            registriesSupply[i] = IToken(registries[i]).totalSupply();
        }



1110        for (uint256 i = 0; i < unitIds.length; ++i) {
            // Check for the unit type to be component / agent only
            if (unitTypes[i] > 1) {
                revert Overflow(unitTypes[i], 1);
            }

            // Check that the unit Ids are in ascending order, not repeating, and no bigger than registries total supply
            if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {
                revert WrongUnitId(unitIds[i], unitTypes[i]);
            }
            lastIds[unitTypes[i]] = unitIds[i];

            // Check the component / agent Id ownership
            address unitOwner = IToken(registries[unitTypes[i]]).ownerOf(unitIds[i]);
            if (unitOwner != account) {
                revert OwnerOnly(unitOwner, account);
            }
        }


1180        for (uint256 i = 0; i < unitIds.length; ++i) {
            // Check for the unit type to be component / agent only
            if (unitTypes[i] > 1) {
                revert Overflow(unitTypes[i], 1);
            }

            // Check that the unit Ids are in ascending order, not repeating, and no bigger than registries total supply
            if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {
                revert WrongUnitId(unitIds[i], unitTypes[i]);
            }
            lastIds[unitTypes[i]] = unitIds[i];

            // Check the component / agent Id ownership
            address unitOwner = IToken(registries[unitTypes[i]]).ownerOf(unitIds[i]);
            if (unitOwner != account) {
                revert OwnerOnly(unitOwner, account);
            }
        }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L702

## [G-15] Access mappings directly rather than using accessor functions

**Issue Description**\
When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

**Proposed Optimization**\
Access mappings directly inside functions instead of through accessor functions whenever possible to remove the extra layer of indirection.

**Estimated Gas Savings**\
Reading a mapping value directly saves approximately `600` gas vs using an accessor function.

**Attachments**

- **Code Snippets**

```solidity
File: tokenomics/contracts/DonatorBlacklist.sol

82    function isDonatorBlacklisted(address account) external view returns (bool status) {
83        status = mapBlacklistedDonators[account];
84    }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L82

```solidity
File: tokenomics/contracts/Tokenomics.sol

1245    function getUnitPoint(uint256 epoch, uint256 unitType) external view returns (UnitPoint memory up) {
1246        up = mapEpochTokenomics[epoch].unitPoints[unitType];
1247    }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1037

```solidity
File: tokenomics/contracts/Treasury.sol

526    function isEnabled(address token) external view returns (bool enabled) {
527        enabled = mapEnabledTokens[token];
528    }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L137

```solidity
File: registries/contracts/UnitRegistry.sol

152    function getUnit(uint256 unitId) external view virtual returns (Unit memory unit) {
153        unit = mapUnits[unitId];
154    }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L152

```solidity
File: governance/contracts/veOLAS.sol

164    function getUserPoint(address account, uint256 idx) external view returns (PointVoting memory) {
165        return mapUserPoints[account][idx];
166    }
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L164

## [G-16] Functions Guaranteed to revert when called by normal users can be markd payable

**Issue Description**\
By default, Solidity checks that the call value is 0 for non-payable functions, incurring unnecessary gas costs for admin functions that do not expect/process ether.

**Proposed Optimization**\
For admin functions that only restrict access and do not process Ether, declare them as payable to avoid call value checks.

**Estimated Gas Savings**\
As noted, making an admin function payable saves gas by removing the unnecessary call value check opcodes.

This reduces contract size and deployment costs. Each removed opcode saves non-trivial amounts of gas.

For contracts with many access-restricted admin functions, this optimization can provide meaningful cumulative savings, especially as usage scales up over time.

**Attachments**

- **Code Snippets**

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

## [G-17] Custom errors are (usually) smaller than require statements

**Issue Description**\
Custom errors are cheaper than require statements with strings because of how custom errors are handled. Solidity stores only the first 4 bytes of the hash of the error signature and returns only that. This means during reverting, only 4 bytes needs to be stored in memory. In the case of string messages in require statements, Solidity has to store(in memory) and revert with at least 64 bytes.

```solidity
Here’s an example below.

// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract CustomError {
    error InvalidAmount();

    function withdraw(uint256 _amount) external pure {
        if (_amount > 10 ether) revert InvalidAmount();
    }
}

// This uses more gas than the above contract
contract NoCustomError {
    function withdraw(uint256 _amount) external pure {
        require(_amount <= 10 ether, "Error: Pass in a valid amount");
    }
}

```

**Proposed Optimization**\
Use custom errors defined with the error keyword instead of require statements with string messages where possible.

**Estimated Gas Savings**\
Reverting with a custom error uses only 4 bytes of storage, while require with a string uses at least 64 bytes.

This means custom errors can save ~60 bytes of storage per revert, translating to real gas savings.

For contracts with many validation checks, this adds up significantly over many transactions.
**Attachments**

- **Code Snippets**

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

## [G-18] <X> += <Y> costs more gas than  <X> = <X> + <Y> for state variables

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

## [G-19] STATE VARIABLES can be packed into fewer storage slots by truncating timestamp bytes

Optimize Storage with byte truncation for time related state variables

Certain state variables, particularly timestamps, can be safely stored using uint32. By optimizing these variables, contracts can utilize storage
more efficiently. This not only results in a reduction in the initial gas costs (due to fewer Gsset operations) but also provides savings in subsequent
read and write operations. Consider truncating the timestamp bytes for optimal storage use.

```solidity
File: governance/contracts/veOLAS.sol

99    uint64 internal constant WEEK = 1 weeks;

101    uint256 internal constant MAXTIME = 4 * 365 * 86400;

103    int128 internal constant IMAXTIME = 4 * 365 * 86400;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L98

```solidity
File: tokenomics/contracts/Depository.sol

75    uint256 public constant MIN_VESTING = 1 days;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L75

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

15    // One year in seconds
    uint256 public constant ONE_YEAR = 1 days * 365;
    // Minimum epoch length
    uint256 public constant MIN_EPOCH_LENGTH = 1 weeks;
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L15

## [G-20] Duplicated if() checks should be refactored to a modifier or function

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

## [G-21] Refactor Event to avoid emitting empty data

```solidity
File: governance/contracts/multisigs/GuardCM.sol

568        emit GuardUnpaused();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L348

```solidity
File: tokenomics/contracts/Treasury.sol


538        emit PauseTreasury();

549        emit UnpauseTreasury();
```

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L112

## [G-22]  abi.encode() is less efficient than abi.encodePacked()

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
