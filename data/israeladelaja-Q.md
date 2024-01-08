## [L-01] `GenericBondCalculator.getCurrentPriceLP()` price can be skewed by selling a large amount of OLAS tokens into the liquidity pool. 
There is 1 instance of this issue:
This function is located in `GenericBondCalculator.sol` and is also called in `Depository.sol` with the same function name. An OLAS holder with a significant amount of tokens or a "whale" could potentially skew the LP price returned by selling a large amount of tokens into the liquidity pool. This is a low severity as it is an `external view` function and is not guaranteed to be used when calculating the LP price for bond products but can also impact other entities that call this function for data (hypothetically).
```solidity
/// @dev Gets current reserves of OLAS / totalSupply of LP tokens.
/// @param token Token address.
/// @return priceLP Resulting reserveX / totalSupply ratio with 18 decimals.
function getCurrentPriceLP(address token) external view returns (uint256 priceLP)
{
    IUniswapV2Pair pair = IUniswapV2Pair(token);
    uint256 totalSupply = pair.totalSupply();
    if (totalSupply > 0) {
        address token0 = pair.token0();
        address token1 = pair.token1();
        uint256 reserve0;
        uint256 reserve1;
        // requires low gas
        (reserve0, reserve1, ) = pair.getReserves();
        // token0 != olas && token1 != olas, this should never happen
        if (token0 == olas || token1 == olas) {
            // If OLAS is in token0, assign its reserve to reserve1, otherwise the reserve1 is already correct
            if (token0 == olas) {
                reserve1 = reserve0;
            }
            // Calculate the LP price based on reserves and totalSupply ratio multiplied by 1e18
            // Inspired by: https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy#L262
            priceLP = (reserve1 * 1e18) / totalSupply;
        }
    }
}
```



## [R-01] `sload(PROXY_TOKENOMICS)` in `TokenomicsProxy.sol` can be `and`ed in assembly with `0xffffffffffffffffffffffffffffffffffffffff` for extra security
There is 1 instance of this issue:
As can be seen from contracts like `GnosisSafeProxy.sol` at https://github.com/safe-global/safe-contracts/blob/v1.3.0/contracts/proxies/GnosisSafeProxy.sol#L29 when loading the implementation address from storage, it is added to the stack as the result of an `and` operation with `0xffffffffffffffffffffffffffffffffffffffff` for added security.

