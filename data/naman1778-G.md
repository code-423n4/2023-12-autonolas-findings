## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

There are 2 instances of this issue in 2 files:

```
File: governance/contracts/bridges/FxERC20ChildTunnel.sol	

97: bytes memory message = abi.encode(msg.sender, to, amount);
```

    diff --git a/governance/contracts/bridges/FxERC20ChildTunnel.sol b/governance/contracts/bridges/FxERC20ChildTunnel.sol
    index 8caec63..e33149c 100644
    --- a/governance/contracts/bridges/FxERC20ChildTunnel.sol
    +++ b/governance/contracts/bridges/FxERC20ChildTunnel.sol
    @@ -94,7 +94,7 @@ contract FxERC20ChildTunnel is FxBaseChildTunnel {
             }

             // Encode message for root: (address, address, uint256)
    -        bytes memory message = abi.encode(msg.sender, to, amount);
    +        bytes memory message = abi.encodePacked(msg.sender, to, amount);
             // Send message to root
             _sendMessageToRoot(message);

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol

```
File: governance/contracts/bridges/FxERC20RootTunnel.sol	

93: bytes memory message = abi.encode(msg.sender, to, amount);
```
    diff --git a/governance/contracts/bridges/FxERC20RootTunnel.sol b/governance/contracts/bridges/FxERC20RootTunnel.sol
    index fb0fd7b..27b65d8 100755
    --- a/governance/contracts/bridges/FxERC20RootTunnel.sol
    +++ b/governance/contracts/bridges/FxERC20RootTunnel.sol
    @@ -90,7 +90,7 @@ contract FxERC20RootTunnel is FxBaseRootTunnel {
             }

             // Encode message for child: (address, address, uint256)
    -        bytes memory message = abi.encode(msg.sender, to, amount);
    +        bytes memory message = abi.encodePacked(msg.sender, to, amount);
             // Send message to child
             _sendMessageToChild(message);

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There are 19 instances of this issue in 12 files:

```
File: governance/contracts/veOLAS.sol	

762: return interfaceId == type(IERC20).interfaceId || interfaceId == type(IVotes).interfaceId ||
763:     interfaceId == type(IERC165).interfaceId;
```

    diff --git a/governance/contracts/veOLAS.sol b/governance/contracts/veOLAS.sol
    index 6f14419..da43aae 100644
    --- a/governance/contracts/veOLAS.sol
    +++ b/governance/contracts/veOLAS.sol
    @@ -759,8 +759,8 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
         /// @param interfaceId A specified interface Id.
         /// @return True if this contract implements the interface defined by interfaceId.
         function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
    -        return interfaceId == type(IERC20).interfaceId || interfaceId == type(IVotes).interfaceId ||
    -            interfaceId == type(IERC165).interfaceId;
    +        return (((interfaceId ^ type(IERC20).interfaceId) & (interfaceId ^ (type(IVotes).interfaceId) &
    +            (interfaceId ^ type(IERC165).interfaceId)) == 0);
         }

         /// @dev Reverts the transfer of this token.

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol

```
File: governance/contracts/wveOLAS.sol	

147: if (_ve == address(0) || _token == address(0)) {
```

    diff --git a/governance/contracts/wveOLAS.sol b/governance/contracts/wveOLAS.sol
    index 7a55140..3ae722c 100644
    --- a/governance/contracts/wveOLAS.sol
    +++ b/governance/contracts/wveOLAS.sol
    @@ -144,7 +144,7 @@ contract wveOLAS {
         /// @param _token OLAS address.
         constructor(address _ve, address _token) {
             // Check for the zero address
    -        if (_ve == address(0) || _token == address(0)) {
    +        if (((_ve ^ address(0)) & (_token ^ address(0))) == 0) {
                 revert ZeroAddress();
             }
             ve = _ve;

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol

```
File: governance/contracts/multisigs/GuardCM.sol	

144: if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {

259: if (chainId == 100 || chainId == 10200) {

304: if (chainId == 137 || chainId == 80001) {

418: if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) {

453: if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) {

506: if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) {

513: if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0)) {
```

    diff --git a/governance/contracts/multisigs/GuardCM.sol b/governance/contracts/multisigs/GuardCM.sol
    index 4472342..140a46f 100644
    --- a/governance/contracts/multisigs/GuardCM.sol
    +++ b/governance/contracts/multisigs/GuardCM.sol
    @@ -141,7 +141,7 @@ contract GuardCM {
             address _governor
         ) {
             // Check for zero addresses
    -        if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {
    +        if (((_timelock ^ address(0)) & (_multisig ^ address(0)) & (_governor ^ address(0))) == 0) {
                 revert ZeroAddress();
             }
             owner = _timelock;
    @@ -256,7 +256,7 @@ contract GuardCM {
         ) internal
         {
             // Gnosis chains
    -        if (chainId == 100 || chainId == 10200) {
    +        if (((chainId ^ 100) & (chainId ^ 10200)) == 0) {
                 // Check the L1 initial selector
                 bytes4 functionSig = bytes4(data);
                 if (functionSig != REQUIRE_TO_PASS_MESSAGE) {
    @@ -301,7 +301,7 @@ contract GuardCM {
             }

             // Polygon chains
    -        if (chainId == 137 || chainId == 80001) {
    +        if (((chainId ^ 137) & (chainId ^ 80001)) == 0) {
                 // Check the L1 initial selector
                 bytes4 functionSig = bytes4(data);
                 if (functionSig != SEND_MESSAGE_TO_CHILD) {
    @@ -415,7 +415,7 @@ contract GuardCM {
                     bytes4 functionSig = bytes4(data);
                     // Check the schedule or scheduleBatch function authorized parameters
                     // All other functions are not checked for
    -                if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) {
    +                if (((functionSig ^ SCHEDULE) & (functionSig ^ SCHEDULE_BATCH)) == 0) {
                         // Data length is too short: need to have enough bytes for the schedule() function
                         // with one selector extracted from the payload
                         if (data.length < MIN_SCHEDULE_DATA_LENGTH) {
    @@ -450,7 +450,7 @@ contract GuardCM {
             }

             // Check array length
    -        if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) {
    +        if (((targets.length ^ selectors.length) | (targets.length ^ statuses.length) | (targets.length ^ chainIds.length)) != 0) {
                 revert WrongArrayLength(targets.length, selectors.length, statuses.length, chainIds.length);
             }

    @@ -503,14 +503,14 @@ contract GuardCM {
             }

             // Check for array correctness
    -        if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) {
    +        if (((bridgeMediatorL1s.length ^ bridgeMediatorL2s.length) | (bridgeMediatorL1s.length ^ chainIds.length)) != 0) {
                 revert WrongArrayLength(bridgeMediatorL1s.length, bridgeMediatorL2s.length, chainIds.length, chainIds.length);
             }

             // Link L1 and L2 bridge mediators, set L2 chain Ids
             for (uint256 i = 0; i < chainIds.length; ++i) {
                 // Check for zero addresses
    -            if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0)) {
    +            if (((bridgeMediatorL1s[i] ^ address(0)) & (bridgeMediatorL2s[i] ^ address(0))) == 0) {
                     revert ZeroAddress();
                 }

    @@ -604,4 +604,4 @@ contract GuardCM {
             // L2 chain Id occupies next 64 bits
             chainId = bridgeMediatorL2ChainId >> 160;
         }
    -}
    \ No newline at end of file
    +}

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol

```
File: governance/contracts/bridges/FxGovernorTunnel.sol	

64: if (_fxChild == address(0) || _rootGovernor == address(0)) {
```
    diff --git a/governance/contracts/bridges/FxGovernorTunnel.sol b/governance/contracts/bridges/FxGovernorTunnel.sol
    index 5b672e5..7e2d2a0 100644
    --- a/governance/contracts/bridges/FxGovernorTunnel.sol
    +++ b/governance/contracts/bridges/FxGovernorTunnel.sol
    @@ -61,7 +61,7 @@ contract FxGovernorTunnel is IFxMessageProcessor {
         /// @param _rootGovernor Root Governor address.
         constructor(address _fxChild, address _rootGovernor) {
             // Check fo zero addresses
    -        if (_fxChild == address(0) || _rootGovernor == address(0)) {
    +        if (((_fxChild ^ address(0)) & (_rootGovernor ^ address(0))) == 0) {
                 revert ZeroAddress();
             }

    @@ -166,4 +166,4 @@ contract FxGovernorTunnel is IFxMessageProcessor {
             // Emit received message
             emit MessageReceived(stateId, rootMessageSender, data);
         }
    -}
    \ No newline at end of file

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol

```
File: governance/contracts/bridges/HomeMediator.sol	

64: if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) {
```

    diff --git a/governance/contracts/bridges/HomeMediator.sol b/governance/contracts/bridges/HomeMediator.sol
    index e315798..e74e8da 100644
    --- a/governance/contracts/bridges/HomeMediator.sol
    +++ b/governance/contracts/bridges/HomeMediator.sol
    @@ -61,7 +61,7 @@ contract HomeMediator {
         /// @param _foreignGovernor Foreign Governor address (ETH).
         constructor(address _AMBContractProxyHome, address _foreignGovernor) {
             // Check fo zero addresses
    -        if (_AMBContractProxyHome == address(0) || _foreignGovernor == address(0)) {
    +        if (((_AMBContractProxyHome ^ address(0)) & (_foreignGovernor ^ address(0))) == 0) {
                 revert ZeroAddress();
             }

    @@ -166,4 +166,4 @@ contract HomeMediator {
             // Emit received message
             emit MessageReceived(governor, data);
         }
    -}
    \ No newline at end of file
    +}

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol

```
File: governance/contracts/bridges/FxERC20ChildTunnel.sol	

39: if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0)) {
```

    diff --git a/governance/contracts/bridges/FxERC20ChildTunnel.sol b/governance/contracts/bridges/FxERC20ChildTunnel.sol
    index 8caec63..f019ddb 100644
    --- a/governance/contracts/bridges/FxERC20ChildTunnel.sol
    +++ b/governance/contracts/bridges/FxERC20ChildTunnel.sol
    @@ -36,7 +36,7 @@ contract FxERC20ChildTunnel is FxBaseChildTunnel {
         /// @param _rootToken Corresponding L1 token address.
         constructor(address _fxChild, address _childToken, address _rootToken) FxBaseChildTunnel(_fxChild) {
             // Check for zero addresses
    -        if (_fxChild == address(0) || _childToken == address(0) || _rootToken == address(0)) {
    +        if (((_fxChild ^ address(0)) & (_childToken ^ address(0)) & (_rootToken ^ address(0))) == 0) {
                 revert ZeroAddress();
             }

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol

```
File: governance/contracts/bridges/FxERC20RootTunnel.sol	

42: if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
    43:     _rootToken == address(0)) {
```

    diff --git a/governance/contracts/bridges/FxERC20RootTunnel.sol b/governance/contracts/bridges/FxERC20RootTunnel.sol
    index fb0fd7b..001de0c 100755
    --- a/governance/contracts/bridges/FxERC20RootTunnel.sol
    +++ b/governance/contracts/bridges/FxERC20RootTunnel.sol
    @@ -39,8 +39,8 @@ contract FxERC20RootTunnel is FxBaseRootTunnel {
             FxBaseRootTunnel(_checkpointManager, _fxRoot)
         {
             // Check for zero addresses
    -        if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
    -            _rootToken == address(0)) {
    +        if (((_checkpointManager ^ address(0)) & (_fxRoot ^ address(0)) & (_childToken ^ address(0)) &
    +            (_rootToken ^ address(0))) == 0) {
                 revert ZeroAddress();
             }

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol

```
File: tokenomics/contracts/Depository.sol	

111: if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) {
```
    diff --git a/tokenomics/contracts/Depository.sol b/tokenomics/contracts/Depository.sol
    index f9c7d64..2156a7a 100644
    --- a/tokenomics/contracts/Depository.sol
    +++ b/tokenomics/contracts/Depository.sol
    @@ -108,7 +108,7 @@ contract Depository is IErrorsTokenomics {
             owner = msg.sender;

             // Check for at least one zero contract address
    -        if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) {
    +        if (((_olas ^ address(0)) & (_tokenomics ^ address(0)) & (_treasury ^ address(0)) & (_bondCalculator ^ address(0))) == 0) {
                 revert ZeroAddress();
             }
             olas = _olas;

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol

```
File: tokenomics/contracts/Dispenser.sol	

36: if (_tokenomics == address(0) || _treasury == address(0)) {
```
    diff --git a/tokenomics/contracts/Dispenser.sol b/tokenomics/contracts/Dispenser.sol
    index 9b8f4f5..3e5aa3e 100644
    --- a/tokenomics/contracts/Dispenser.sol
    +++ b/tokenomics/contracts/Dispenser.sol
    @@ -33,7 +33,7 @@ contract Dispenser is IErrorsTokenomics {
             _locked = 1;

             // Check for at least one zero contract address
    -        if (_tokenomics == address(0) || _treasury == address(0)) {
    +        if (((_tokenomics ^ address(0)) & (_treasury ^ address(0))) == 0) {
                 revert ZeroAddress();
             }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol

```
File: tokenomics/contracts/GenericBondCalculator.sol	

31: if (_olas == address(0) || _tokenomics == address(0)) {

82: if (token0 == olas || token1 == olas) {
```
    diff --git a/tokenomics/contracts/GenericBondCalculator.sol b/tokenomics/contracts/GenericBondCalculator.sol
    index 4a0286d..ac783b8 100644
    --- a/tokenomics/contracts/GenericBondCalculator.sol
    +++ b/tokenomics/contracts/GenericBondCalculator.sol
    @@ -28,7 +28,7 @@ contract GenericBondCalculator {
         /// @param _tokenomics Tokenomics contract address.
         constructor(address _olas, address _tokenomics) {
             // Check for at least one zero contract address
    -        if (_olas == address(0) || _tokenomics == address(0)) {
    +        if (((_olas ^ address(0)) & (_tokenomics ^ address(0))) == 0 ) {
                 revert ZeroAddress();
             }

    @@ -79,7 +79,7 @@ contract GenericBondCalculator {
                 // requires low gas
                 (reserve0, reserve1, ) = pair.getReserves();
                 // token0 != olas && token1 != olas, this should never happen
    -            if (token0 == olas || token1 == olas) {
    +            if (((token0 ^ olas) & (token1 ^ olas)) == 0) {
                     // If OLAS is in token0, assign its reserve to reserve1, otherwise the reserve1 is already correct
                     if (token0 == olas) {
                         reserve1 = reserve0;

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol

```
File: tokenomics/contracts/Tokenomics.sol	

283: if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
284:     _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
285:     _serviceRegistry == address(0)) {
```

    diff --git a/tokenomics/contracts/Tokenomics.sol b/tokenomics/contracts/Tokenomics.sol
    index 2ef2708..b73249a 100644
    --- a/tokenomics/contracts/Tokenomics.sol
    +++ b/tokenomics/contracts/Tokenomics.sol
    @@ -280,9 +280,9 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
             }

             // Check for at least one zero contract address
    -        if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
    -            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
    -            _serviceRegistry == address(0)) {
    +        if (((_olas ^ address(0)) & (_treasury ^ address(0)) & (_depository ^ address(0)) & (_dispenser ^ address(0)) &
    +            (_ve ^ address(0)) & (_componentRegistry ^ address(0)) & (_agentRegistry ^ address(0)) &
    +            (_serviceRegistry ^ address(0))) == 0) {
                 revert ZeroAddress();
             }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol

```
File: tokenomics/contracts/Treasury.sol	

100: if (_olas == address(0) || _tokenomics == address(0) || _depository == address(0) || _dispenser == address(0)) {
```
    diff --git a/tokenomics/contracts/Treasury.sol b/tokenomics/contracts/Treasury.sol
    index 516c631..cc0e87d 100644
    --- a/tokenomics/contracts/Treasury.sol
    +++ b/tokenomics/contracts/Treasury.sol
    @@ -97,7 +97,7 @@ contract Treasury is IErrorsTokenomics {
             _locked = 1;

             // Check for at least one zero contract address
    -        if (_olas == address(0) || _tokenomics == address(0) || _depository == address(0) || _dispenser == address(0)) {
    +        if (((_olas ^ address(0)) & (_tokenomics ^ address(0)) & (_depository ^ address(0)) & (_dispenser ^ address(0))) == 0) {
                 revert ZeroAddress();
             }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol%5D

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(5,10,16);
            c1.optimized(5,10,16);
        }
    }
    contract Contract0 {
        address a;
        function not_optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(x!=5 || y!=10 || z!=15){
                return true;
            }
        }
    }
    contract Contract1 {
        address a;
        function optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(((x^5) | (y^10) | (z^15)) != 0){
                return true;
            }
        }
    }
    

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 53705                                     | 300             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 622             | 622 | 622    | 622 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 49899                                     | 280             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 569             | 569 | 569    | 569 | 1       |

## [G-03] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 49 instances of this issue in 14 files:

```
File: governance/contracts/veOLAS.sol	

232: for (uint256 i = 0; i < 255; ++i) {

561: for (uint256 i = 0; i < 128; ++i) {

693: for (uint256 i = 0; i < 255; ++i) {
```

    diff --git a/governance/contracts/veOLAS.sol b/governance/contracts/veOLAS.sol
    index 6f14419..d4b59d4 100644
    --- a/governance/contracts/veOLAS.sol
    +++ b/governance/contracts/veOLAS.sol
    @@ -229,7 +229,7 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
             {
                 // The timestamp is rounded by a week and < 2^64-1
                 uint64 tStep = (lastCheckpoint / WEEK) * WEEK;
    -            for (uint256 i = 0; i < 255; ++i) {
    +            for (uint256 i = 254; i >= 0 ; --i) {
                     // Hopefully it won't happen that this won't get used in 5 years!
                     // If it does, users will be able to withdraw but vote weight will be broken
                     // This is always practically < 2^64-1
    @@ -558,7 +558,7 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
             }

             // Binary search that will be always enough for 128-bit numbers
    -        for (uint256 i = 0; i < 128; ++i) {
    +        for (uint256 i = 127; i >= 0; --i) {
                 if ((minPointNumber + 1) > maxPointNumber) {
                     break;
                 }
    @@ -690,7 +690,7 @@ contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
         function _supplyLockedAt(PointVoting memory lastPoint, uint64 ts) internal view returns (uint256 vSupply) {
             // The timestamp is rounded and < 2^64-1
             uint64 tStep = (lastPoint.ts / WEEK) * WEEK;
    -        for (uint256 i = 0; i < 255; ++i) {
    +        for (uint256 i = 254; i >= 0; --i) {
                 // This is always practically < 2^64-1
                 unchecked {
                     tStep += WEEK;

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol
    
```
File: governance/contracts/multisigs/GuardCM.sol	

212: for (uint256 i = 0; i < data.length;) {

237: for (uint256 j = 0; j < payloadLength; ++j) {

273: for (uint256 i = 0; i < payload.length; ++i) {

292: for (uint256 i = 0; i < bridgePayload.length; ++i) {

318: for (uint256 i = 0; i < payload.length; ++i) {

340: for (uint256 i = 0; i < payload.length; ++i) {

360: for (uint i = 0; i < targets.length; ++i) {

458: for (uint256 i = 0; i < targets.length; ++i) {

511: for (uint256 i = 0; i < chainIds.length; ++i) {
```

    diff --git a/governance/contracts/multisigs/GuardCM.sol b/governance/contracts/multisigs/GuardCM.sol
    index 4472342..cd8ac69 100644
    --- a/governance/contracts/multisigs/GuardCM.sol
    +++ b/governance/contracts/multisigs/GuardCM.sol
    @@ -209,7 +209,7 @@ contract GuardCM {
         function _verifyBridgedData(bytes memory data, uint256 chainId) internal {
             // Unpack and process the data
             // We need to skip first 12 bytes as those are zeros from encoding
    -        for (uint256 i = 0; i < data.length;) {
    +        for (uint256 i = data.length - 1; i >= 0 ;) {
                 address target;
                 uint32 payloadLength;
                 // solhint-disable-next-line no-inline-assembly
    @@ -234,11 +234,11 @@ contract GuardCM {

                 // Get the payload
                 bytes memory payload = new bytes(payloadLength);
    -            for (uint256 j = 0; j < payloadLength; ++j) {
    +            for (uint256 j = payloadLength - 1; j >= 0; --j) {
                     payload[j] = data[i + j];
                 }
                 // Offset the data by the payload number of bytes
    -            i += payloadLength;
    +            i -= payloadLength;

                 // Verify the scope of the data
                 _verifyData(target, payload, chainId);
    @@ -270,7 +270,7 @@ contract GuardCM {

                 // Copy the data without the selector
                 bytes memory payload = new bytes(data.length - SELECTOR_DATA_LENGTH);
    -            for (uint256 i = 0; i < payload.length; ++i) {
    +            for (uint256 i = payload.length - 1; i >= 0 ; --i) {
                     payload[i] = data[i + 4];
                 }

    @@ -289,7 +289,7 @@ contract GuardCM {

                 // Copy the data without a selector
                 bytes memory bridgePayload = new bytes(mediatorPayload.length - SELECTOR_DATA_LENGTH);
    -            for (uint256 i = 0; i < bridgePayload.length; ++i) {
    +            for (uint256 i = bridgePayload.length - 1; i >= 0 ; --i) {
                     bridgePayload[i] = mediatorPayload[i + SELECTOR_DATA_LENGTH];
                 }

    @@ -315,7 +315,7 @@ contract GuardCM {

                 // Copy the data without the selector
                 bytes memory payload = new bytes(data.length - SELECTOR_DATA_LENGTH);
    -            for (uint256 i = 0; i < payload.length; ++i) {
    +            for (uint256 i = payload.length - 1; i >= 0 ; --i) {
                     payload[i] = data[i + SELECTOR_DATA_LENGTH];
                 }

    @@ -337,7 +337,7 @@ contract GuardCM {
         function _verifySchedule(bytes memory data, bytes4 selector) internal {
             // Copy the data without the selector
             bytes memory payload = new bytes(data.length - SELECTOR_DATA_LENGTH);
    -        for (uint256 i = 0; i < payload.length; ++i) {
    +        for (uint256 i = payload.length - 1; i >= 0 ; -i) {
                 payload[i] = data[i + 4];
             }

    @@ -357,7 +357,7 @@ contract GuardCM {
             }

             // Traverse all the schedule targets and selectors extracted from calldatas
    -        for (uint i = 0; i < targets.length; ++i) {
    +        for (uint i = targets.length - 1; i >= 0; -i) {
                 // Get the bridgeMediatorL2 and L2 chain Id, if any
                 uint256 bridgeMediatorL2ChainId = mapBridgeMediatorL1L2ChainIds[targets[i]];
                 // bridgeMediatorL2 occupies first 160 bits
    @@ -455,7 +455,7 @@ contract GuardCM {
             }

             // Traverse all the targets and selectors to build their paired values
    -        for (uint256 i = 0; i < targets.length; ++i) {
    +        for (uint256 i = targets.length - 1; i >= 0 ; --i) {
                 // Check for zero address targets
                 if (targets[i] == address(0)) {
                     revert ZeroAddress();
    @@ -508,7 +508,7 @@ contract GuardCM {
             }

             // Link L1 and L2 bridge mediators, set L2 chain Ids
    -        for (uint256 i = 0; i < chainIds.length; ++i) {
    +        for (uint256 i = chainIds.length - 1; i >= 0 ; --i) {
                 // Check for zero addresses
                 if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0)) {
                     revert ZeroAddress();
    @@ -604,4 +604,4 @@ contract GuardCM {
             // L2 chain Id occupies next 64 bits
             chainId = bridgeMediatorL2ChainId >> 160;
         }
    -}


https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol

```
File: governance/contracts/bridges/FxGovernorTunnel.sol	

125: for (uint256 i = 0; i < dataLength;) {

153: for (uint256 j = 0; j < payloadLength; ++j) {
```
    diff --git a/governance/contracts/bridges/FxGovernorTunnel.sol b/governance/contracts/bridges/FxGovernorTunnel.sol
    index 5b672e5..f61eaa5 100644
    --- a/governance/contracts/bridges/FxGovernorTunnel.sol
    +++ b/governance/contracts/bridges/FxGovernorTunnel.sol
    @@ -122,7 +122,7 @@ contract FxGovernorTunnel is IFxMessageProcessor {
             }

             // Unpack and process the data
    -        for (uint256 i = 0; i < dataLength;) {
    +        for (uint256 i = dataLength  - 1; i >= 0 ;) {
                 address target;
                 uint96 value;
                 uint32 payloadLength;
    @@ -150,11 +150,11 @@ contract FxGovernorTunnel is IFxMessageProcessor {

                 // Get the payload
                 bytes memory payload = new bytes(payloadLength);
    -            for (uint256 j = 0; j < payloadLength; ++j) {
    +            for (uint256 j = payloadLength - 1; j >= 0 ; --j) {
                     payload[j] = data[i + j];
                 }
                 // Offset the data by the payload number of bytes
    -            i += payloadLength;
    +            i -= payloadLength;

                 // Call the target with the provided payload
                 (bool success, ) = target.call{value: value}(payload);
    @@ -166,4 +166,4 @@ contract FxGovernorTunnel is IFxMessageProcessor {
             // Emit received message
             emit MessageReceived(stateId, rootMessageSender, data);
         }
    -}

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol

```
File: governance/contracts/bridges/HomeMediator.sol	

125: for (uint256 i = 0; i < dataLength;) {

153: for (uint256 j = 0; j < payloadLength; ++j) {
```
    diff --git a/governance/contracts/bridges/HomeMediator.sol b/governance/contracts/bridges/HomeMediator.sol
    index e315798..62db386 100644
    --- a/governance/contracts/bridges/HomeMediator.sol
    +++ b/governance/contracts/bridges/HomeMediator.sol
    @@ -122,7 +122,7 @@ contract HomeMediator {
             }

             // Unpack and process the data
    -        for (uint256 i = 0; i < dataLength;) {
    +        for (uint256 i = dataLength - 1 ; i >= 0 ;) {
                 address target;
                 uint96 value;
                 uint32 payloadLength;
    @@ -150,11 +150,11 @@ contract HomeMediator {

                 // Get the payload
                 bytes memory payload = new bytes(payloadLength);
    -            for (uint256 j = 0; j < payloadLength; ++j) {
    +            for (uint256 j = payloadLength - 1; j >= 0 ; --j) {
                     payload[j] = data[i + j];
                 }
                 // Offset the data by the payload number of bytes
    -            i += payloadLength;
    +            i -= payloadLength;

                 // Call the target with the provided payload
                 (bool success, ) = target.call{value: value}(payload);
    @@ -166,4 +166,4 @@ contract HomeMediator {
             // Emit received message
             emit MessageReceived(governor, data);
         }
    -}


https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol

```
File: registries/contracts/UnitRegistry.sol	

98: for (uint256 i = 0; i < numSubComponents; ++i) {

211: for (uint32 i = 0; i < numUnits; ++i) {

227: for (counter = 0; counter < maxNumComponents; ++counter) {

234: for (uint32 i = 0; i < numUnits; ++i) {

262: for (uint32 i = 0; i < counter; ++i) {
```
    diff --git a/registries/contracts/UnitRegistry.sol b/registries/contracts/UnitRegistry.sol
    index 80b3235..d4b6076 100644
    --- a/registries/contracts/UnitRegistry.sol
    +++ b/registries/contracts/UnitRegistry.sol
    @@ -95,7 +95,7 @@ abstract contract UnitRegistry is GenericRegistry {
             if (unitType == UnitType.Component) {
                 uint256 numSubComponents = subComponentIds.length;
                 uint32[] memory addSubComponentIds = new uint32[](numSubComponents + 1);
    -            for (uint256 i = 0; i < numSubComponents; ++i) {
    +            for (uint256 i = numSubComponents - 1; i >= 0 ; --i) {
                     addSubComponentIds[i] = subComponentIds[i];
                 }
                 // Adding self component Id
    @@ -208,7 +208,7 @@ abstract contract UnitRegistry is GenericRegistry {

             // Get total possible number of components and lists of components
             uint32 maxNumComponents;
    -        for (uint32 i = 0; i < numUnits; ++i) {
    +        for (uint32 i = numUnits - 1; i >= 0; --i) {
                 // Get subcomponents for each unit Id based on the subcomponentsFromType
                 components[i] = _getSubComponents(subcomponentsFromType, unitIds[i]);
                 numComponents[i] = uint32(components[i].length);
    @@ -224,14 +224,14 @@ abstract contract UnitRegistry is GenericRegistry {
             // Overall component counter
             uint32 counter;
             // Iterate until we process all components, at the maximum of the sum of all the components in all units
    -        for (counter = 0; counter < maxNumComponents; ++counter) {
    +        for (counter = maxNumComponents - 1; counter >= 0 ; -counter) {
                 // Index of a minimal component
                 uint32 minIdxComponent;
                 // Amount of components identified as the next minimal component number
                 uint32 numComponentsCheck;
                 uint32 tryMinComponent = type(uint32).max;
                 // Assemble an array of all first components from each component array
    -            for (uint32 i = 0; i < numUnits; ++i) {
    +            for (uint32 i = numUnits - 1; i >= 0; --i) {
                     // Either get a component that has a higher id than the last one ore reach the end of the processed Ids
                     for (; processedComponents[i] < numComponents[i]; ++processedComponents[i]) {
                         if (minComponent < components[i][processedComponents[i]]) {
    @@ -259,7 +259,7 @@ abstract contract UnitRegistry is GenericRegistry {

             // Return the exact set of found subcomponents with the counter length
             subComponentIds = new uint32[](counter);
    -        for (uint32 i = 0; i < counter; ++i) {
    +        for (uint32 i = counter - 1; i >= 0; --i) {
                 subComponentIds[i] = allComponents[i];
             }
         }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol

```
File: registries/contracts/ ComponentRegistry.sol	
 
29: for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
```

    diff --git a/registries/contracts/ComponentRegistry.sol b/registries/contracts/ComponentRegistry.sol
    index 43a6dd4..e2a59f8 100644
    --- a/registries/contracts/ComponentRegistry.sol
    +++ b/registries/contracts/ComponentRegistry.sol
    @@ -26,7 +26,7 @@ contract ComponentRegistry is UnitRegistry {
         /// @param maxComponentId Maximum component Id.
         function _checkDependencies(uint32[] memory dependencies, uint32 maxComponentId) internal virtual override {
             uint32 lastId;
    -        for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
    +        for (uint256 iDep = dependencies.length - 1; iDep >= 0 ; --iDep) {
                 if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) {
                     revert ComponentNotFound(dependencies[iDep]);
                 }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol

```
File: registries/contracts/AgentRegistry.sol	

40: for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
```
    diff --git a/registries/contracts/AgentRegistry.sol b/registries/contracts/AgentRegistry.sol
    index e069c6d..61ee4d9 100644
    --- a/registries/contracts/AgentRegistry.sol
    +++ b/registries/contracts/AgentRegistry.sol
    @@ -37,7 +37,7 @@ contract AgentRegistry is UnitRegistry {
             // Get the components total supply
             uint32 componentTotalSupply = uint32(IRegistry(componentRegistry).totalSupply());
             uint32 lastId;
    -        for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
    +        for (uint256 iDep = dependencies.length - 1; iDep >= 0 ; --iDep) {
                 if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) {
                     revert ComponentNotFound(dependencies[iDep]);
                 }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol
```
File: registries/contracts/multisigs/GnosisSafeMultisig.sol	
 
79: for (uint256 i = 0; i < payloadLength; ++i) {
```
    diff --git a/registries/contracts/multisigs/GnosisSafeMultisig.sol b/registries/contracts/multisigs/GnosisSafeMultisig.sol
    index 207cefc..75ebf9e 100644
    --- a/registries/contracts/multisigs/GnosisSafeMultisig.sol
    +++ b/registries/contracts/multisigs/GnosisSafeMultisig.sol
    @@ -76,7 +76,7 @@ contract GnosisSafeMultisig {
                 if (dataLength > DEFAULT_DATA_LENGTH) {
                     uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
                     payload = new bytes(payloadLength);
    -                for (uint256 i = 0; i < payloadLength; ++i) {
    +                for (uint256 i = payloadLength - 1; i >= 0 ; --i) {
                         payload[i] = data[i + DEFAULT_DATA_LENGTH];
                     }
                 }
    @@ -105,4 +105,4 @@ contract GnosisSafeMultisig {
             // Create a gnosis safe multisig via the proxy factory
             multisig = IGnosisSafeProxyFactory(gnosisSafeProxyFactory).createProxyWithNonce(gnosisSafe, safeParams, nonce);
         }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol

```
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

113: for (uint256 i = 0; i < payloadLength; ++i) {

139: for (uint256 i = 0; i < numOwners; ++i) {
```

    diff --git a/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol b/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol
    index 60024d4..71b9ebc 100644
    --- a/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol
    +++ b/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol
    @@ -110,7 +110,7 @@ contract GnosisSafeSameAddressMultisig {
             if (dataLength > DEFAULT_DATA_LENGTH) {
                 uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
                 bytes memory payload = new bytes(payloadLength);
    -            for (uint256 i = 0; i < payloadLength; ++i) {
    +            for (uint256 i = payloadLength - 1; i >= 0; --i) {
                     payload[i] = data[i + DEFAULT_DATA_LENGTH];
                 }

    @@ -136,10 +136,10 @@ contract GnosisSafeSameAddressMultisig {
             // The owners' addresses in the multisig itself are stored in reverse order compared to how they were added:
             // https://etherscan.io/address/0xd9db270c1b5e3bd161e8c8503c55ceabee709552#code#F6#L56
             // Thus, the check must be carried out accordingly.
    -        for (uint256 i = 0; i < numOwners; ++i) {
    +        for (uint256 i = numOwners - 1; i >= 0 ; --i) {
                 if (owners[i] != checkOwners[numOwners - i - 1]) {
                     revert WrongOwner(owners[i]);
                 }
             }
         }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

```
File: tokenomics/contracts/Depository.sol	

255: for (uint256 i = 0; i < numProducts; ++i) {

274: for (uint256 i = 0; i < numClosedProducts; ++i) {

357: for (uint256 i = 0; i < bondIds.length; ++i) {

402: for (uint256 i = 0; i < numProducts; ++i) {

413: for (uint256 i = 0; i < numProducts; ++i) {

448: for (uint256 i = 0; i < numBonds; ++i) {

468: for (uint256 i = 0; i < numBonds; ++i) {
```
    diff --git a/tokenomics/contracts/Depository.sol b/tokenomics/contracts/Depository.sol
    index f9c7d64..267c7b5 100644
    --- a/tokenomics/contracts/Depository.sol
    +++ b/tokenomics/contracts/Depository.sol
    @@ -252,7 +252,7 @@ contract Depository is IErrorsTokenomics {
             uint256[] memory ids = new uint256[](numProducts);
             uint256 numClosedProducts;
             // Traverse to close all possible products
    -        for (uint256 i = 0; i < numProducts; ++i) {
    +        for (uint256 i = numProducts - 1; i >= 0 ; --i) {
                 uint256 productId = productIds[i];
                 // Check if the product is still open by getting its supply amount
                 uint256 supply = mapBondProducts[productId].supply;
    @@ -271,7 +271,7 @@ contract Depository is IErrorsTokenomics {

             // Get the correct array size of closed product Ids
             closedProductIds = new uint256[](numClosedProducts);
    -        for (uint256 i = 0; i < numClosedProducts; ++i) {
    +        for (uint256 i = numClosedProducts - 1; i >= 0 ; --i) {
                 closedProductIds[i] = ids[i];
             }
         }
    @@ -354,7 +354,7 @@ contract Depository is IErrorsTokenomics {
         /// #if_succeeds {:msg "payouts are zeroed"} forall (uint k in bondIds) mapUserBonds[bondIds[k]].payout == 0;
         /// #if_succeeds {:msg "maturities are zeroed"} forall (uint k in bondIds) mapUserBonds[bondIds[k]].maturity == 0;
         function redeem(uint256[] memory bondIds) external returns (uint256 payout) {
    -        for (uint256 i = 0; i < bondIds.length; ++i) {
    +        for (uint256 i = bondIds.length - 1; i >= 0 ; --i) {
                 // Get the amount to pay and the maturity status
                 uint256 pay = mapUserBonds[bondIds[i]].payout;
                 bool matured = block.timestamp >= mapUserBonds[bondIds[i]].maturity;
    @@ -399,7 +399,7 @@ contract Depository is IErrorsTokenomics {
             bool[] memory positions = new bool[](numProducts);
             uint256 numSelectedProducts;
             // Traverse to find requested products
    -        for (uint256 i = 0; i < numProducts; ++i) {
    +        for (uint256 i = numProducts - 1; i >= 0; -i) {
                 // Product is always active if its supply is not zero, and inactive otherwise
                 if ((active && mapBondProducts[i].supply > 0) || (!active && mapBondProducts[i].supply == 0)) {
                     positions[i] = true;
    @@ -410,7 +410,7 @@ contract Depository is IErrorsTokenomics {
             // Form active or inactive products index array
             productIds = new uint256[](numSelectedProducts);
             uint256 numPos;
    -        for (uint256 i = 0; i < numProducts; ++i) {
    +        for (uint256 i = numProducts - 1; i >= 0; --i) {
                 if (positions[i]) {
                     productIds[numPos] = i;
                     ++numPos;
    @@ -445,7 +445,7 @@ contract Depository is IErrorsTokenomics {
             uint256 numBonds = bondCounter;
             bool[] memory positions = new bool[](numBonds);
             // Record the bond number if it belongs to the account address and was not yet redeemed
    -        for (uint256 i = 0; i < numBonds; ++i) {
    +        for (uint256 i = numBonds - 1; i >= 0; --i) {
                 // Check if the bond belongs to the account
                 // If not and the address is zero, the bond was redeemed or never existed
                 if (mapUserBonds[i].account == account) {
    @@ -465,7 +465,7 @@ contract Depository is IErrorsTokenomics {
             // Form pending bonds index array
             bondIds = new uint256[](numAccountBonds);
             uint256 numPos;
    -        for (uint256 i = 0; i < numBonds; ++i) {
    +        for (uint256 i = numBonds - 1; i >= 0; --i) {
                 if (positions[i]) {
                     bondIds[numPos] = i;
                     ++numPos;

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol

```
File: tokenomics/contracts/DonatorBlacklist.sol	

67: for (uint256 i = 0; i < accounts.length; ++i) {
```
    diff --git a/tokenomics/contracts/DonatorBlacklist.sol b/tokenomics/contracts/DonatorBlacklist.sol
    index 00d84f4..bc48378 100644
    --- a/tokenomics/contracts/DonatorBlacklist.sol
    +++ b/tokenomics/contracts/DonatorBlacklist.sol
    @@ -64,7 +64,7 @@ contract DonatorBlacklist {
                 revert WrongArrayLength(accounts.length, statuses.length);
             }

    -        for (uint256 i = 0; i < accounts.length; ++i) {
    +        for (uint256 i = accounts.length - 1; i >= 0 ; --i) {
                 // Check for the zero address
                 if (accounts[i] == address(0)) {
                     revert ZeroAddress();

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol
```
File: tokenomics/contracts/Tokenomics.sol	

702: for (uint256 i = 0; i < numServices; ++i) {

714: for (uint256 unitType = 0; unitType < 2; ++unitType) {

728: for (uint256 j = 0; j < numServiceUnits; ++j) {

758: for (uint256 j = 0; j < numServiceUnits; ++j) {

808: for (uint256 i = 0; i <  numServices; ++i) {

978: for (uint256 i = 0; i < 2; ++i) {

1104: for (uint256 i = 0; i < 2; ++i) {

1110: for (uint256 i = 0; i < unitIds.length; ++i) {

1132: for (uint256 i = 0; i < unitIds.length; ++i) {

1174: for (uint256 i = 0; i < 2; ++i) {

1180: for (uint256 i = 0; i < unitIds.length; ++i) {

1202: for (uint256 i = 0; i < unitIds.length; ++i) {
```

    diff --git a/tokenomics/contracts/Tokenomics.sol b/tokenomics/contracts/Tokenomics.sol
    index 2ef2708..a7be6ff 100644
    --- a/tokenomics/contracts/Tokenomics.sol
    +++ b/tokenomics/contracts/Tokenomics.sol
    @@ -699,7 +699,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
             // Get the number of services
             uint256 numServices = serviceIds.length;
             // Loop over service Ids to calculate their partial contributions
    -        for (uint256 i = 0; i < numServices; ++i) {
    +        for (uint256 i = numServices - 1; i >= 0 ; -i) {
                 // Check if the service owner or donator stakes enough OLAS for its components / agents to get a top-up
                 // If both component and agent owner top-up fractions are zero, there is no need to call external contract
                 // functions to check each service owner veOLAS balance
    @@ -711,7 +711,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
                 }

                 // Loop over component and agent Ids
    -            for (uint256 unitType = 0; unitType < 2; ++unitType) {
    +            for (uint256 unitType = 1; unitType >= 0 ; --unitType) {
                     // Get the number and set of units in the service
                     (uint256 numServiceUnits, uint32[] memory serviceUnitIds) = IServiceRegistry(serviceRegistry).
                         getUnitIdsOfService(IServiceRegistry.UnitType(unitType), serviceIds[i]);
    @@ -725,7 +725,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
                         // The amount has to be adjusted for the number of units in the service
                         uint96 amount = uint96(amounts[i] / numServiceUnits);
                         // Accumulate amounts for each unit Id
    -                    for (uint256 j = 0; j < numServiceUnits; ++j) {
    +                    for (uint256 j = numServiceUnits - 1; j >= 0; --j) {
                             // Get the last epoch number the incentives were accumulated for
                             uint256 lastEpoch = mapUnitIncentives[unitType][serviceUnitIds[j]].lastEpoch;
                             // Check if there were no donations in previous epochs and set the current epoch
    @@ -755,7 +755,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
                     }

                     // Record new units and new unit owners
    -                for (uint256 j = 0; j < numServiceUnits; ++j) {
    +                for (uint256 j = numServiceUnits - 1; j >= 0; --j) {
                         // Check if the component / agent is used for the first time
                         if (!mapNewUnits[unitType][serviceUnitIds[j]]) {
                             mapNewUnits[unitType][serviceUnitIds[j]] = true;
    @@ -805,7 +805,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
             // Get the number of services
             uint256 numServices = serviceIds.length;
             // Loop over service Ids, accumulate donation value and check for the service existence
    -        for (uint256 i = 0; i < numServices; ++i) {
    +        for (uint256 i = numServices - 1; i >= 0; -i) {
                 // Check for the service Id existence
                 if (!IServiceRegistry(serviceRegistry).exists(serviceIds[i])) {
                     revert ServiceDoesNotExist(serviceIds[i]);
    @@ -975,7 +975,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
                 emit IncentiveFractionsUpdated(eCounter + 1);
             } else {
                 // Copy current tokenomics point into the next one such that it has necessary tokenomics parameters
    -            for (uint256 i = 0; i < 2; ++i) {
    +            for (uint256 i = 1; i >= 0; --i) {
                     nextEpochPoint.unitPoints[i].topUpUnitFraction = tp.unitPoints[i].topUpUnitFraction;
                     nextEpochPoint.unitPoints[i].rewardUnitFraction = tp.unitPoints[i].rewardUnitFraction;
                 }
    @@ -1101,13 +1101,13 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {

             // Component / agent total supply
             uint256[] memory registriesSupply = new uint256[](2);
    -        for (uint256 i = 0; i < 2; ++i) {
    +        for (uint256 i = 1; i >= 0; --i) {
                 registriesSupply[i] = IToken(registries[i]).totalSupply();
             }

             // Check the input data
             uint256[] memory lastIds = new uint256[](2);
    -        for (uint256 i = 0; i < unitIds.length; ++i) {
    +        for (uint256 i = unitIds.length - 1; i >= 0; --i) {
                 // Check for the unit type to be component / agent only
                 if (unitTypes[i] > 1) {
                     revert Overflow(unitTypes[i], 1);
    @@ -1129,7 +1129,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
             // Get the current epoch counter
             uint256 curEpoch = epochCounter;

    -        for (uint256 i = 0; i < unitIds.length; ++i) {
    +        for (uint256 i = unitIds.length - 1; i >= 0; --i) {
                 // Get the last epoch number the incentives were accumulated for
                 uint256 lastEpoch = mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch;
                 // Finalize unit rewards and top-ups if there were pending ones from the previous epoch
    @@ -1171,13 +1171,13 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {

             // Component / agent total supply
             uint256[] memory registriesSupply = new uint256[](2);
    -        for (uint256 i = 0; i < 2; ++i) {
    +        for (uint256 i = 1; i >= 0; --i) {
                 registriesSupply[i] = IToken(registries[i]).totalSupply();
             }

             // Check the input data
             uint256[] memory lastIds = new uint256[](2);
    -        for (uint256 i = 0; i < unitIds.length; ++i) {
    +        for (uint256 i = unitIds.length - 1; i >= 0; --i) {
                 // Check for the unit type to be component / agent only
                 if (unitTypes[i] > 1) {
                     revert Overflow(unitTypes[i], 1);
    @@ -1199,7 +1199,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
             // Get the current epoch counter
             uint256 curEpoch = epochCounter;

    -        for (uint256 i = 0; i < unitIds.length; ++i) {
    +        for (uint256 i = unitIds.length - 1; i >= 0; --i) {
                 // Get the last epoch number the incentives were accumulated for
                 uint256 lastEpoch = mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch;
                 // Calculate rewards and top-ups if there were pending ones from the previous epoch
     ESCOD
    -        for (uint256 i = 0; i < unitIds.length; ++i) {
    +        for (uint256 i = unitIds.length - 1; i >= 0; --i) {
                 // Get the last epoch number the incentives were accumulated for
                 uint256 lastEpoch = mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch;
                 // Finalize unit rewards and top-ups if there were pending ones from the previous epoch
    @@ -1171,13 +1171,13 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {

             // Component / agent total supply
             uint256[] memory registriesSupply = new uint256[](2);
    -        for (uint256 i = 0; i < 2; ++i) {
    +        for (uint256 i = 1; i >= 0; --i) {
                 registriesSupply[i] = IToken(registries[i]).totalSupply();
             }

             // Check the input data
             uint256[] memory lastIds = new uint256[](2);
    -        for (uint256 i = 0; i < unitIds.length; ++i) {
    +        for (uint256 i = unitIds.length - 1; i >= 0; --i) {
                 // Check for the unit type to be component / agent only
                 if (unitTypes[i] > 1) {
                     revert Overflow(unitTypes[i], 1);
    @@ -1199,7 +1199,7 @@ contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
             // Get the current epoch counter
             uint256 curEpoch = epochCounter;

    -        for (uint256 i = 0; i < unitIds.length; ++i) {
    +        for (uint256 i = unitIds.length - 1; i >= 0; --i) {
                 // Get the last epoch number the incentives were accumulated for
                 uint256 lastEpoch = mapUnitIncentives[unitTypes[i]][unitIds[i]].lastEpoch;
                 // Calculate rewards and top-ups if there were pending ones from the previous epoch

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol

```
File: tokenomics/contracts/TokenomicsConstants.sol	

55: for (uint256 i = 0; i < numYears; ++i) {

92: for (uint256 i = 1; i < numYears; ++i) {
```
    diff --git a/tokenomics/contracts/TokenomicsConstants.sol b/tokenomics/contracts/TokenomicsConstants.sol
    index b359ea9..ddd2e54 100644
    --- a/tokenomics/contracts/TokenomicsConstants.sol
    +++ b/tokenomics/contracts/TokenomicsConstants.sol
    @@ -52,7 +52,7 @@ abstract contract TokenomicsConstants {
                 uint256 maxMintCapFraction = 2;

                 // Get the supply cap until the current year
    -            for (uint256 i = 0; i < numYears; ++i) {
    +            for (uint256 i = numYears - 1; i >= 0 ; -i) {
                     supplyCap += (supplyCap * maxMintCapFraction) / 100;
                 }
                 // Return the difference between last two caps (inflation for the current year)
    @@ -89,7 +89,7 @@ abstract contract TokenomicsConstants {
                 uint256 maxMintCapFraction = 2;

                 // Get the supply cap until the year before the current year
    -            for (uint256 i = 1; i < numYears; ++i) {
    +            for (uint256 i = numYears - 1; i >= 1; --i) {
                     supplyCap += (supplyCap * maxMintCapFraction) / 100;
                 }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol

```
File: tokenomics/contracts/Treasury.sol	

276: for (uint256 i = 0; i < numServices; ++i) {
```
    diff --git a/tokenomics/contracts/Treasury.sol b/tokenomics/contracts/Treasury.sol
    index 516c631..5ada912 100644
    --- a/tokenomics/contracts/Treasury.sol
    +++ b/tokenomics/contracts/Treasury.sol
    @@ -273,7 +273,7 @@ contract Treasury is IErrorsTokenomics {
             }

             uint256 totalAmount;
    -        for (uint256 i = 0; i < numServices; ++i) {
    +        for (uint256 i = numServices - 1; i >= 0; --i) {
                 if (amounts[i] == 0) {
                     revert ZeroValue();
                 }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |