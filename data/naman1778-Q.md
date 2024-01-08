## [N-01] Typos

    File: governance/contracts/multisigs/GuardCM.sol	

    /// @audit Numberf
    40: /// @param numValues2 Numberf of values in a second array.

    /// @audit Numberf
    41: /// @param numValues3 Numberf of values in a third array.

    /// @audit Numberf
    42: /// @param numValues4 Numberf of values in a fourth array.

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol

    File: governance/contracts/bridges/HomeMediator.sol	

    /// @audit fo
    63: // Check fo zero addresses

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol

    File: registries/contracts/interfaces/IErrorsRegistries.sol	

    /// @audit Numberf
    21: /// @param numValues2 Numberf of values in a second array.

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/interfaces/IErrorsRegistries.sol

## [N-02] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

    File: governance/contracts/multisigs/GuardCM.sol	

    155: if (msg.sender != owner) {
    156:     revert OwnerOnly(msg.sender, owner);
    157: }

    171: if (msg.sender != owner) {
    172:     revert OwnerOnly(msg.sender, owner);
    173: }

    448: if (msg.sender != owner) {
    449:     revert OwnerOnly(msg.sender, owner);
    450: }

    501: if (msg.sender != owner) {
    502:     revert OwnerOnly(msg.sender, owner);
    503: }

    562: if (msg.sender != owner) {
    563:     revert OwnerOnly(msg.sender, owner);
    564: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol

    File: governance/contracts/bridges/FxGovernorTunnel.sol	

    84: if (msg.sender != address(this)) {
    85:     revert SelfCallOnly(msg.sender, address(this));
    86: }

    109: if(msg.sender != fxChild) {
    110:     revert FxChildOnly(msg.sender, fxChild);
    111: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxGovernorTunnel.sol

    File: governance/contracts/bridges/HomeMediator.sol	

    84: if (msg.sender != address(this)) {
    85:     revert SelfCallOnly(msg.sender, address(this));
    86: }

    107: if (msg.sender != AMBContractProxyHome) {
    108:     revert AMBContractProxyHomeOnly(msg.sender, AMBContractProxyHome);
    109: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/HomeMediator.sol

    File: governance/contracts/bridges/BridgedERC20.sol	

    32: if (msg.sender != owner) {
    33:     revert OwnerOnly(msg.sender, owner);
    34: }

    50: if (msg.sender != owner) {
    51:     revert OwnerOnly(msg.sender, owner);
    52: }

    61: if (msg.sender != owner) {
    62:     revert OwnerOnly(msg.sender, owner);
    63: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/BridgedERC20.sol

    File: registries/contracts/GenericRegistry.sol	

    39: if (msg.sender != owner) {
    40:     revert OwnerOnly(msg.sender, owner);
    41: }

    55: if (msg.sender != owner) {
    56:     revert OwnerOnly(msg.sender, owner);
    57: }

    80: if (msg.sender != owner) {
    81:     revert OwnerOnly(msg.sender, owner);
    82: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericRegistry.sol

    File: registries/contracts/UnitRegistry.sol	

    59: if (manager != msg.sender) {
    60:     revert ManagerOnly(msg.sender, manager);
    61: }

    125: if (manager != msg.sender) {
    126:     revert ManagerOnly(msg.sender, manager);
    127: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol

    File: registries/contracts/GenericManager.sol	

    22: if (msg.sender != owner) {
    23:     revert OwnerOnly(msg.sender, owner);
    24: }

    38: if (msg.sender != owner) {
    39:     revert OwnerOnly(msg.sender, owner);
    40: }

    49: if (msg.sender != owner) {
    50:     revert OwnerOnly(msg.sender, owner);
    51: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/GenericManager.sol

    File: tokenomics/contracts/Depository.sol	

    125: if (msg.sender != owner) {
    126:     revert OwnerOnly(msg.sender, owner);
    127: }

    145: if (msg.sender != owner) {
    146:     revert OwnerOnly(msg.sender, owner);
    147: }

    165: if (msg.sender != owner) {
    166:     revert OwnerOnly(msg.sender, owner);
    167: }

    185: if (msg.sender != owner) {
    186:     revert OwnerOnly(msg.sender, owner);
    187: }

    246: if (msg.sender != owner) {
    247:     revert OwnerOnly(msg.sender, owner);
    248: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol

    File: tokenomics/contracts/Dispenser.sol	

    48: if (msg.sender != owner) {
    49:     revert OwnerOnly(msg.sender, owner);
    50: }

    66: if (msg.sender != owner) {
    67:     revert OwnerOnly(msg.sender, owner);
    68: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Dispenser.sol

    File: tokenomics/contracts/DonatorBlacklist.sol	

    38: if (msg.sender != owner) {
    39:     revert OwnerOnly(msg.sender, owner);
    40: }

    58: if (msg.sender != owner) {
    59:     revert OwnerOnly(msg.sender, owner);
    60: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/DonatorBlacklist.sol

    File: tokenomics/contracts/Tokenomics.sol	

    386: if (msg.sender != owner) {
    387:     revert OwnerOnly(msg.sender, owner);
    388: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol

    File: tokenomics/contracts/Treasury.sol	

    139: if (msg.sender != owner) {
    140:     revert OwnerOnly(msg.sender, owner);
    141: }

    158: if (msg.sender != owner) {
    159:     revert OwnerOnly(msg.sender, owner);
    160: }

    184: if (msg.sender != owner) {
    185:     revert OwnerOnly(msg.sender, owner);
    186: }

    214: if (depository != msg.sender) {
    215:     revert ManagerOnly(msg.sender, depository);
    216: }

    315: if (msg.sender != owner) {
    316:     revert OwnerOnly(msg.sender, owner);
    317: }

    396: if (dispenser != msg.sender) {
    397:     revert ManagerOnly(msg.sender, dispenser);
    398: }

    435: if (msg.sender != tokenomics) {
    436:     revert ManagerOnly(msg.sender, tokenomics);
    437: }

    468: if (msg.sender != owner) {
    469:     revert OwnerOnly(msg.sender, owner);
    470: }

    489: if (msg.sender != owner) {
    490:     revert OwnerOnly(msg.sender, owner);
    491: }

    509: if (msg.sender != owner) {
    510:     revert OwnerOnly(msg.sender, owner);
    511: }

    533: if (msg.sender != owner) {
    534:     revert OwnerOnly(msg.sender, owner);
    535: }

    544: if (msg.sender != owner) {
    545:     revert OwnerOnly(msg.sender, owner);
    546: }

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol%5D

## [N-03] Lack of *address(0)* checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

    File: governance/contracts/veOLAS.sol	

    132: constructor(address _token, string memory _name, string memory _symbol)

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol

    File: registries/contracts/AgentRegistry.sol	

    20: constructor(string memory _name, string memory _symbol, string memory _baseURI, address _componentRegistry)

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/AgentRegistry.sol

    File: registries/contracts/RegistriesManager.sol	

    15: constructor(address _componentRegistry, address _agentRegistry) {

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/RegistriesManager.sol

    File: registries/contracts/multisigs/GnosisSafeMultisig.sol	

    37: constructor (address payable _gnosisSafe, address _gnosisSafeProxyFactory) {

https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/multisigs/GnosisSafeMultisig.sol

## [N-04] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

    File: tokenomics/contracts/Depository.sol	

    225: mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;

    229: mapping(uint256 => mapping(uint256 => IncentiveBalances)) public mapUnitIncentives;

https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol

## [N-05] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

1) splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
2) clearly documenting the functions and implementations the owner can change,
3) documenting the risks associated with privileged users and single points of failure, and
4) ensuring that users are aware of all the risks associated with the system.

## [N-06] Using Vulnerable Version of OpenZeppelin

The package.json configuration file says that the project is using 4.8.3 of OpenZeppelin which has a vulnerability.

Although there is no security vulnerability covering the project, it is recommended to use the latest version 5.0.1.

    File: governance/package.json

    28: "@openzeppelin/contracts": "=4.8.3",

https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/package.json