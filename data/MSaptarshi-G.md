## [G-01] solidity compiler versions doesn't match
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/GovernorOLAS.sol#L2
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L2
........
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxERC20RootTunnel.sol#L2
## [G-02] while importing contracts import them with name 
```- import "@openzeppelin/contracts/token/ERC20/IERC20.sol";```
```+ import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";```
## [G-03] Solidity restricts 120 charecters per line for good redeabality
`import {GovernorTimelockControl, TimelockController} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";
`
## [G-04] solidity code format not correct
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/veOLAS.sol#L1
### Recomendation
Here struct is initialised before initialising contract
