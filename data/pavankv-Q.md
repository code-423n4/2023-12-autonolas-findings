## 1. Depository contract can make `effectiveBond` as `ZERO` which breaks the invariant of the protocol.

The function reserveAmountForBondProgram() is called by the depository contract during bond program creation to subtract an amount from the effectiveBond value, which represents bonds remaining from previous epochs. However, while comments within the code explicitly state that the effective bond value must exceed the amount or supply parameter specified in the depository contract, the reserveAmountForBondProgram() function, in practice, permits the amount to equal the effectiveBond value.

```solidity

        // Effective bond must be bigger than the requested amount
        uint256 eBond = effectiveBond;
        if (eBond >= amount) { //QA and Analysis
```

**Recommendation**

Our recommendation is to align the code how comments stated to maintain the protocol's invariant.

```solidity
// Effective bond must be bigger than the requested amount
        uint256 eBond = effectiveBond;
        if (eBond >= amount) { //@audit changed here

```

code snippet:-
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L615C1-L617C31