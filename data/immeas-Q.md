# QA report

## Summary

| id | title |
| --- | --- |
| [L-01](#l-01-missing-to-call-checkpoint-for-a-year-will-lock-the-rewards-in-tokenomics) | Missing to call `checkpoint` for a year will lock the rewards in `Tokenomics` |
| [L-02](#l-02-idf-can-be-manipulated-and-kept-at-epsilonrate) | `IDF` can be manipulated and kept at `epsilonRate` |
| [L-03](#l-03-no-slippagedeadline-for-depositorydeposit) | no slippage/deadline for `Depository::deposit` |
| [L-04](#l-04-implementation-contract-can-be-initialized) | Implementation contract can be initialized |
| [NC-01](#nc-01-expression-can-be-simplified) | Expression can be simplified |

## Low

### L-01 Missing to call `checkpoint` for a year will lock the rewards in `Tokenomics`

Donations to service owners are unlocked when an epoch ends. Epochs ends by anyone calling `checkpoint` on the `Tokenomics` contract:

[`Tokenomics::checkpoint`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L900-L904):
```solidity
File: tokenomics/contracts/Tokenomics.sol

900:        // Check if the time passed since the last epoch end time is bigger than the specified epoch length,
901:        // but not bigger than a year in seconds
902:        if (diffNumSeconds < curEpochLen || diffNumSeconds > ONE_YEAR) {
903:            return false;
904:        }
```

`diffNumSeconds` is set as `block.timestamp - prevEpochTime` above. I.e. the time since the epoch started (and previous ended).

Not calling `checkpoint` within a year will permanently freeze the `Tokenomics` contract. Since `epochLen` can be as large as `ONE_YEAR` this can potentially leave no room to call `checkpoint` as the call would have to be lucky and have a block where block.timestamp = `prevEpochTime + ONE_YEAR` which is unlikely.

#### Recommendation
Consider dropping the `diffNumSeconds > ONE_YEAR` requirement as that can only lock the contract.

### L-02 `IDF` can be manipulated and kept at `epsilonRate`

When `checkpoint` is called it calculates a new `IDF`:

[`Tokenomics::_calculateIDF`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L848-L850):
```solidity
File: tokenomics/contracts/Tokenomics.sol

848:        UD60x18 fpNumNewOwners = convert(numNewOwners);
849:        fp = fp.add(fpNumNewOwners);
850:        fp = fp.mul(codeUnits);
```

Here we can see that it scales with `numNewOwners`. `numNewOwners` is as it says new owners. It is registered when a donation is made:

[`Tokenomics::_trackServiceDonations`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L760-L770):
```solidity
File: tokenomics/contracts/Tokenomics.sol

760:                    if (!mapNewUnits[unitType][serviceUnitIds[j]]) {
761:                        mapNewUnits[unitType][serviceUnitIds[j]] = true;
762:                        mapEpochTokenomics[curEpoch].unitPoints[unitType].numNewUnits++;
763:                        // Check if the owner has introduced component / agent for the first time
764:                        // This is done together with the new unit check, otherwise it could be just a new unit owner
765:                        address unitOwner = IToken(registries[unitType]).ownerOf(serviceUnitIds[j]);
766:                        if (!mapNewOwners[unitOwner]) {
767:                            mapNewOwners[unitOwner] = true;
768:                            mapEpochTokenomics[curEpoch].epochPoint.numNewOwners++;
769:                        }
770:                    }
```

Here it checks that the service has not been seen before and the owner has not been seen before. Creating a new account and having it create a new service is easy though. Thus anyone having bonds in `Depository` can simply create services with new accounts. Then make the smallest possible donation to these services and thus increase `numNewOwners`. As they are the owners of the accounts they'll get their donation back later as well.

#### Recommendation
Consider not using `IDF` at all, or accept that it can possibly always be the maximum `epsilonRate`.

### L-03 no slippage/deadline for `Depository::deposit`

[`Depository::deposit`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L320):
```solidity
File:: tokenomics/contracts/Depository.sol

320:        payout = IGenericBondCalculator(bondCalculator).calculatePayoutOLAS(tokenAmount, product.priceLP);
```

The amount of token is decided by the `priceLP` which is managed by the owner. Since this price needs to be kept relatively up to date with current market it is reasonable a user might send a transaction at one price and later when the transaction is actually executed the price has changed.

#### Recommendation
Consider adding a `deadline` and/or a `minPayout` for the `deposit` call.

### L-04 Implementation contract can be initialized

[`Tokenomics::constructor`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L231-L234)
```solidity
File: tokenomics/contracts/Tokenomics.sol

231:    /// @dev Tokenomics constructor.
232:    constructor()
233:        TokenomicsConstants()
234:    {}
```

`Tokenomics` is an implementation contract intended to be called through a proxy. It is best practice to prevent the implementation contract from being able to be initialized as it's not intended to be used.

#### Recommendation
Consider implementing something similar to OZ `_disableInitializer` in the implementation contract. Perhaps by setting `owner` to `msg.sender` in the implementation contract.

## Non critical / Informational

### NC-01 Expression can be simplified

[`Tokenomics::_trackServiceDonations`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L709-L710):
```solidity
File: tokenomics/contracts/Tokenomics.sol

709:                topUpEligible = (IVotingEscrow(ve).getVotes(serviceOwner) >= veOLASThreshold  ||
710:                    IVotingEscrow(ve).getVotes(donator) >= veOLASThreshold) ? true : false;
```

As the expression already resolves to `true`/`false` the extra `? true : false` isn't needed.