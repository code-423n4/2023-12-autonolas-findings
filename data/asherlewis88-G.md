https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/interfaces/IVotingEscrow.sol

Consider using factory to reduce potential bugs or malicious sellers that come with escrow. Also, factory uses less gas than constructor as it is only executed once. Suggestion to make it robust:



interface IVotingEscrow {

    function getVotes(address account) external view returns (uint256); {

contract WETHSaleFactory {
    IERC20 public OLAS;
    mapping(address => WETHSale) public sales;

    constructor(IERC20 _weth) public {
        weth = _weth;
    }

    function deploy() external {
        require(sales[msg.sender] == WETHSale(0), "Only one sale per seller.");

        sales[msg.sender] = new WETHSale(weth, msg.sender);
    }
}
