# G-01. Using a positive conditional flow to save a NOT opcode #

Estimated savings: 3 gas per instance

Reference : https://code4rena.com/reports/2023-07-basin#g-13-using-a-positive-conditional-flow-to-save-a-not-opcode

Instances : 
```
File : governance/contracts/bridges/FxERC20ChildTunnel.sol

80 :         if (!success) {

103 :         if (!success) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20ChildTunnel.sol#L80

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20ChildTunnel.sol#L103
```
File : governance/contracts/bridges/FxERC20RootTunnel.sol

99 :         if (!success) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20RootTunnel.sol#L99
```
File : governance/contracts/bridges/HomeMediator.sol

161 :             if (!success) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/HomeMediator.sol#L161
```
File : registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol

119 :             if (!success) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L119
```
File : tokenomics/contracts/Dispenser.sol

108 :         if (!success) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Dispenser.sol#L108
```
File : tokenomics/contracts/TokenomicsProxy.sol

49 :         if (!success) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsProxy.sol#L49
```
File : tokenomics/contracts/Treasury.sol

236 :         if (!success) {

340 :                 if (!success) {

363 :                 if (!success) {

407 :             if (!success) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L236

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L340

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L363

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L407


# G-02. Gas saving is achieved by removing the delete keyword (~60k) #

30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

Reference : https://code4rena.com/reports/2023-06-lukso#g-10-gas-saving-is-achieved-by-removing-the-delete-keyword-60k

Instances :
```
File : tokenomics/contracts/Depository.sol

264 :                 delete mapBondProducts[productId];

341 :             delete mapBondProducts[productId];

379 :             delete mapUserBonds[bondIds[i]];
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L264

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L341

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L379

# G-03. Variables: “constants” expressions are expressions, not constants #

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

If the variable was immutable instead: the calculation would only be done once at deploy time, and then the result would be saved and read directly at runtime rather than being recalculated.

Consequences: each usage of a “constant” costs ~100gas more on each access (it is still a little better than storing the result in storage, but not much..). since these are not real constants, they can’t be referenced from a real constant environment (e.g. from assembly, or from another library )

Reference : https://code4rena.com/reports/2022-03-lifinance#g-04-variables-constants-expressions-are-expressions-not-constants
```
File : governance/contracts/multisigs/GuardCM.sol

97 :     bytes4 public constant SCHEDULE = bytes4(keccak256(bytes("schedule(address,uint256,bytes,bytes32,bytes32,uint256)")));

99 :     bytes4 public constant SCHEDULE_BATCH = bytes4(keccak256(bytes("scheduleBatch(address[],uint256[],bytes[],bytes32,bytes32,uint256)")));

101 :     bytes4 public constant REQUIRE_TO_PASS_MESSAGE = bytes4(keccak256(bytes("requireToPassMessage(address,bytes,uint256)")));

103 :     bytes4 public constant PROCESS_MESSAGE_FROM_FOREIGN = bytes4(keccak256(bytes("processMessageFromForeign(bytes)")));

105 :     bytes4 public constant SEND_MESSAGE_TO_CHILD = bytes4(keccak256(bytes("sendMessageToChild(address,bytes)")));
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L97

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L99

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L101

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L103

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L105

# G-04. Use constants instead of type(uintx).max #

using type(uintx).max can result in higher gas costs because it involves a runtime operation to calculate the maximum value at runtime. This calculation is performed every time the expression is evaluated.

To save gas, it is recommended to use constants instead of type(uintx).max to represent the maximum value. By declaring a constant with the maximum value, the value is known at compile-time and does not require any runtime calculations.

Reference : https://code4rena.com/reports/2023-07-basin#g-05-use-constants-instead-of-typeuintxmax

Instances : 
```
File: governance/contracts/veOLAS.sol

392 :         if (amount > type(uint96).max) {

449 :         if (amount > type(uint96).max) {

474 :         if (amount > type(uint96).max) {
 ```
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L392
 
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L449
 
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L474
 ```
 File : lockbox-solana/solidity/liquidity_lockbox.sol
 
 95 :         if (positionData.liquidity > type(uint64).max) {
 ```
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/lockbox-solana/solidity/liquidity_lockbox.sol#L95
 ```
 File : registries/contracts/UnitRegistry.sol
 
 232 :             uint32 tryMinComponent = type(uint32).max;
 ```
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/UnitRegistry.sol#L232
 ```
 File : tokenomics/contracts/Depository.sol
 
 195 :         if (priceLP > type(uint160).max) {
 
 205 :         if (supply > type(uint96).max) {
 
 216 :         if (maturity > type(uint32).max) {
 
 311 :         if (maturity > type(uint32).max) {
 ```
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L195
 
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L205
 
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L216
 
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L311
 ```
 FIle : tokenomics/contracts/GenericBondCalculator.sol
 
 59 :         if (totalTokenValue > type(uint192).max) {
 ```
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/GenericBondCalculator.sol#L59
 ```
 File : tokenomics/contracts/Tokenomics.sol
 
 639 :         if (eBond > type(uint96).max) {
 ```
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L639
 ```
 File : tokenomics/contracts/Treasury.sol
 
 128 :         if (amount + ETHFromServices > type(uint96).max) {
 
 194 :         if (_minAcceptedETH > type(uint96).max) {
 
 291 :         if (donationETH + ETHOwned > type(uint96).max) {
 ```
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L128
 
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L194
 
 https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L291

# G-05. abi.encode() is less efficient than abi.encodepacked() #

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Reference : https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

Instances :
```
File : governance/contracts/bridges/FxERC20ChildTunnel.sol

97 :         bytes memory message = abi.encode(msg.sender, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20ChildTunnel.sol#L97
```
File : governance/contracts/bridges/FxERC20RootTunnel.sol

93 :         bytes memory message = abi.encode(msg.sender, to, amount);
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20RootTunnel.sol#L93

# G-06. OR in if-condition can be rewritten to two single if conditions #

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

Reference : https://code4rena.com/reports/2023-08-shell#g-05-or-in-if-condition-can-be-rewritten-to-two-single-if-conditions

Instances :
```
File : governance/contracts/bridges/FxERC20RootTunnel.sol

42 :         if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
            _rootToken == address(0)) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20RootTunnel.sol#L42C9-L43C40
```
File : governance/contracts/multisigs/GuardCM.sol

144 :         if (_timelock == address(0) || _multisig == address(0) || _governor == address(0)) {

453 :         if (targets.length != selectors.length || targets.length != statuses.length || targets.length != chainIds.length) {

506 :         if (bridgeMediatorL1s.length != bridgeMediatorL2s.length || bridgeMediatorL1s.length != chainIds.length) {

513 :             if (bridgeMediatorL1s[i] == address(0) || bridgeMediatorL2s[i] == address(0)) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L144

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L453

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L506

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/multisigs/GuardCM.sol#L513
```
File : registries/contracts/AgentRegistry.sol

41 :             if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > componentTotalSupply) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/AgentRegistry.sol#L41
```
File : registries/contracts/ComponentRegistry.sol

30 :             if (dependencies[iDep] < (lastId + 1) || dependencies[iDep] > maxComponentId) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/ComponentRegistry.sol#L30
```
File : tokenomics/contracts/Depository.sol

111 :         if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) || _bondCalculator == address(0)) {

363 :             if (pay == 0 || !matured) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L111

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L363
```
File : tokenomics/contracts/Dispenser.sol

36 :         if (_tokenomics == address(0) || _treasury == address(0)) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Dispenser.sol#L36
```
File : tokenomics/contracts/GenericBondCalculator.sol

31 :         if (_olas == address(0) || _tokenomics == address(0)) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/GenericBondCalculator.sol#L31
```
File : tokenomics/contracts/Tokenomics.sol

1117 :             if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {

1187 :             if (unitIds[i] <= lastIds[unitTypes[i]] || unitIds[i] > registriesSupply[unitTypes[i]]) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L1117

https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L1187
```
File : tokenomics/contracts/Treasury.sol

100 :         if (_olas == address(0) || _tokenomics == address(0) || _depository == address(0) || _dispenser == address(0)) {
```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L100

