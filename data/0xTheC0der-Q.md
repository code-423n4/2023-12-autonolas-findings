# Summary
* L-1: Front-running of `setFxChildTunnel`/`setFxRootTunnel` after deployment
* L-2: Inconsistent `OLAS` supply cap
* L-3: Treasury conservation law can be broken via `selfdestruct`
* L-4: Donator blacklisting is ineffective and can be circumvented
* L-5: Users can be DoS'ed on `Depository.deposit(...)` by front-runner
* L-6: OLAS minting via treasury is not guaranteed
* NC-1: `Treasury.drainServiceSlashedFunds()` can leave dust behind in `ServiceRegistry`
* NC-2: Add implementation of `GuardCM.checkAfterExecution(...)`

## L-1: Front-running of `setFxChildTunnel`/`setFxRootTunnel` after deployment

The [FxERC20ChildTunnel.setFxRootTunnel(...)](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20ChildTunnel.sol) and [FxERC20RootTunnel.setFxChildTunnel(...)](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/bridges/FxERC20RootTunnel.sol) methods [need to be called immediately after deployment](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/docs/deployment.md?plain=1#L34-L35) of these 2 contracts in order to finish their initialization.  

However, those methods' **access is not restricted**:
```solidity
function setFxRootTunnel(address _fxRootTunnel) external virtual {
    require(fxRootTunnel == address(0x0), "FxBaseChildTunnel: ROOT_TUNNEL_ALREADY_SET");
    fxRootTunnel = _fxRootTunnel;
}

function setFxChildTunnel(address _fxChildTunnel) public virtual {
    require(fxChildTunnel == address(0x0), "FxBaseRootTunnel: CHILD_TUNNEL_ALREADY_SET");
    fxChildTunnel = _fxChildTunnel;
}
```
As a consequence, the initialization can be front-run by an attacker after deployment to **redirect** the tunnels.  
However, this attack can only happen after deployment before the protocol is in use and needs to go unnoticed for a while to cause any damage. Otherwise, redeployment is straight-forward in case this happens.

#### Why is it implemented like this?
Because those 2 tunnel contracts are dependent on each other's address. One is on Ethereum, the other on Polygon.

#### How to solve this?
Deploy those contracts via `create2` with a *salt* in order to ensure **deterministic** contract addresses before deployment.  
Therefore, the **pre-computed** root/child tunnel address can already be provided via the constructor which avoids the need for a subsequent initialization call that can be front-run.

## L-2: Inconsistent `OLAS` supply cap

The [TokenomicsConstants.getSupplyCapForYear(...)](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L30-L61) method has **pre-defined progressive supply cap levels** for the first 10 years and 2% inflation afterwards, while the [OLAS.inflationRemainder()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L98-L114) method, which actually enforces the supply cap on minting, has the **same fixed supply cap** for the first 10 years and 2% inflation afterwards.  
Therefore there is a discrepancy between the amount of `OLAS` which is expected to be minted vs. the amount which actually can be minted within the first 10 years.  

The expected supply cap levels for the first 10 years from `TokenomicsConstants.sol`:
```solidity
uint96[10] memory supplyCaps = [
    529_659_000_00e16,
    569_913_084_00e16,
    641_152_219_50e16,
    708_500_141_72e16,
    771_039_876_00e16,
    828_233_282_97e16,
    879_860_040_11e16,
    925_948_139_65e16,
    966_706_331_40e16,
    1_000_000_000e18
];
```

The actual supply cap for the first 10 years from `OLAS.sol`:
```solidity
uint256 public constant tenYearSupplyCap = 1_000_000_000e18;
```


## L-3: Treasury conservation law can be broken via `selfdestruct`
According to [Treasury.sol](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L38):
```solidity
/// invariant {:msg "broken conservation law"} address(this).balance == ETHFromServices + ETHOwned;
```
Although the contract's constructor and methods perfectly respect this invariant, one can simply force-fund the contract with `ETH` using the [selfdestruct](https://www.evm.codes/#ff?fork=shanghai) EVM opcode which consequently breaks the given invariant.  

Recommendation: Add a `sync()` method which adds any surplus balance to the `ETHOwned` state variable while respecting the `type(uint96).max` limit, see also [receive()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L120-L133).

## L-4: Donator blacklisting is ineffective and can be circumvented
A [blacklisted donator](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/DonatorBlacklist.sol#L82-L84) can simply circumvent the blacklisting by using another account for donations. This is already proven within the test case ["Claim incentives for unit owners when donator and service owner are different accounts"](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/test/Dispenser.js#L308-L310) which shows that the donator can be an unrelated account.  

The only point where the actual address of the donator is important is in [Tokenomics._trackServiceDonations(...)](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L682-L774):
```solidity
bool topUpEligible;
if (incentiveFlags[2] || incentiveFlags[3]) {
    address serviceOwner = IToken(serviceRegistry).ownerOf(serviceIds[i]);
    topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
        IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
}
```
Here, the top-up eligibility is determined based on the votes of the service owner **or** donator.  
Therefore, the blacklisting **is only effective if** the service owner does not have enough votes **and** the alternative donator (to circumvent blacklist) does not have enough votes either.

## L-5: Users can be DoS'ed on `Depository.deposit(...)` by front-runner
**Anyone** can front-run a user's [Depository.deposit(uint256 productId, uint256 tokenAmount)](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L279-L346) transaction.  
However, this is especially a problem whem the user desires a **full deposit** where `tokenAmount` is chosen such that the whole `product.supply` is used. In this case, the attacker can front-run the transaction with a miniscule `tokenAmount` in order to cause DoS for the user via a [ProductSupplyLow](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L322-L325) error.

## L-6: `OLAS` minting via treasury is not guaranteed

1. The [OLAS.mint(...)](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L75-L85) method does **not** revert if the requested amount cannot be limited due to the inflation limit:

```solidity
    function mint(address account, uint256 amount) external {
        // Access control
        if (msg.sender != minter) {
            revert ManagerOnly(msg.sender, minter);
        }

        // Check the inflation schedule and mint
        if (inflationControl(amount)) {
            _mint(account, amount);
        }
    }
```

2. The treasury itself correctly accounts for the OLAS inflation limit and thereby guarantees the minting of the the requested `OLAS` amount [here](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L240-L242) and [here](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/tokenomics/contracts/Treasury.sol#L414-L416).
3. However, the `minter` of `OLAS` can be changed via [changeMinter(address newMinter)](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/OLAS.sol#L58-L69) by the `owner`.
4. Then, the new `minter` can mint `OLAS` up to the inflation limit.
5. Next, the minter is changed back to the treasury contract.
6. Now, the minting via treasury (see references in step 2) will fail to mint enough `OLAS` **without** any error.

## NC-1: `Treasury.drainServiceSlashedFunds()` can leave dust behind in `ServiceRegistry`
Since the [drainServiceSlashedFunds()](https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Treasury.sol#L466-L483) method enforces the `minAcceptedETH` limit before draining funds from the `ServiceRegistry`. There can be occasions where small amounts (`< minAcceptedETH`) are left behind in the `ServiceRegistry` without any possibility to withdraw them.  

Recommendation: Do not enforce `minAcceptedETH` when draining slashed funds.

## NC-2: Add implementation of `GuardCM.checkAfterExecution(...)`

The implementation of the [GuardCM.checkAfterExecution(...)](https://github.com/code-423n4/2023-12-autonolas/blob/ddcf4dbd8ad9b2daa87e190280deb4d41ac9d647/governance/contracts/multisigs/GuardCM.sol#L571-L572) method, which is the counterpart to `GuardCM.checkTransaction(...)`, is empty.  
There might be settings within the protocol that should not be changed via a `multisig` call. Thus, it is recommended to check for such invariants after the call within that method and revert if necessary.

