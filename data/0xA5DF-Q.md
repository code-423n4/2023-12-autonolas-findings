| ID  | Title                                                                                         | Severity |
|-----|-----------------------------------------------------------------------------------------------|----------|
| L1  | Guard and owners can't be changed in CM                                                       | Low      |
| L2  | CM can replace the guard while paused                                                         | Low      |
| L3  | Users might burn their tokens by mistake                                                      | Low      |
| L4  | Users can exhaust/spam the system by starting the governorCheckProposalId                     | Low      |
| L5  | CM guard shouldn’t be allowed to call other contracts by default                              | Low      |
| L6  | There’s no check that the sum of the top up fractions isn’t less than 100                     | Low      |
| L7  | treasury doesn't allow to withdraw tokens that were sent directly to the contract             | Low      |
| L8  | `getUserPoint` can revert when index is out of bound error                                    | Low      |
| L9  | ultiple tokens might share the same `tokenURI()`                                              | Low      |
| L10 | `Treasury.receive()` function would revert on simple ETH transfer                             | Low      |
| L11 | `getCurrentPriceLP()` underestimates the price of the LP                                      | Low      |
| L12 | OLAS price might change after product creation                                                | Low      |
| R1  | `getCurrentPriceLP()` should revert in case that none of the tokens are olas                  | Refactor |
| R2  | `ComponentRegistry._getSubComponents()` doesn't assert that the UnitType is component         | Refactor |
| R3  | ComponentRegistry doesn’t implement `calculateSubComponents`                                  | Refactor |
| NC1 | `GnosisSafeSameAddressMultisig.create()` function name doesn’t reflect what the function does | NC       |

## L1 - Guard and owners can't be changed in CM
Gnosis safe guard/owners change are done by an external call made by the Safe to itself.
The GuardCM [prevents any calls to self](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L427-L430). This makes sense as it prevents the CM owners from changing the guard on their own, but there's no alternative way to change the guard or owners when needed.

As mitigation - consider adding mechanism for the governance to approve self calls when needed.

## L2 - CM can replace the guard while paused
While the GuardCM is paused the CM can run any function, including the call to self to replace the guard.
That means the governance would lose the ability to unpause the guard later on.

## L3 - Users might burn their tokens by mistake
The OLAS token allows anybody to burn their tokens, this function might lead to users burning their own tokens by mistake.

[OLAS.sol#L117-L120](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L117-L120)
```solidity
    function burn(uint256 amount) external {
        _burn(msg.sender, amount);
    }
```

## L4 - Users can exhaust/spam the system by starting the governorCheckProposalId
The `governorCheckProposalId` is a proposal ID that'll be used to check if the governance is still 'alive'. If the proposal doesn't pass then 

* In order to propose you need proposal threshold (5 ETH in testing)
* In order for a vote to pass you need at least `quorum` for votes 
* That means that an attacker can continuously spam the system with proposing this vote, while each time the propose is raised the protocol would have to pass the vote AND pass another proposal to replace the `governorCheckProposalId`


## L5 - CM guard shouldn’t be allowed to call other contracts by default
The guard checks the calls only if they are to the timelock or CM addresses, calls to other addresses are allowed and aren't checked.
While I can't point to a practical security risk that seems to stem from that it's best to reject calls to other addresses by default (If there's any other address that the CM needs to call then that should be added to the list of allowed addresses, but the rest should be prohibited).

## L6 - There’s no check that the sum of the top up fractions isn’t less than 100
When incentive fractions are changed there's a check that the fractions aren't greater than 100. But there's no check to ensure they're not less than 100.
In case that their sum is less than 100 then the tokenomics contract wouldn't be using the full potential of top ups.

[Tokenomics.sol#L580-L583](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Tokenomics.sol#L580-L583)

```solidity
        // Same check for top-up fractions
        if (_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction > 100) {
            revert WrongAmount(_maxBondFraction + _topUpComponentFraction + _topUpAgentFraction, 100);
        }
```


## L7 - treasury doesn't allow to withdraw tokens that were sent directly to the contract
The Treasury allows the owner to withdraw the tokens 

[Treasury.sol#L366-L369](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L366-L369)

```solidity
            }  else {
                // Insufficient amount of LP tokens
                revert LowerThan(tokenAmount, reserves);
            }
```

## L8 - `getUserPoint` can revert when index is out of bound error
It seems that `wveOLAS.getUserPoint()` tries to wrap the call to `veOLAS` and prevent out of bond index error.
However, it fails to do so. It checks that `userNumPoints > 0` instead of checking that `idx < userNumPoints`.
In case that `idx` is greater than `userNumPoints` the check might pass and the call would revert with an index out of bound error.

[wveOLAS.sol#L188-L199](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/wveOLAS.sol#L188-L199)
```solidity
    /// @dev Gets the checkpoint structure at number `idx` for `account`.
    /// @notice The out of bound condition is treated by the default code generation check.
    /// @param account User wallet address.
    /// @param idx User point number.
    /// @return uPoint The requested user point.
    function getUserPoint(address account, uint256 idx) public view returns (PointVoting memory uPoint) {
        // Get the number of user points
        uint256 userNumPoints = IVEOLAS(ve).getNumUserPoints(account);
        if (userNumPoints > 0) {
            uPoint = IVEOLAS(ve).getUserPoint(account, idx);
        }
    }
```

## L9 -Multiple tokens might share the same `tokenURI()`
The `tokenURI()` is based on the `unitHash` which isn't unique for the tokens, meaning the `tokenURI()` isn't unique either.
This goes against the [ERC721 standard](https://eips.ethereum.org/EIPS/eip-721#:~:text=///%20%40notice%20A%20distinct%20Uniform%20Resource%20Identifier%20(URI)%20for%20a%20given%20asset.) which requires it to be distinct for each asset:
>  A distinct Uniform Resource Identifier (URI) for a given asset.

Mitigation - keep track of the hashes, and don’t allow the same hash to be created twice.
Alternately, find a way to add the unitId to the tokenURI

[GenericRegistry.sol#L131-L141](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/GenericRegistry.sol#L131-L141)




## L10 - `Treasury.receive()` function would revert on simple ETH transfer
When ETH is sent to a contract without any additional gas (e.g. via Solidity's `trasnfer()` or `send()`) then the gas limit for the `receive()` function is 2300 gas.
The `Treasury.receive()` function uses more than that (it `sload`s 2 storage variable, and then `sstore`s 1), meaning the function would revert in those cases.

(users can still send ETH using `call()` with sufficient gas, in that case it won't revert)

## L11 - `getCurrentPriceLP()` underestimates the price of the LP
`getCurrentPriceLP()` calculates the price based only on the OLAS part of the LP token, it doesn't take into account the value of the other token used in the pool.

This results in a price estimation that's low by 50% than the actual price. If the results of this function are fed into the `Depository` when creating a product then it'd result in a price that's too low and LP holders won't deposit.

[GenericBondCalculator.sol#L70-L92](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L70-L92)


## L12 - OLAS price might change after product creation
The LP price of a product is set during its creation and isn't changed afterwards.
In reality the price of the LP can change (due to OLAS price change) which will end up in buying LP in a higher price than the actual price (loss for the protocol) or the price being too low for the LPs to sell their LP tokens.

## R1 - `getCurrentPriceLP()` should revert in case that none of the tokens are olas

`getCurrentPriceLP()` states that there should never be a case where none of the tokens are olas.
The function checks that, but it doesn't revert if the check fails. Instead it does nothing (which will return zero as a value).
It makes more sense to revert in case of an error rather than return zero.

[GenericBondCalculator.sol#L81-L83](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/GenericBondCalculator.sol#L81-L83)

```solidity
            // token0 != olas && token1 != olas, this should never happen
            if (token0 == olas || token1 == olas) {
                // If OLAS is in token0, assign its reserve to reserve1, otherwise the reserve1 is already correct
```


## R2 - `ComponentRegistry._getSubComponents()` doesn't assert that the UnitType is component
The function ignores `UnitType` and seems to assume that it'll always be a component (since components can't hold subcomponent of other types).
While this is expected to be the case, it might be worth to assert that.

[ComponentRegistry.sol#L37-L45](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol#L37-L45)
```solidity
    /// @dev Gets subcomponents of a provided component Id.
    /// @notice For components this means getting the linearized map of components from the local map of subcomponents.
    /// @param componentId Component Id.
    /// @return subComponentIds Set of subcomponents.
    function _getSubComponents(UnitType, uint32 componentId) internal view virtual override
        returns (uint32[] memory subComponentIds)
    {
        subComponentIds = mapSubComponents[uint256(componentId)];
    }
```

## R3 - ComponentRegistry doesn’t implement `calculateSubComponents`
`calculateSubComponents` is part of the [IRegistry interface](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/interfaces/IRegistry.sol#L38) but isn’t implemented in [ComponentRegistry](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/registries/contracts/ComponentRegistry.sol) (there's an internal function with a similar name, but not a public/external) 

## NC1 - `GnosisSafeSameAddressMultisig.create()` function name doesn’t reflect what the function does
The function doesn’t seem to create any new instance, but just to verify an existing instance. 

