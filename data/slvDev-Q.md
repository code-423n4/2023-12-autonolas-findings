## Low Findings

|  | Issue | Instances |
| --- | --- | --- |
| [L-01] | Low Level Eth Transfer to Custom Addresses | 2 |
| [L-02] | Excessive Loop Iterations with complex logic mey lead to DOS | 3 |
| [L-03] | Unsafe use of `transfer()/transferFrom()` with IERC20 | 8 |
| [L-04] | Array is `push()`ed but not `pop()`ed | 2 |
| [L-05] | Unchecked Return Values of `transfer()/transferFrom()` | 3 |
| [L-06] | Non-compliance with EIP-721 in `tokenURI()` Implementation | 1 |
| [L-07] | Mint to zero address | 2 |
| [L-08] | Possible Revert ERC20 Transfers with Zero Value | 1 |
| [L-09] | Consider Adding a Denylist modifier to Your NFT Contracts | 1 |
| [L-10] | Missing `address(0)` Check in Constructor/Initializer | 18 |
| [L-11] | Vulnerable versions of packages are being used | 1 |
| [L-12] | Unauthorized `receive()/fallback()` Functions | 2 |

## NonCritical Findings

|  | Issue | Instances |
| --- | --- | --- |
| [NC-01] | Avoid using constant decimals in expressions | 10 |
| [NC-02] | Equal `block.timestamp` Cases Not Handled | 9 |
| [NC-03] | Contract uses both `require()`/`revert()` as well as custom errors | 21 |
| [NC-04] | Solidity Version Too Recent to be Trusted | 4 |
| [NC-05] | Inconsistent spacing in comments | 33 |
| [NC-06] | Function/Constructor Argument Names Not in mixedCase | 28 |
| [NC-07] | State-Altering Functions Should Emit Events | 3 |
| [NC-08] | Upgrade `openzeppelin` to the Latest Version - 5.0.0 | 11 |
| [NC-09] | Duplicated `require()`/`revert()` checks should be refactored to a modifier or function | 72 |
| [NC-10] | Consider Avoid Loops in Public Functions | 29 |
| [NC-11] | Consider moving `msg.sender` checks to a common authorization `modifier` | 57 |
| [NC-12] | Unused Function Parameters | 24 |
| [NC-13] | Consider using `delete` rather than assigning `false` to clear values | 3 |
| [NC-14] | Style guide: Non-`external`/`public` variable names should begin with an underscore | 3 |
| [NC-15] | Double Type Casting | 16 |
| [NC-16] | Use Unchecked for Divisions on Constant or Immutable Values | 8 |
| [NC-17] | Missing Event Emission After Critical Initialize() Function | 1 |
| [NC-18] | Variables need not be initialized to zero | 48 |
| [NC-19] | Consider using descriptive `constant`s when passing zero as a function argument | 6 |

### Low Findings Details

### [L-01] Low Level Eth Transfer to Custom Addresses

Contracts should avoid making low-level calls to custom addresses, especially if these calls are based on address parameters in the function. 
Such behavior can lead to unexpected execution of untrusted code. Instead, consider using Solidity's high-level function calls or contract interactions.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: tokenomics/contracts/Treasury.sol

339: (success, ) = to.call{value: tokenAmount}("");
406: (success, ) = account.call{value: accountRewards}("");
```

| [Line #339](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L339) | [Line #406](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L406) | </details>

### [L-02] Excessive Loop Iterations with complex logic mey lead to DOS

Using loops with a fixed large number of iterations can pose a risk.
If the iterations approach or exceed the block gas limit, they might lead to Denial of Service (DOS) attacks, rendering the entire contract inoperable.
Ensure that the number of loop iterations is reasonably capped or designed to prevent possible DOS scenarios.

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

232: for (uint256 i = 0; i < 255; ++i) {
561: for (uint256 i = 0; i < 128; ++i) {
693: for (uint256 i = 0; i < 255; ++i) {
```

| [Line #232](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L232) | [Line #561](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L561) | [Line #693](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L693) | </details>

### [L-03] Unsafe use of `transfer()/transferFrom()` with IERC20

Not all tokens adhere strictly to the ERC20 standard, even if they are largely accepted as compliant.
A prominent example is Tether (USDT) on L1, where the transfer() and transferFrom() functions do not return booleans, contrary to the standard's requirements.
Instead, these functions provide no return value at all.
This discrepancy can lead to issues when such tokens are cast to IERC20: their non-standard function signatures can cause calls to revert.
As a safer alternative, consider using OpenZeppelin's SafeERC20 library, which offers safeTransfer() and safeTransferFrom() functions to address these incompatibilities.

<details>
<summary><i>8 issue instances in 5 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

364: IERC20(token).transferFrom(msg.sender, address(this), amount);
534: IERC20(token).transfer(msg.sender, amount);
```

| [Line #364](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364) | [Line #534](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L534) | 
```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol

79: bool success = IERC20(childToken).transfer(to, amount);
102: bool success = IERC20(childToken).transferFrom(msg.sender, address(this), amount);
```

| [Line #79](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79) | [Line #102](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L102) | 
```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol

98: bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
```

| [Line #98](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L98) | 
```solidity
File: tokenomics/contracts/Depository.sol

390: IToken(olas).transfer(msg.sender, payout);
```

| [Line #390](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L390) | 
```solidity
File: tokenomics/contracts/Treasury.sol

235: bool success = IToken(token).transferFrom(account, address(this), tokenAmount);
362: success = IToken(token).transfer(to, tokenAmount);
```

| [Line #235](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L235) | [Line #362](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L362) | </details>


### [L-04] Array is `push()`ed but not `pop()`ed

Arrays in the contract have entries added using the `push` method, but no corresponding `pop` method is found to remove them. 
Ensure to consider whether old entries need to be removed or if there should be a maximum array length.
Cases with specific potential issues might be reported separately under different issue categories.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

315: mapUserPoints[account].push(uNew);
```

| [Line #315](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L315) | 
```solidity
File: registries/contracts/UnitRegistry.sol

143: mapUnitIdHashes[unitId].push(unitHash);
```

| [Line #143](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L143) | </details>


### [L-05] Unchecked Return Values of `transfer()/transferFrom()`

`transfer()` or `transferFrom()` methods without checking their return values.
Some IERC20 implementations use these boolean return values to signal transaction failures instead of using revert().
Not checking these values could allow transactions to proceed without the intended token transfers.
It is recommended to always check the return values of these methods.

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

364: IERC20(token).transferFrom(msg.sender, address(this), amount);
534: IERC20(token).transfer(msg.sender, amount);
```

| [Line #364](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L364) | [Line #534](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L534) | 
```solidity
File: tokenomics/contracts/Depository.sol

390: IToken(olas).transfer(msg.sender, payout);
```

| [Line #390](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L390) | 
```solidity
File: tokenomics/contracts/Treasury.sol

362: success = IToken(token).transfer(to, tokenAmount);
```

| [Line #362](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L362) | </details>


### [L-06] Non-compliance with EIP-721 in `tokenURI()` Implementation

According to EIP-721, the `tokenURI()` function should revert if the token ID passed is not a valid NFT.
However, some contracts do not adhere to this standard.

If the NFT has not been minted, `tokenURI()` should revert as per the EIP-721 standard, ensuring better compliance and robustness.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: registries/contracts/GenericRegistry.sol

135: function tokenURI(uint256 unitId) public view virtual override returns (string memory) {
        bytes32 unitHash = _getUnitHash(unitId);
        // Parse 2 parts of bytes32 into left and right hex16 representation, and concatenate into string
        // adding the base URI and a cid prefix for the full base16 multibase prefix IPFS hash representation
        return string(abi.encodePacked(baseURI, CID_PREFIX, _toHex16(bytes16(unitHash)),
            _toHex16(bytes16(unitHash << 128))));
    }
```

| [Line #135](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L135) | </details>

### [L-07] Mint to zero address

`solmate/src/tokens/ERC20.sol` allow to mint tokens to zero address. 

Minting tokens to the zero address should be avoided. In several locations in the code, precautions are not being taken to prevent minting to the zero address. This may lead to unintended consequences.
Consider applying checks in the minting functions to ensure tokens are not minted to the zero address.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: governance/contracts/OLAS.sol

75: function mint(address account, uint256 amount) external {
```

| [Line #75](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L75) | 
```solidity
File: governance/contracts/bridges/BridgedERC20.sol

48: function mint(address account, uint256 amount) external {
```

| [Line #48](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L48) | </details>

### [L-08] Possible Revert ERC20 Transfers with Zero Value

Some ERC20 tokens (for example, LEND) revert on attempts to transfer a zero value.
This can become a problem in any contract that doesn't handle these cases correctly.

If a token transfer fails, any additional logic in a function might also be affected or even fail entirely.
Therefore, contracts interacting with ERC20 tokens should account for the possibility that a zero-value transfer might revert.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol

/// @audit `amount` has not been checked for zero value before transfer
79: bool success = IERC20(childToken).transfer(to, amount);
```

| [Line #79](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L79) | </details>

### [L-09] Consider Adding a Denylist modifier to Your NFT Contracts

Given the rise in NFT thefts, it's important to implement safeguards to prevent hacked or stolen NFTs from being converted into liquidity on your platform. One such safeguard is a denylist modifier.

A denylist modifier would prevent reported stolen NFTs from being listed or transacted. Many marketplaces such as Opensea and NFT projects like Manifold already implement this feature in their smart contracts. 

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: registries/contracts/GenericRegistry.sol

9: abstract contract GenericRegistry is IErrorsRegistries, ERC721 {
```

| [Line #9](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L9) | </details>

### [L-10] Missing `address(0)` Check in Constructor/Initializer

The constructor/initializer does not include a check for `address(0)` when initializing state variables that hold addresses.
Initializing a state variable with `address(0)` can lead to unintended behavior and vulnerabilities in the contract, 
such as sending funds to an inaccessible address. 
It is recommended to include a validation step to ensure that address parameters are not set to `address(0)`.

<details>
<summary><i>18 issue instances in 7 files:</i></summary>

```solidity
File: governance/contracts/GovernorOLAS.sol

/// @audit `governanceToken` passed to inherited constructor is not checked for `address(0)`
/// @audit `timelock` passed to inherited constructor is not checked for `address(0)`
16: constructor(
        IVotes governanceToken,
        TimelockController timelock,
        uint256 initialVotingDelay,
        uint256 initialVotingPeriod,
        uint256 initialProposalThreshold,
        uint256 quorumFraction
    )
        Governor("Governor OLAS")
        GovernorSettings(initialVotingDelay, initialVotingPeriod, initialProposalThreshold)
        GovernorVotes(governanceToken)
        GovernorVotesQuorumFraction(quorumFraction)
        GovernorTimelockControl(timelock)
    {}
```

| [Line #16](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L16) | 
```solidity
File: governance/contracts/Timelock.sol

/// @audit `proposers` passed to inherited constructor is not checked for `address(0)`
/// @audit `executors` passed to inherited constructor is not checked for `address(0)`
10: constructor(uint256 minDelay, address[] memory proposers, address[] memory executors)
        TimelockController(minDelay, proposers, executors, msg.sender)
    {}
```

| [Line #10](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/Timelock.sol#L10) | 
```solidity
File: governance/contracts/veOLAS.sol

/// @audit `_token` has lack of `address(0)` check before use
132: constructor(address _token, string memory _name, string memory _symbol)
    {
```

| [Line #132](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L132) | 
```solidity
File: registries/contracts/UnitRegistry.sol

/// @audit `_unitType` has lack of `address(0)` check before use
35: constructor(UnitType _unitType) {
```

| [Line #35](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L35) | 
```solidity
File: registries/contracts/AgentRegistry.sol

/// @audit `_componentRegistry` has lack of `address(0)` check before use
20: constructor(string memory _name, string memory _symbol, string memory _baseURI, address _componentRegistry)
        UnitRegistry(UnitType.Agent)
        ERC721(_name, _symbol)
    {
```

| [Line #20](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L20) | 
```solidity
File: registries/contracts/RegistriesManager.sol

/// @audit `_componentRegistry` has lack of `address(0)` check before use
/// @audit `_agentRegistry` has lack of `address(0)` check before use
15: constructor(address _componentRegistry, address _agentRegistry) {
```

| [Line #15](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L15) | 
```solidity
File: registries/contracts/multisigs/GnosisSafeMultisig.sol

/// @audit `_gnosisSafe` has lack of `address(0)` check before use
/// @audit `_gnosisSafeProxyFactory` has lack of `address(0)` check before use
37: constructor (address payable _gnosisSafe, address _gnosisSafeProxyFactory) {
```

| [Line #37](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol#L37) | </details>

### [L-11] Vulnerable versions of packages are being used

The package.json configuration file says that the project is using OZ which has a not last update version.
Latest non vulnerable version `4.9.3` for @openzeppelin/contracts and @openzeppelin/contracts-upgradeable.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: app/2023-12-autonolas/governance/package.json

28: "@openzeppelin/contracts": "=4.8.3",
```

| [Line #28](https://github.com/code-423n4/2023-12-autonolas/blob/main/app/2023-12-autonolas/governance/package.json#L28) | </details>

### [L-12] Unauthorized `receive()/fallback()` Functions

Empty receive() or payable fallback() functions without proper authorization can result in a loss of funds. If the intention is for the Ether to be used, the function should call another function, otherwise, it should revert (e.g. require(msg.sender == address(weth))).

Having no access control on the function means that someone may send Ether to the contract and have no way to get anything back out. Even if the concern is spending a small amount of gas to check the sender, the code should at least have a function to rescue unused Ether, ensuring funds are not trapped within the contract.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

73: receive() external payable
```
[73](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L73)

```solidity
File: governance/contracts/bridges/HomeMediator.sol

73: receive() external payable
```
[73](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L73)
</details>


### NonCritical Findings Details

### [NC-01] Avoid using constant decimals in expressions

The use of fixed decimal values such as `1e18/10 ** 18` in Solidity contracts can lead to bugs, and vulnerabilities, when interacting with tokens having different decimal configurations.
Not all ERC20 tokens follow the standard 18 decimal places, and assumptions about decimal places can lead to miscalculations.
        
Always retrieve and use the `decimals()` function from the token contract itself when performing calculations involving token amounts.
This ensures that your contract correctly handles tokens with any number of decimal places, mitigating the risk of numerical errors or under/overflows.

<details>
<summary><i>10 issue instances in 3 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

225: block_slope = (1e18 * uint256(block.number - lastPoint.blockNumber)) / uint256(block.timestamp - lastPoint.ts);
258: lastPoint.blockNumber = initialPoint.blockNumber + uint64((block_slope * uint256(tStep - initialPoint.ts)) / 1e18);
```
[225](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L225) | [258](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L258)

```solidity
File: tokenomics/contracts/GenericBondCalculator.sol

89: priceLP = (reserve1 * 1e18) / totalSupply;
```
[89](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L89)

```solidity
File: tokenomics/contracts/Tokenomics.sol

341: devsPerCapital = 1e18;
342: tp.epochPoint.idf = 1e18;
357: codePerDev = 1e18;
861: idf = 1e18 + fKD;
1048: nextEpochPoint.epochPoint.idf = 1e18;
1256: idf = 1e18;
1266: idf = 1e18;
```
[341](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L341) | [342](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L342) | [357](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L357) | [861](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L861) | [1048](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1048) | [1256](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1256) | [1266](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1266)
</details>

### [NC-02] Equal `block.timestamp` Cases Not Handled

Solidity code that checks if `block.timestamp` is greater than a certain variable but fails to handle the case when `block.timestamp` is equal to the variable may lead to unintended behaviors and potential vulnerabilities.

To ensure correct and secure function execution, it's recommended to include checks for cases where `block.timestamp` equals the compared variable.

<details>
<summary><i>9 issue instances in 1 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

188: if (oldLocked.endTime > block.timestamp && oldLocked.amount > 0) {
192: if (newLocked.endTime > block.timestamp && newLocked.amount > 0) {
222: if (block.timestamp > lastPoint.ts) {
240: if (tStep > block.timestamp) {
297: if (oldLocked.endTime > block.timestamp) {
306: if (newLocked.endTime > block.timestamp && newLocked.endTime > oldLocked.endTime) {
445: if (unlockTime > block.timestamp + MAXTIME) {
502: if (unlockTime > block.timestamp + MAXTIME) {
512: if (lockedBalance.endTime > block.timestamp) {
```

| [Line #188](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L188) | [Line #192](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L192) | [Line #222](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L222) | [Line #240](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L240) | [Line #297](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L297) | [Line #306](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L306) | [Line #445](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L445) | [Line #502](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L502) | [Line #512](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L512) | </details>


### [NC-03] Contract uses both `require()`/`revert()` as well as custom errors

The contract uses both `require()/revert()` and `custom errors` for error handling.

It's recommended to use a consistent approach for error handling in a single file.

<details>
<summary><i>21 issue instances in 21 files:</i></summary>

```solidity
File: governance/contracts/OLAS.sol

17: contract OLAS is ERC20 {
```

| [Line #17](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L17) | 
```solidity
File: governance/contracts/veOLAS.sol

86: contract veOLAS is IErrors, IVotes, IERC20, IERC165 {
```

| [Line #86](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L86) | 
```solidity
File: governance/contracts/wveOLAS.sol

130: contract wveOLAS {
```

| [Line #130](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L130) | 
```solidity
File: governance/contracts/multisigs/GuardCM.sol

88: contract GuardCM {
```

| [Line #88](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L88) | 
```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

46: contract FxGovernorTunnel is IFxMessageProcessor {
```

| [Line #46](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L46) | 
```solidity
File: governance/contracts/bridges/HomeMediator.sol

46: contract HomeMediator {
```

| [Line #46](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L46) | 
```solidity
File: governance/contracts/bridges/BridgedERC20.sol

18: contract BridgedERC20 is ERC20 {
```

| [Line #18](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L18) | 
```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol

24: contract FxERC20ChildTunnel is FxBaseChildTunnel {
```

| [Line #24](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L24) | 
```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol

24: contract FxERC20RootTunnel is FxBaseRootTunnel {
```

| [Line #24](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L24) | 
```solidity
File: registries/contracts/ComponentRegistry.sol

8: contract ComponentRegistry is UnitRegistry {
```

| [Line #8](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L8) | 
```solidity
File: registries/contracts/AgentRegistry.sol

9: contract AgentRegistry is UnitRegistry {
```

| [Line #9](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L9) | 
```solidity
File: registries/contracts/RegistriesManager.sol

9: contract RegistriesManager is GenericManager {
```

| [Line #9](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L9) | 
```solidity
File: registries/contracts/multisigs/GnosisSafeMultisig.sol

24: contract GnosisSafeMultisig {
```

| [Line #24](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol#L24) | 
```solidity
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

50: contract GnosisSafeSameAddressMultisig {
```

| [Line #50](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L50) | 
```solidity
File: tokenomics/contracts/Depository.sol

62: contract Depository is IErrorsTokenomics {
```

| [Line #62](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L62) | 
```solidity
File: tokenomics/contracts/Dispenser.sol

11: contract Dispenser is IErrorsTokenomics {
```

| [Line #11](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L11) | 
```solidity
File: tokenomics/contracts/DonatorBlacklist.sol

20: contract DonatorBlacklist {
```

| [Line #20](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L20) | 
```solidity
File: tokenomics/contracts/GenericBondCalculator.sol

20: contract GenericBondCalculator {
```

| [Line #20](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L20) | 
```solidity
File: tokenomics/contracts/Tokenomics.sol

118: contract Tokenomics is TokenomicsConstants, IErrorsTokenomics {
```

| [Line #118](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L118) | 
```solidity
File: tokenomics/contracts/TokenomicsProxy.sol

26: contract TokenomicsProxy {
```

| [Line #26](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsProxy.sol#L26) | 
```solidity
File: tokenomics/contracts/Treasury.sol

39: contract Treasury is IErrorsTokenomics {
```

| [Line #39](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L39) | </details>



### [NC-04] Solidity Version Too Recent to be Trusted

The current Solidity version used in the code is too recent to be trusted.
Using a newer version might introduce unrecovered bugs. It is recommended to use version 0.8.21.

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: governance/contracts/multisigs/GuardCM.sol

2: pragma solidity ^0.8.23;
```

| [Line #2](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L2) | 
```solidity
File: governance/contracts/bridges/BridgedERC20.sol

2: pragma solidity ^0.8.23;
```

| [Line #2](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L2) | 
```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol

2: pragma solidity ^0.8.23;
```

| [Line #2](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L2) | 
```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol

2: pragma solidity ^0.8.23;
```

| [Line #2](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L2) | </details>



### [NC-05] Inconsistent spacing in comments
Some lines use // x and some use //x. The instances below point out the usages that don't follow the majority, within each file:
<details>
<summary><i>33 issue instances in 8 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

373: ///      cannot extend their locktime and deposit for a brand new user.
639: ///         as the behavior and the return value is undefined.
```

| [Line #373](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L373) | [Line #639](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L639) | 
```solidity
File: governance/contracts/multisigs/GuardCM.sol

206: ///         address target, uint96 value, uint32 payloadLength, bytes payload.
436: ///         and corresponding supported L2-s, if the contract interacts with them.
491: ///         corresponding L2 bridge mediator contracts, and supported chain Ids.
537: ///         If the proposal is defeated (not enough votes or never voted on),
538: ///         the governance is considered inactive for about a week.
```

| [Line #206](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L206) | [Line #436](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L436) | [Line #491](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L491) | [Line #537](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L537) | [Line #538](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L538) | 
```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

79: ///         This triggers a self-contract transaction of FxGovernorTunnel that changes the Root Governor address.
102: ///        transactions packed into a single buffer, where each transaction is composed as follows:
103: ///        - target address of 20 bytes (160 bits);
104: ///        - value of 12 bytes (96 bits), as a limit for all of Autonolas ecosystem contracts;
105: ///        - payload length of 4 bytes (32 bits), as 2^32 - 1 characters is more than enough to fill a whole block;
106: ///        - payload as bytes, with the length equal to the specified payload length.
```

| [Line #79](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L79) | [Line #102](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L102) | [Line #103](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L103) | [Line #104](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L104) | [Line #105](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L105) | [Line #106](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L106) | 
```solidity
File: governance/contracts/bridges/HomeMediator.sol

79: ///         This triggers a self-contract transaction of HomeMediator that changes the Foreign Governor address.
100: ///        continuous transactions packed into a single buffer, where each transaction is composed as follows:
101: ///        - target address of 20 bytes (160 bits);
102: ///        - value of 12 bytes (96 bits), as a limit for all of Autonolas ecosystem contracts;
103: ///        - payload length of 4 bytes (32 bits), as 2^32 - 1 characters is more than enough to fill a whole block;
104: ///        - payload as bytes, with the length equal to the specified payload length.
```

| [Line #79](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L79) | [Line #100](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L100) | [Line #101](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L101) | [Line #102](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L102) | [Line #103](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L103) | [Line #104](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L104) | 
```solidity
File: governance/contracts/bridges/BridgedERC20.sol

17: ///      another chain must be minted and burned solely by the bridge mediator contract.
```

| [Line #17](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L17) | 
```solidity
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

71: ///         the set of owners' addresses and the threshold. There are two scenarios possible:
72: ///         1. The multisig proxy is already updated before reaching this function. Then the multisig address
73: ///            must be passed as a payload such that its owners and threshold are verified against those specified
74: ///            in the argument list.
75: ///         2. The multisig proxy is not yet updated. Then the multisig address must be passed in a packed bytes of
76: ///            data along with the Gnosis Safe `execTransaction()` function arguments packed payload. That payload
77: ///            is going to modify the mulsisig proxy as per its signed transaction. At the end, the updated multisig
78: ///            proxy is going to be verified with the provided set of owners' addresses and the threshold.
79: ///         Note that owners' addresses in the multisig are stored in reverse order compared to how they were added:
80: ///         https://etherscan.io/address/0xd9db270c1b5e3bd161e8c8503c55ceabee709552#code#F6#L56
```

| [Line #71](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L71) | [Line #72](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L72) | [Line #73](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L73) | [Line #74](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L74) | [Line #75](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L75) | [Line #76](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L76) | [Line #77](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L77) | [Line #78](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L78) | [Line #79](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L79) | [Line #80](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L80) | 
```solidity
File: tokenomics/contracts/Tokenomics.sol

866: ///         not valid not to call a checkpoint for longer than a year. Thus, the function will return false otherwise.
```

| [Line #866](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L866) | 
```solidity
File: tokenomics/contracts/Treasury.sol

250: ///         configuration component / agent owners until the service is re-deployed when new agent Ids are accounted for.
464: ///if_succeeds {:msg "slashed funds check"} IServiceRegistry(ITokenomics(tokenomics).serviceRegistry()).slashedFunds() >= minAcceptedETH
```

| [Line #250](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L250) | [Line #464](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L464) | </details>


### [NC-06] Function/Constructor Argument Names Not in mixedCase

Underscore before of after function argument names is a common convention in Solidity NOT a documentation requirement.

Function arguments should use mixedCase for better readability and consistency with Solidity style guidelines. 
Examples of good practice include: initialSupply, account, recipientAddress, senderAddress, newOwner. 
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-argument-names)

Rule exceptions
- Allow constant variable name/symbol/decimals to be lowercase (ERC20).
- Allow `_` at the beginning of the mixedCase match for `private variables` and `unused parameters`.

<details>
<summary><i>28 issue instances in 18 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

132: constructor(address _token, string memory _name, string memory _symbol)
    {
```

| [Line #132](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L132) | 
```solidity
File: governance/contracts/wveOLAS.sol

145: constructor(address _ve, address _token) {
```

| [Line #145](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L145) | 
```solidity
File: governance/contracts/multisigs/GuardCM.sol

138: constructor(
        address _timelock,
        address _multisig,
        address _governor
    ) {
```

| [Line #138](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L138) | 
```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

62: constructor(address _fxChild, address _rootGovernor) {
```

| [Line #62](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L62) | 
```solidity
File: governance/contracts/bridges/HomeMediator.sol

62: constructor(address _AMBContractProxyHome, address _foreignGovernor) {
```

| [Line #62](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L62) | 
```solidity
File: governance/contracts/bridges/FxERC20ChildTunnel.sol

37: constructor(address _fxChild, address _childToken, address _rootToken) FxBaseChildTunnel(_fxChild) {
```

| [Line #37](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol#L37) | 
```solidity
File: governance/contracts/bridges/FxERC20RootTunnel.sol

38: constructor(address _checkpointManager, address _fxRoot, address _childToken, address _rootToken)
        FxBaseRootTunnel(_checkpointManager, _fxRoot)
    {
```

| [Line #38](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol#L38) | 
```solidity
File: registries/contracts/UnitRegistry.sol

34: constructor(UnitType _unitType) {
```

| [Line #34](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L34) | 
```solidity
File: registries/contracts/ComponentRegistry.sol

16: constructor(string memory _name, string memory _symbol, string memory _baseURI)
        UnitRegistry(UnitType.Component)
        ERC721(_name, _symbol)
    {
```

| [Line #16](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L16) | 
```solidity
File: registries/contracts/AgentRegistry.sol

20: constructor(string memory _name, string memory _symbol, string memory _baseURI, address _componentRegistry)
        UnitRegistry(UnitType.Agent)
        ERC721(_name, _symbol)
    {
```

| [Line #20](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L20) | 
```solidity
File: registries/contracts/RegistriesManager.sol

14: constructor(address _componentRegistry, address _agentRegistry) {
```

| [Line #14](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol#L14) | 
```solidity
File: registries/contracts/multisigs/GnosisSafeMultisig.sol

37: constructor (address payable _gnosisSafe, address _gnosisSafeProxyFactory) {
```

| [Line #37](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol#L37) | 
```solidity
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

60: constructor(bytes32 _proxyHash) {
```

| [Line #60](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L60) | 
```solidity
File: tokenomics/contracts/Depository.sol

143: function changeManagers(address _tokenomics, address _treasury) external {
163: function changeBondCalculator(address _bondCalculator) external {
106: constructor(address _olas, address _tokenomics, address _treasury, address _bondCalculator)
    {
```

| [Line #143](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L143) | [Line #163](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L163) | [Line #106](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L106) | 
```solidity
File: tokenomics/contracts/Dispenser.sol

64: function changeManagers(address _tokenomics, address _treasury) external {
30: constructor(address _tokenomics, address _treasury)
    {
```

| [Line #64](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L64) | [Line #30](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L30) | 
```solidity
File: tokenomics/contracts/GenericBondCalculator.sol

29: constructor(address _olas, address _tokenomics) {
```

| [Line #29](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L29) | 
```solidity
File: tokenomics/contracts/Tokenomics.sol

264: function initializeTokenomics(
        address _olas,
        address _treasury,
        address _depository,
        address _dispenser,
        address _ve,
        uint256 _epochLen,
        address _componentRegistry,
        address _agentRegistry,
        address _serviceRegistry,
        address _donatorBlacklist
    ) external
    {
423: function changeManagers(address _treasury, address _depository, address _dispenser) external {
450: function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {
474: function changeDonatorBlacklist(address _donatorBlacklist) external {
497: function changeTokenomicsParameters(
        uint256 _devsPerCapital,
        uint256 _codePerDev,
        uint256 _epsilonRate,
        uint256 _epochLen,
        uint256 _veOLASThreshold
    ) external
    {
562: function changeIncentiveFractions(
        uint256 _rewardComponentFraction,
        uint256 _rewardAgentFraction,
        uint256 _maxBondFraction,
        uint256 _topUpComponentFraction,
        uint256 _topUpAgentFraction
    ) external
    {
```

| [Line #264](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L264) | [Line #423](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L423) | [Line #450](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L450) | [Line #474](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L474) | [Line #497](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L497) | [Line #562](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L562) | 
```solidity
File: tokenomics/contracts/Treasury.sol

156: function changeManagers(address _tokenomics, address _depository, address _dispenser) external {
182: function changeMinAcceptedETH(uint256 _minAcceptedETH) external {
95: constructor(address _olas, address _tokenomics, address _depository, address _dispenser) payable {
```

| [Line #156](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L156) | [Line #182](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L182) | [Line #95](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L95) | </details>


### [NC-07] State-Altering Functions Should Emit Events

Functions that alter state should emit events to inform users of the state change.
This is crucial for functions that modify the state and don't return a value.
The absence of events in such scenarios could lead to lack of transparency and traceability, undermining the contract's reliability.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

275: totalNumPoints = curNumPoint;
```

| [Line #275](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L275) | 
```solidity
File: tokenomics/contracts/Tokenomics.sol

290: owner = msg.sender;
824: lastDonationBlockNumber = uint32(block.number);
```

| [Line #290](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L290) | [Line #824](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L824) | </details>


### [NC-08] Upgrade `openzeppelin` to the Latest Version - 5.0.0

These contracts import contracts from @openzeppelin/contracts but they are not using the latest version.
For more information, please visit: [OpenZeppelin GitHub Releases](https://github.com/OpenZeppelin/openzeppelin-contracts/releases)
It is recommended to always use the latest version to take advantage of updates and security fixes.

<details>
<summary><i>11 issue instances in 3 files:</i></summary>

```solidity
File: governance/contracts/GovernorOLAS.sol

4: import {IERC165} from "@openzeppelin/contracts/utils/introspection/IERC165.sol";
5: import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";
6: import {GovernorSettings} from "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";
7: import {GovernorCompatibilityBravo} from "@openzeppelin/contracts/governance/compatibility/GovernorCompatibilityBravo.sol";
8: import {GovernorVotes, IVotes} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";
9: import {GovernorVotesQuorumFraction} from "@openzeppelin/contracts/governance/extensions/GovernorVotesQuorumFraction.sol";
10: import {GovernorTimelockControl, TimelockController} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";
```

| [Line #4](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L4) | [Line #5](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L5) | [Line #6](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L6) | [Line #7](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L7) | [Line #8](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L8) | [Line #9](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L9) | [Line #10](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/GovernorOLAS.sol#L10) | 
```solidity
File: governance/contracts/Timelock.sol

4: import "@openzeppelin/contracts/governance/TimelockController.sol";
```

| [Line #4](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/Timelock.sol#L4) | 
```solidity
File: governance/contracts/veOLAS.sol

4: import "@openzeppelin/contracts/governance/utils/IVotes.sol";
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
6: import "@openzeppelin/contracts/utils/introspection/IERC165.sol";
```

| [Line #4](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L4) | [Line #5](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L5) | [Line #6](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L6) | </details>


### [NC-09] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function

Duplicated require() or revert() checks should be refactored to a modifier or function.
This helps in maintaining a clean and organized codebase and saves deployment costs.

<details>
<summary><i>72 issue instances in 12 files:</i></summary>

```solidity
File: governance/contracts/OLAS.sol

44: if (msg.sender != owner) {
            revert ManagerOnly(msg.sender, owner);
59: if (msg.sender != owner) {
            revert ManagerOnly(msg.sender, owner);
```

| [Line #44](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44) | [Line #59](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59) | 
```solidity
File: governance/contracts/veOLAS.sol

379: if (amount == 0) {
            revert ZeroValue();
427: if (amount == 0) {
            revert ZeroValue();
461: if (amount == 0) {
            revert ZeroValue();
387: if (lockedBalance.endTime < (block.timestamp + 1)) {
            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);
469: if (lockedBalance.endTime < (block.timestamp + 1)) {
            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);
494: if (lockedBalance.endTime < (block.timestamp + 1)) {
            revert LockExpired(msg.sender, lockedBalance.endTime, block.timestamp);
392: if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);
449: if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);
474: if (amount > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);
465: if (lockedBalance.amount == 0) {
            revert NoValueLocked(msg.sender);
490: if (lockedBalance.amount == 0) {
            revert NoValueLocked(msg.sender);
```

| [Line #379](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L379) | [Line #427](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L427) | [Line #461](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L461) | [Line #387](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L387) | [Line #469](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L469) | [Line #494](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L494) | [Line #392](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L392) | [Line #449](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L449) | [Line #474](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L474) | [Line #465](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L465) | [Line #490](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L490) | 
```solidity
File: governance/contracts/multisigs/GuardCM.sol

155: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
171: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
448: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
501: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
562: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
```

| [Line #155](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155) | [Line #171](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171) | [Line #448](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448) | [Line #501](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501) | [Line #562](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562) | 
```solidity
File: governance/contracts/bridges/BridgedERC20.sol

32: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
50: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
61: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
```

| [Line #32](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32) | [Line #50](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50) | [Line #61](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61) | 
```solidity
File: registries/contracts/GenericRegistry.sol

39: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
55: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
80: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
```

| [Line #39](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L39) | [Line #55](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L55) | [Line #80](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L80) | 
```solidity
File: registries/contracts/UnitRegistry.sol

59: if (manager != msg.sender) {
            revert ManagerOnly(msg.sender, manager);
125: if (manager != msg.sender) {
            revert ManagerOnly(msg.sender, manager);
69: if (unitHash == 0) {
            revert ZeroValue();
139: if (unitHash == 0) {
            revert ZeroValue();
```

| [Line #59](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L59) | [Line #125](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L125) | [Line #69](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L69) | [Line #139](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L139) | 
```solidity
File: registries/contracts/GenericManager.sol

22: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
38: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
49: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
```

| [Line #22](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L22) | [Line #38](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L38) | [Line #49](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L49) | 
```solidity
File: tokenomics/contracts/Depository.sol

125: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
145: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
165: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
185: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
246: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
216: if (maturity > type(uint32).max) {
            revert Overflow(maturity, type(uint32).max);
311: if (maturity > type(uint32).max) {
            revert Overflow(maturity, type(uint32).max);
```

| [Line #125](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L125) | [Line #145](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L145) | [Line #165](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L165) | [Line #185](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L185) | [Line #246](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L246) | [Line #216](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L216) | [Line #311](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L311) | 
```solidity
File: tokenomics/contracts/Dispenser.sol

48: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
66: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
```

| [Line #48](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L48) | [Line #66](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L66) | 
```solidity
File: tokenomics/contracts/DonatorBlacklist.sol

38: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
58: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
```

| [Line #38](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L38) | [Line #58](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L58) | 
```solidity
File: tokenomics/contracts/Tokenomics.sol

386: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
406: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
425: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
452: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
476: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
506: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
571: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
611: if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
632: if (depository != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
1094: if (unitTypes.length != unitIds.length) {
            revert WrongArrayLength(unitTypes.length, unitIds.length);
1164: if (unitTypes.length != unitIds.length) {
            revert WrongArrayLength(unitTypes.length, unitIds.length);
1112: if (unitTypes[i] > 1) {
                revert Overflow(unitTypes[i], 1);
1182: if (unitTypes[i] > 1) {
                revert Overflow(unitTypes[i], 1);
1117: if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {
                revert WrongUnitId(unitIds[i], unitTypes[i]);
1187: if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {
                revert WrongUnitId(unitIds[i], unitTypes[i]);
1124: if (unitOwner != account) {
                revert OwnerOnly(unitOwner, account);
1194: if (unitOwner != account) {
                revert OwnerOnly(unitOwner, account);
```

| [Line #386](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L386) | [Line #406](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L406) | [Line #425](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L425) | [Line #452](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L452) | [Line #476](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L476) | [Line #506](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L506) | [Line #571](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L571) | [Line #611](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L611) | [Line #632](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L632) | [Line #1094](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1094) | [Line #1164](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1164) | [Line #1112](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1112) | [Line #1182](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1182) | [Line #1117](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1117) | [Line #1187](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1187) | [Line #1124](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1124) | [Line #1194](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1194) | 
```solidity
File: tokenomics/contracts/Treasury.sol

121: if (msg.value < minAcceptedETH) {
            revert LowerThan(msg.value, minAcceptedETH);
265: if (msg.value < minAcceptedETH) {
            revert LowerThan(msg.value, minAcceptedETH);
139: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
158: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
184: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
315: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
468: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
489: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
509: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
533: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
544: if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
391: if (paused == 2) {
            revert Paused();
430: if (paused == 2) {
            revert Paused();
```

| [Line #121](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L121) | [Line #265](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L265) | [Line #139](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L139) | [Line #158](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L158) | [Line #184](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L184) | [Line #315](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L315) | [Line #468](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L468) | [Line #489](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L489) | [Line #509](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L509) | [Line #533](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L533) | [Line #544](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L544) | [Line #391](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L391) | [Line #430](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L430) | </details>



### [NC-10] Consider Avoid Loops in Public Functions

Using loops within `public` or `external` functions can pose risks and inefficiencies.
Unpredictable gas consumption due to loop iterations can hinder a function's usability and cost-effectiveness. 
Furthermore, if the loop's logic can be externally influenced or altered, it might be exploited to intentionally drain gas, making the function impractical or uneconomical to call.
To ensure consistent performance and avoid potential vulnerabilities, it's advisable to avoid or limit loops in public functions, especially if their logic can change.

<details>
<summary><i>29 issue instances in 11 files:</i></summary>

```solidity
File: governance/contracts/OLAS.sol

108: for (uint256 i = 0; i < numYears; ++i) {
```

| [Line #108](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L108) | 
```solidity
File: governance/contracts/multisigs/GuardCM.sol

458: for (uint256 i = 0; i < targets.length; ++i) {
511: for (uint256 i = 0; i < chainIds.length; ++i) {
```

| [Line #458](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L458) | [Line #511](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L511) | 
```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

125: for (uint256 i = 0; i < dataLength;) {
153: for (uint256 j = 0; j < payloadLength; ++j) {
```

| [Line #125](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L125) | [Line #153](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L153) | 
```solidity
File: governance/contracts/bridges/HomeMediator.sol

125: for (uint256 i = 0; i < dataLength;) {
153: for (uint256 j = 0; j < payloadLength; ++j) {
```

| [Line #125](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L125) | [Line #153](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L153) | 
```solidity
File: registries/contracts/UnitRegistry.sol

98: for (uint256 i = 0; i < numSubComponents; ++i) {
```

| [Line #98](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L98) | 
```solidity
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

113: for (uint256 i = 0; i < payloadLength; ++i) {
139: for (uint256 i = 0; i < numOwners; ++i) {
```

| [Line #113](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L113) | [Line #139](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L139) | 
```solidity
File: tokenomics/contracts/Depository.sol

255: for (uint256 i = 0; i < numProducts; ++i) {
274: for (uint256 i = 0; i < numClosedProducts; ++i) {
357: for (uint256 i = 0; i < bondIds.length; ++i) {
402: for (uint256 i = 0; i < numProducts; ++i) {
413: for (uint256 i = 0; i < numProducts; ++i) {
448: for (uint256 i = 0; i < numBonds; ++i) {
468: for (uint256 i = 0; i < numBonds; ++i) {
```

| [Line #255](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L255) | [Line #274](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L274) | [Line #357](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L357) | [Line #402](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L402) | [Line #413](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L413) | [Line #448](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L448) | [Line #468](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L468) | 
```solidity
File: tokenomics/contracts/DonatorBlacklist.sol

67: for (uint256 i = 0; i < accounts.length; ++i) {
```

| [Line #67](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L67) | 
```solidity
File: tokenomics/contracts/Tokenomics.sol

808: for (uint256 i = 0; i < numServices; ++i) {
978: for (uint256 i = 0; i < 2; ++i) {
1104: for (uint256 i = 0; i < 2; ++i) {
1110: for (uint256 i = 0; i < unitIds.length; ++i) {
1132: for (uint256 i = 0; i < unitIds.length; ++i) {
1174: for (uint256 i = 0; i < 2; ++i) {
1180: for (uint256 i = 0; i < unitIds.length; ++i) {
1202: for (uint256 i = 0; i < unitIds.length; ++i) {
```

| [Line #808](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L808) | [Line #978](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L978) | [Line #1104](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1104) | [Line #1110](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1110) | [Line #1132](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1132) | [Line #1174](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1174) | [Line #1180](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1180) | [Line #1202](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1202) | 
```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

55: for (uint256 i = 0; i < numYears; ++i) {
92: for (uint256 i = 1; i < numYears; ++i) {
```

| [Line #55](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L55) | [Line #92](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L92) | 
```solidity
File: tokenomics/contracts/Treasury.sol

276: for (uint256 i = 0; i < numServices; ++i) {
```

| [Line #276](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L276) | </details>


### [NC-11] Consider moving `msg.sender` checks to a common authorization `modifier`

Functions that are only allowed to be called by a specific actor should use a modifier to check if the caller is the specified actor (e.g., owner, specific role, etc.).
Using `require` to check `msg.sender` in the function body is less efficient and less clear.

Consider refactoring these `require` statements into a modifier for better readability, organization, and gas efficiency.

<details>
<summary><i>57 issue instances in 13 files:</i></summary>

```solidity
File: governance/contracts/OLAS.sol

44: if (msg.sender != owner) {
59: if (msg.sender != owner) {
77: if (msg.sender != minter) {
```

| [Line #44](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L44) | [Line #59](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L59) | [Line #77](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L77) | 
```solidity
File: governance/contracts/multisigs/GuardCM.sol

155: if (msg.sender != owner) {
171: if (msg.sender != owner) {
448: if (msg.sender != owner) {
501: if (msg.sender != owner) {
540: if (msg.sender == owner) {
562: if (msg.sender != owner) {
```

| [Line #155](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L155) | [Line #171](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L171) | [Line #448](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L448) | [Line #501](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L501) | [Line #540](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L540) | [Line #562](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L562) | 
```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

84: if (msg.sender != address(this)) {
109: if(msg.sender != fxChild) {
```

| [Line #84](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L84) | [Line #109](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L109) | 
```solidity
File: governance/contracts/bridges/HomeMediator.sol

84: if (msg.sender != address(this)) {
107: if (msg.sender != AMBContractProxyHome) {
```

| [Line #84](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L84) | [Line #107](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L107) | 
```solidity
File: governance/contracts/bridges/BridgedERC20.sol

32: if (msg.sender != owner) {
50: if (msg.sender != owner) {
61: if (msg.sender != owner) {
```

| [Line #32](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L32) | [Line #50](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L50) | [Line #61](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol#L61) | 
```solidity
File: registries/contracts/GenericRegistry.sol

39: if (msg.sender != owner) {
55: if (msg.sender != owner) {
80: if (msg.sender != owner) {
```

| [Line #39](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L39) | [Line #55](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L55) | [Line #80](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol#L80) | 
```solidity
File: registries/contracts/UnitRegistry.sol

59: if (manager != msg.sender) {
125: if (manager != msg.sender) {
```

| [Line #59](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L59) | [Line #125](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L125) | 
```solidity
File: registries/contracts/GenericManager.sol

22: if (msg.sender != owner) {
38: if (msg.sender != owner) {
49: if (msg.sender != owner) {
```

| [Line #22](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L22) | [Line #38](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L38) | [Line #49](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L49) | 
```solidity
File: tokenomics/contracts/Depository.sol

125: if (msg.sender != owner) {
145: if (msg.sender != owner) {
165: if (msg.sender != owner) {
185: if (msg.sender != owner) {
246: if (msg.sender != owner) {
368: if (mapUserBonds[bondIds[i]].account != msg.sender) {
```

| [Line #125](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L125) | [Line #145](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L145) | [Line #165](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L165) | [Line #185](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L185) | [Line #246](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L246) | [Line #368](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L368) | 
```solidity
File: tokenomics/contracts/Dispenser.sol

48: if (msg.sender != owner) {
66: if (msg.sender != owner) {
```

| [Line #48](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L48) | [Line #66](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol#L66) | 
```solidity
File: tokenomics/contracts/DonatorBlacklist.sol

38: if (msg.sender != owner) {
58: if (msg.sender != owner) {
```

| [Line #38](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L38) | [Line #58](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L58) | 
```solidity
File: tokenomics/contracts/Tokenomics.sol

386: if (msg.sender != owner) {
406: if (msg.sender != owner) {
425: if (msg.sender != owner) {
452: if (msg.sender != owner) {
476: if (msg.sender != owner) {
506: if (msg.sender != owner) {
571: if (msg.sender != owner) {
611: if (depository != msg.sender) {
632: if (depository != msg.sender) {
795: if (treasury != msg.sender) {
1089: if (dispenser != msg.sender) {
```

| [Line #386](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L386) | [Line #406](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L406) | [Line #425](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L425) | [Line #452](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L452) | [Line #476](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L476) | [Line #506](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L506) | [Line #571](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L571) | [Line #611](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L611) | [Line #632](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L632) | [Line #795](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L795) | [Line #1089](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1089) | 
```solidity
File: tokenomics/contracts/Treasury.sol

139: if (msg.sender != owner) {
158: if (msg.sender != owner) {
184: if (msg.sender != owner) {
214: if (depository != msg.sender) {
315: if (msg.sender != owner) {
396: if (dispenser != msg.sender) {
435: if (msg.sender != tokenomics) {
468: if (msg.sender != owner) {
489: if (msg.sender != owner) {
509: if (msg.sender != owner) {
533: if (msg.sender != owner) {
544: if (msg.sender != owner) {
```

| [Line #139](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L139) | [Line #158](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L158) | [Line #184](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L184) | [Line #214](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L214) | [Line #315](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L315) | [Line #396](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L396) | [Line #435](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L435) | [Line #468](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L468) | [Line #489](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L489) | [Line #509](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L509) | [Line #533](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L533) | [Line #544](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L544) | </details>



### [NC-12] Unused Function Parameters

Function parameters that are not used within the function body can introduce confusion and reduce code clarity.
It's recommended to comment out the variable name or remove it if not needed. This will not only improve the readability of the code but also suppress any compiler warnings associated with unused parameters.
Ensure that removing or commenting out the parameter doesn't affect any function calls or overrides in derived contracts.

If function is `virtual`, it's recommended to remove the parameter name from the function declaration.

<details>
<summary><i>24 issue instances in 1 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

/// @audit `to` never used in function body or passed to modifiers
/// @audit `amount` never used in function body or passed to modifiers
767: function transfer(address to, uint256 amount) external virtual override returns (bool) {
/// @audit `spender` never used in function body or passed to modifiers
/// @audit `amount` never used in function body or passed to modifiers
772: function approve(address spender, uint256 amount) external virtual override returns (bool) {
/// @audit `from` never used in function body or passed to modifiers
/// @audit `to` never used in function body or passed to modifiers
/// @audit `amount` never used in function body or passed to modifiers
777: function transferFrom(address from, address to, uint256 amount) external virtual override returns (bool) {
/// @audit `owner` never used in function body or passed to modifiers
/// @audit `spender` never used in function body or passed to modifiers
782: function allowance(address owner, address spender) external view virtual override returns (uint256)
    {
/// @audit `account` never used in function body or passed to modifiers
788: function delegates(address account) external view virtual override returns (address)
    {
/// @audit `delegatee` never used in function body or passed to modifiers
794: function delegate(address delegatee) external virtual override
    {
/// @audit `delegatee` never used in function body or passed to modifiers
/// @audit `nonce` never used in function body or passed to modifiers
/// @audit `expiry` never used in function body or passed to modifiers
/// @audit `v` never used in function body or passed to modifiers
/// @audit `r` never used in function body or passed to modifiers
/// @audit `s` never used in function body or passed to modifiers
800: function delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
    external virtual override
    {
```

| [Line #767](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L767) | [Line #772](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L772) | [Line #777](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L777) | [Line #782](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L782) | [Line #788](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L788) | [Line #794](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L794) | [Line #800](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L800) | </details>


### [NC-13] Consider using `delete` rather than assigning `false` to clear values

The use of the delete keyword is recommended over simply assigning values to false when you intend to reset the state of a variable.
The delete keyword more closely aligns with the semantic intent of clearing or resetting a variable.
This practice also makes the code more readable and highlights the change in state, which may encourage a more thorough audit of the surrounding logic.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: registries/contracts/GenericManager.sol

52: paused = false;
```

| [Line #52](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol#L52) | 
```solidity
File: tokenomics/contracts/Treasury.sol

455: success = false;
518: mapEnabledTokens[token] = false;
```

| [Line #455](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L455) | [Line #518](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L518) | </details>


### [NC-14] Style guide: Non-`external`/`public` variable names should begin with an underscore

The naming convention for non-public (private and internal) variables in Solidity recommends the use of a leading underscore.

Since `constants` can be public, to avoid confusion, they should also be prefixed with an underscore.

This practice clearly differentiates between public/external and non-public variables, enhancing code clarity and reducing the likelihood of misinterpretation or errors.
Following this convention improves code readability and maintainability.

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

99: uint64 internal constant WEEK = 1 weeks;
101: uint256 internal constant MAXTIME = 4 * 365 * 86400;
103: int128 internal constant IMAXTIME = 4 * 365 * 86400;
```

| [Line #99](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L99) | [Line #101](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L101) | [Line #103](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L103) | </details>


### [NC-15] Double Type Casting

Double type casting should be avoided in Solidity contracts to prevent unintended consequences and ensure accurate data representation.
Performing multiple type casts in succession can lead to unexpected truncation, rounding errors, or loss of precision, potentially compromising the contract's functionality and reliability.
Furthermore, double type casting can make the code less readable and harder to maintain, increasing the likelihood of errors and misunderstandings during development and debugging.
To ensure precise and consistent data handling, developers should use appropriate data types and avoid unnecessary or excessive type casting, promoting a more robust and dependable contract execution.

<details>
<summary><i>16 issue instances in 2 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

190: uOld.bias = uOld.slope * int128(uint128(oldLocked.endTime - uint64(block.timestamp)));
194: uNew.bias = uNew.slope * int128(uint128(newLocked.endTime - uint64(block.timestamp)));
245: lastPoint.bias -= lastPoint.slope * int128(int64(tStep - lastCheckpoint));
597: uPoint.bias -= uPoint.slope * int128(int64(ts) - int64(uPoint.ts));
599: vBalance = uint256(int256(uPoint.bias));
680: uPoint.bias -= uPoint.slope * int128(int64(uint64(blockTime)) - int64(uPoint.ts));
682: balance = uint256(uint128(uPoint.bias));
704: lastPoint.bias -= lastPoint.slope * int128(int64(tStep) - int64(lastPoint.ts));
713: vSupply = uint256(uint128(lastPoint.bias));
```

| [Line #190](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L190) | [Line #194](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L194) | [Line #245](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L245) | [Line #597](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L597) | [Line #599](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L599) | [Line #680](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L680) | [Line #682](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L682) | [Line #704](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L704) | [Line #713](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L713) | 
```solidity
File: governance/contracts/multisigs/GuardCM.sol

192: uint256 targetSelectorChainId = uint256(uint160(target));
194: targetSelectorChainId |= uint256(uint32(bytes4(data))) << 160;
476: uint256 targetSelectorChainId = uint256(uint160(targets[i]));
478: targetSelectorChainId |= uint256(uint32(selectors[i])) << 160;
525: uint256 bridgeMediatorL2ChainId = uint256(uint160(bridgeMediatorL2s[i]));
584: uint256 targetSelectorChainId = uint256(uint160(target));
586: targetSelectorChainId |= uint256(uint32(selector)) << 160;
```

| [Line #192](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L192) | [Line #194](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L194) | [Line #476](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L476) | [Line #478](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L478) | [Line #525](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L525) | [Line #584](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L584) | [Line #586](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L586) | </details>


### [NC-16] Use Unchecked for Divisions on Constant or Immutable Values

Unsigned divisions on constant or immutable values do not result in overflow.
Therefore, these operations can be marked as unchecked, optimizing gas usage without compromising safety.

For instance, if `a` is an unsigned integer and `b` is a constant or immutable, a / b can be safely rewritten as: unchecked { a / b }

<details>
<summary><i>8 issue instances in 2 files:</i></summary>

```solidity
File: governance/contracts/OLAS.sol

101: uint256 numYears = (block.timestamp - timeLaunch) / oneYear;
```

| [Line #101](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L101) | 
```solidity
File: governance/contracts/veOLAS.sol

14: # w ^ = amount * time_locked / MAXTIME
189: uOld.slope = int128(oldLocked.amount) / IMAXTIME;
193: uNew.slope = int128(newLocked.amount) / IMAXTIME;
231: uint64 tStep = (lastCheckpoint / WEEK) * WEEK;
433: unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
487: unlockTime = ((block.timestamp + unlockTime) / WEEK) * WEEK;
692: uint64 tStep = (lastPoint.ts / WEEK) * WEEK;
```

| [Line #14](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L14) | [Line #189](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L189) | [Line #193](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L193) | [Line #231](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L231) | [Line #433](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L433) | [Line #487](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L487) | [Line #692](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L692) | </details>


### [NC-17] Missing Event Emission After Critical Initialize() Function

Emitting an initialization event offers clear, on-chain evidence of the contract's initialization state, enhancing transparency and auditability.
This practice aids users and developers in accurately tracking the contract's lifecycle, pinpointing the precise moment of its initialization.
Moreover, it aligns with best practices for event logging in smart contracts, ensuring that significant state changes are both observable and verifiable through emitted events.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: tokenomics/contracts/Tokenomics.sol

264: function initializeTokenomics(
        address _olas,
        address _treasury,
        address _depository,
        address _dispenser,
        address _ve,
        uint256 _epochLen,
        address _componentRegistry,
        address _agentRegistry,
        address _serviceRegistry,
        address _donatorBlacklist
    ) external
```
[264](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L264)
</details>

### [NC-18] Variables need not be initialized to zero

By default, `int/uint` variables in Solidity are initialized to `zero`.
Explicitly setting variables to zero during their declaration is redundant and might cause confusion.
Removing the explicit zero initialization can improve code readability and understanding.

<details>
<summary><i>48 issue instances in 15 files:</i></summary>

```solidity
File: governance/contracts/OLAS.sol

108: for (uint256 i = 0; i < numYears; ++i) {
```
[108](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/OLAS.sol#L108)

```solidity
File: governance/contracts/veOLAS.sol

232: for (uint256 i = 0; i < 255; ++i) {
561: for (uint256 i = 0; i < 128; ++i) {
693: for (uint256 i = 0; i < 255; ++i) {
```
[232](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L232) | [561](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L561) | [693](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L693)

```solidity
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
[212](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L212) | [237](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L237) | [273](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L273) | [292](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L292) | [318](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L318) | [340](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L340) | [360](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L360) | [458](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L458) | [511](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol#L511)

```solidity
File: governance/contracts/bridges/FxGovernorTunnel.sol

125: for (uint256 i = 0; i < dataLength;) {
153: for (uint256 j = 0; j < payloadLength; ++j) {
```
[125](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L125) | [153](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol#L153)

```solidity
File: governance/contracts/bridges/HomeMediator.sol

125: for (uint256 i = 0; i < dataLength;) {
153: for (uint256 j = 0; j < payloadLength; ++j) {
```
[125](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L125) | [153](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol#L153)

```solidity
File: registries/contracts/UnitRegistry.sol

98: for (uint256 i = 0; i < numSubComponents; ++i) {
211: for (uint32 i = 0; i < numUnits; ++i) {
234: for (uint32 i = 0; i < numUnits; ++i) {
262: for (uint32 i = 0; i < counter; ++i) {
```
[98](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L98) | [211](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L211) | [234](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L234) | [262](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L262)

```solidity
File: registries/contracts/ComponentRegistry.sol

29: for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
```
[29](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/ComponentRegistry.sol#L29)

```solidity
File: registries/contracts/AgentRegistry.sol

40: for (uint256 iDep = 0; iDep < dependencies.length; ++iDep) {
```
[40](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol#L40)

```solidity
File: registries/contracts/multisigs/GnosisSafeMultisig.sol

79: for (uint256 i = 0; i < payloadLength; ++i) {
```
[79](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol#L79)

```solidity
File: registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

113: for (uint256 i = 0; i < payloadLength; ++i) {
139: for (uint256 i = 0; i < numOwners; ++i) {
```
[113](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L113) | [139](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L139)

```solidity
File: tokenomics/contracts/Depository.sol

255: for (uint256 i = 0; i < numProducts; ++i) {
274: for (uint256 i = 0; i < numClosedProducts; ++i) {
357: for (uint256 i = 0; i < bondIds.length; ++i) {
402: for (uint256 i = 0; i < numProducts; ++i) {
413: for (uint256 i = 0; i < numProducts; ++i) {
448: for (uint256 i = 0; i < numBonds; ++i) {
468: for (uint256 i = 0; i < numBonds; ++i) {
```
[255](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L255) | [274](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L274) | [357](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L357) | [402](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L402) | [413](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L413) | [448](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L448) | [468](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L468)

```solidity
File: tokenomics/contracts/DonatorBlacklist.sol

67: for (uint256 i = 0; i < accounts.length; ++i) {
```
[67](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol#L67)

```solidity
File: tokenomics/contracts/Tokenomics.sol

702: for (uint256 i = 0; i < numServices; ++i) {
714: for (uint256 unitType = 0; unitType < 2; ++unitType) {
728: for (uint256 j = 0; j < numServiceUnits; ++j) {
758: for (uint256 j = 0; j < numServiceUnits; ++j) {
808: for (uint256 i = 0; i < numServices; ++i) {
978: for (uint256 i = 0; i < 2; ++i) {
1104: for (uint256 i = 0; i < 2; ++i) {
1110: for (uint256 i = 0; i < unitIds.length; ++i) {
1132: for (uint256 i = 0; i < unitIds.length; ++i) {
1174: for (uint256 i = 0; i < 2; ++i) {
1180: for (uint256 i = 0; i < unitIds.length; ++i) {
1202: for (uint256 i = 0; i < unitIds.length; ++i) {
```
[702](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L702) | [714](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L714) | [728](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L728) | [758](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L758) | [808](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L808) | [978](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L978) | [1104](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1104) | [1110](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1110) | [1132](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1132) | [1174](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1174) | [1180](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1180) | [1202](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1202)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

55: for (uint256 i = 0; i < numYears; ++i) {
```
[55](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/TokenomicsConstants.sol#L55)

```solidity
File: tokenomics/contracts/Treasury.sol

276: for (uint256 i = 0; i < numServices; ++i) {
```
[276](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L276)
</details>


### [NC-19] Consider using descriptive `constant`s when passing zero as a function argument

In instances where utilizing a zero parameter is essential, it is recommended to employ descriptive constants or an enum instead of directly integrating zero within function calls.
This strategy aids in clearly articulating the caller's intention and minimizes the risk of errors.
Emitting zero also not recomended, as it is not clear what the intention is.

<details>
<summary><i>6 issue instances in 3 files:</i></summary>

```solidity
File: governance/contracts/veOLAS.sol

396: _depositFor(account, amount, 0, lockedBalance, DepositType.DEPOSIT_FOR_TYPE)
478: _depositFor(msg.sender, amount, 0, lockedBalance, DepositType.INCREASE_LOCK_AMOUNT)
506: _depositFor(msg.sender, 0, unlockTime, lockedBalance, DepositType.INCREASE_UNLOCK_TIME)
```
[396](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L396) | [478](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L478) | [506](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L506)

```solidity
File: governance/contracts/wveOLAS.sol

213: getUserPoint(account, 0)
233: getUserPoint(account, 0)
```
[213](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L213) | [233](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/wveOLAS.sol#L233)

```solidity
File: tokenomics/contracts/Tokenomics.sol

329: getInflationForYear(0)
```
[329](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L329)
</details>

