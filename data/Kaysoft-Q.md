## [L-1] Security vulnerability with `GovernorCompatibilityBravo` version 4.8.3

There is 1 instance of this
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/GovernorOLAS.sol#L15

According to Openzeppelin's security disclosure here: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-5h3x-9wvq-w4m2

There is a security vulnerability with the `GovernorCompatibilityBravo` contract particularly the `propose(...)` function which is used in the `GovernorOLAS` contract due to the order of the inherited contracts.

```
contract GovernorOLAS is Governor, GovernorSettings, GovernorCompatibilityBravo, GovernorVotes, GovernorVotesQuorumFraction, GovernorTimelockControl {
```
Due to the order of the parent contracts and how Solidity inheritance work, the `super` keyword in the `propose(...)` function below invokes the vulnerable  `GovernorCompatibilityBravo.propose(...)` function.

```
File: governance/contracts/GovernorOLAS.sol
45: function propose(
46:        address[] memory targets,
47:        uint256[] memory values,
48:        bytes[] memory calldatas,
49:        string memory description
50:    ) public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)
51:    {
52:        return super.propose(targets, values, calldatas, description);
53:    }
```
 References 
- https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-5h3x-9wvq-w4m2

 Recommendation: Consider updating the openzeppelin library to version 4.9.3.

## [L-2] Avoid variable shadowing.
There are 3 instances of this.
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/ComponentRegistry.sol#L20

- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/GovernorOLAS.sol#L18

1. Constructor parameter `ComponentRegistry#constructor._baseURI` shadows the parent's `ERC721._baseURI` storage variable. Variable shadowing can easily lead to a security issue when one of the two variables is referenced instead of the other.

```
File: registry/contracts/ComponentRegistry.sol
16:constructor(string memory _name, string memory _symbol, string memory _baseURI)
17:        UnitRegistry(UnitType.Component)
18:        ERC721(_name, _symbol)
19:    {
20:        baseURI = _baseURI;//@audit-info variable shadowing ERC721._baseURI
21:        owner = msg.sender;
22:    }
``` 
2. The `timelock` parameter of `GovernorOLAS.sol#constructor(...)` shadows `IGovernorTimelock.timelock`.

see SWC-119 here: https://swcregistry.io/docs/SWC-119/

 Recommendation: Consider renaming the `ComponentRegistry#constructor._baseURI` constructor parameter.

## [L-3] Use already existing Openzeppelin libraries where possible.
open source libraries like OZ have undergone several security audits, have stood the test of time, have been battle tested so it is safer to use these libraries that also reduce development time and also makes the code modular.

1. Use Openzeppelin's Ownable2step  library instead of creating another ownable implementation and duplicating it in most contracts.

There are 4 instances of this
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/BridgedERC20.sol#L25C4-L44C1
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L36C9-L54C6
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericManager.sol#L14C5-L34C1
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L37C5-L51C1

2. Use Openzeppelin Pausable library instead of creating an implementation that can be prone to error.

There are 2 instancees of this:
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericManager.sol#L36C5-L55C6
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L531C5-L550C6

3. Use Openzeppelin's ReentrancyGuard library instead of creating an implementation that can be prone to error.

There are 3 instances of this:
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/audits/internal/analysis/reentrancyPoC/ServiceRegistry.sol#L613C8-L617C21
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Dispenser.sol#L92C9-L96C21
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L258C9-L262C21

4. Use Openzepelin's ERC721Enumerable the implemented on in this project is incorrect.

The GenericRegistry ERC721 contract implements the `tokenByIndex(...)` function which is an ERC721Enumerable feature. Consider using Openzeppelin's ERC721Enumerable to avoid potential issues.

There is 1 instance of this:
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L97C5-L102C6


## [L-4] USE THE RIGHT ERROR NAME
There is 1 instances of this:

- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/GenericRegistry.sol#L100

The errorname used in the if condition below is wrong. it should be something like "Unavailable" or "NotFound".

```
File: registries/contracts/GenericRegistry.sol
97: function tokenByIndex(uint256 id) external view virtual returns (uint256 unitId) {
98:        unitId = id + 1;
99:        if (unitId > totalSupply) {
100:            revert Overflow(unitId, totalSupply);//@audit this is not overflow error
101:        }
    }
```

Recommendation: use an error name that reflects the condition that happened in the function.

## [L-5] UNNECESSARY STORAGE VARIABLE UPDATE.
There is 1 instance of this

- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L291

Since the `_locked` storage variable is used for reentrancy guard, there is no need setting the value of `_locked` in the initialize function of the Tokenomics function.

```
291: function initializeTokenomics(
292:        address _olas,
293:        address _treasury,
294:        address _depository,
295:        address _dispenser,
296:        address _ve,
297:        uint256 _epochLen,
298:        address _componentRegistry,
299:        address _agentRegistry,
300:        address _serviceRegistry,
301:        address _donatorBlacklist
302:     ) external
303:    {
...
264: _locked = 1;//@audit unnecessary state update since there is no reentrance risk in initialze function.
...
370: }

```
Recommendation: remove `_locked` state update above.

## [L-6] Tokenomics.sol implementation contract can be initialized
There is 1 instance:
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L264C5-L370C6

The Tokenomics.sol contract is the implementation contract for the TokenomicsProxy.sol proxy contract. The `TokenomicsProxy.sol` is expected to call the initialize function of its `Tokenomic.sol` implementation through a delegate call. 
In a situation where the `Tokenomics.sol` implementation contract is not separately initialized, anyone can initialize the `Tokenomics.sol` implementation contract which could lead to potential security issue for the proxy contract for example if an implementation contract can be taken over and destroyed.

```
File: tokenomics/contracts/Tokenomics.sol
264: function initializeTokenomics(
265:        address _olas,
266:        address _treasury,
267:        address _depository,
268:        address _dispenser,
269:        address _ve,
270:        uint256 _epochLen,
271:        address _componentRegistry,
272:        address _agentRegistry,
273:        address _serviceRegistry,
274:        address _donatorBlacklist
275:    ) external
276:    {
277:        // Check if the contract is already initialized
278:        if (owner != address(0)) {
279:            revert AlreadyInitialized();
280:        }
281:
282:        // Check for at least one zero contract address
283:        if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||
284:            _ve == address(0) || _componentRegistry == address(0) || _agentRegistry == address(0) ||
285:            _serviceRegistry == address(0)) {
286:            revert ZeroAddress();
287:        }
...
370:}

```

Recommendation : Use Openzepelin's initializable library, initializable modifier on the `Tokenomics.sol#initialize(...)` founction and it's `_disableInitializers()` function in the constructor of the `Tokenomics.sol` implementation contract as below.

```
import { Initializable} from "OpenZeppelin/openzeppelin-contracts/contracts/proxy/utils/Initializable.sol";

Tokenomics is Initializable {

/// @custom:oz-upgrades-unsafe-allow constructor
 * constructor() {
 *     _disableInitializers();
 * }

function initialize(...) initializer public {

...
}
...
}
```

