# [L-1] Inaccurate and fluctuating LP token price calculation in UniswapV2

The current method for calculating the price of LP (Liquidity Provider) tokens in UniswapV2 utilizes the formula `priceLP = (reserve1 * 1e18) / totalSupply`. 

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

https://github.com/code-423n4/2023-12-autonolas/blob/a7d2d6c575307356a478b7afc3778a00ef10f1db/tokenomics/contracts/GenericBondCalculator.sol#L67-L92

The variables `reserve1` and `totalSupply` are prone to manipulation through swap and deposit actions. This vulnerability was notably exploited in the Warp Finance hack, underlining the risks associated with this calculation method.

The LP price should be calculated using fair price when it takes into account the constant K and provides resistance to market manipulation and providing a more accurate representation of the LP tokens' value.

Details calculation can be seen here: https://blog.alphaventuredao.io/fair-lp-token-pricing/


## Recommended Mitigation Steps

It is recommended to adopt a fair price calculation method for LP tokens. 

# [L-2] Vulnerability in setFxChildTunnel function allowing front-running

The contract `FxERC20RootTunnel` inherents from `FxBaseRootTunnel`. At initiate state of `FxBaseRootTunnel` contract, the state variable `fxChildTunnel` is not set.

        contract FxERC20RootTunnel is FxBaseRootTunnel { 

https://github.com/code-423n4/2023-12-autonolas/blob/667052268700dffe5f813ad65a974dc366dc6beb/governance/contracts/bridges/FxERC20RootTunnel.sol#L24

We can see that in the constructor of `FxERC20RootTunnel`, the `fxChildTunnel` is also not set.

        constructor(address _checkpointManager, address _fxRoot, address _childToken, address _rootToken)
            FxBaseRootTunnel(_checkpointManager, _fxRoot)
        {
            // Check for zero addresses
            if (_checkpointManager == address(0) || _fxRoot == address(0) || _childToken == address(0) ||
                _rootToken == address(0)) {
                revert ZeroAddress();
            }

            childToken = _childToken;
            rootToken = _rootToken;
        }

https://github.com/code-423n4/2023-12-autonolas/blob/667052268700dffe5f813ad65a974dc366dc6beb/governance/contracts/bridges/FxERC20RootTunnel.sol#L38-L49

The `setFxChildTunnel` function in `FxBaseRootTunnel` is intended for setting this variable. However, this function lacks proper access control, allowing any user to set the `fxChildTunnel`:

        function setFxChildTunnel(address _fxChildTunnel) public virtual {
            require(fxChildTunnel == address(0x0), "FxBaseRootTunnel: CHILD_TUNNEL_ALREADY_SET");
            fxChildTunnel = _fxChildTunnel;
        }

https://github.com/code-423n4/2023-12-autonolas/blob/667052268700dffe5f813ad65a974dc366dc6beb/governance/lib/fx-portal/contracts/tunnel/FxBaseRootTunnel.sol#L54-L57

A malicious actor can exploit this by front-running legitimate transactions to set the fxChildTunnel to a fraudulent address. This can lead to severe consequences:

- Rerouting of Messages: The attacker can divert messages from the root tunnel to the malicious fxChildTunnel, instead of the intended one.

- Loss of Funds: Users' tokens on Layer 1 (L1) could be burnt without the corresponding LP tokens being correctly transferred back, resulting in financial losses.

- Denial of Service (DoS): If the protocol detects the incorrect setting early, correcting it is impossible without redeploying the tunnel and BridgedERC20, as setFxChildTunnel can only be called once.

The same issue is observed in FxERC20ChildTunnel.

https://github.com/code-423n4/2023-12-autonolas/blob/667052268700dffe5f813ad65a974dc366dc6beb/governance/lib/fx-portal/contracts/tunnel/FxBaseChildTunnel.sol#L31-L34

## Recommended Mitigation Steps

Some options to consider:
- the fxChildTunnel can be set during contract deployment through the constructor
- add access control so that only authorized addresses (e.g., contract owners or administrators) can call setFxChildTunnel.

# [NC-1] Wrong natspec

Wrong natspec. The function only allow to calculate total voting power in the future.

        /// @dev Calculate total voting power at some point in the past.
        /// @param lastPoint The point (bias/slope) to start the search from.
        /// @param ts Time to calculate the total voting power at.
        /// @return vSupply Total voting power at that time.
        function _supplyLockedAt(PointVoting memory lastPoint, uint64 ts) internal view returns (uint256 vSupply) {

https://github.com/code-423n4/2023-12-autonolas/blob/667052268700dffe5f813ad65a974dc366dc6beb/governance/contracts/veOLAS.sol#L686-L690

# [NC-2] Enhancement of year constant representation in Solidity code

In the current implementation of the OLAS contract, the constant `oneYear` is defined using the calculation `1 days * 365`

        uint256 public constant oneYear = 1 days * 365;

https://github.com/code-423n4/2023-12-autonolas/blob/667052268700dffe5f813ad65a974dc366dc6beb/governance/contracts/OLAS.sol#L22

While this approach is technically correct, it overlooks the native support for time unit calculations provided by Solidity. Specifically, Solidity allows for a more straightforward and readable expression using `1 years`.

It makes the code easier to understand and aligns with standard Solidity conventions.