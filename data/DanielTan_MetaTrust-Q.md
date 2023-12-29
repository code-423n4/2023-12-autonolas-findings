# `changeOwner` and `changeMinter` should be two steps process

The `OLAS` contract enables the `owner` role to change the `owner` to another address. It’s possible that the `owner` role mistakenly transfers ownership to the wrong address, resulting in a loss of the `owner` role. The current ownership transfer process involves the current owner calling OLAS.changeOwner(). This function validates the new owner is not the zero address and proceeds to write the new owner’s address into the owner’s state variable. If the nominated EOA account is invalid, the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the `owner` check.

A similar scenario happens to the `OLAS.changeMinter` function for the `minter` role.

## Proof of Concept
[OLAS.changeOwner()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L43)
[OLAS.changeMinter()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L58)
[BridgedERC20.changeOwner()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/BridgedERC20.sol#L30)
[FxGovernorTunnel.changeRootGovernor()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/FxGovernorTunnel.sol#L81)
[HomeMediator.changeForeignGovernor()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/bridges/HomeMediator.sol#L81)

## Recommended Mitigation Steps
Consider implementing a two-step process where the owner nominates an owner or minter and the nominated account needs to call an acceptOwnership() function or acceptMinter() function for the transfer of ownership or minter to fully succeed. This ensures the nominated EOA account is valid and active.