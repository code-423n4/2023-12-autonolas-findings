Low Risk

1.
Unsafe casting may overflow:
  GenericBondCalculator.sol::613 => xAbs = x < 0 ? uint256(-x) : uint256(x);
  GenericBondCalculator.sol::614 => yAbs = y < 0 ? uint256(-y) : uint256(y);
  GenericBondCalculator.sol::615 => dAbs = denominator < 0 ? uint256(-denominator) : uint256(denominator);
  GenericBondCalculator.sol::638 => result = sx ^ sy ^ sd == 0 ? -int256(resultAbs) : int256(resultAbs);
  GenericBondCalculator.sol::681 => uint256 xAux = uint256(x);
  FxERC20RootTunnel.sol::261 => return address(uint160(toUint(item)));
  FxERC20RootTunnel.sol::461 => uint8 nextPathNibble = uint8(path[pathPtr]);
  FxERC20RootTunnel.sol::522 => uint8 hpNibble = uint8(_getNthNibbleOfBytes(0, b));
  FxERC20RootTunnel.sol::541 => return bytes1(n % 2 == 0 ? uint8(str[n / 2]) / 0x10 : uint8(str[n / 2]) % 0x10);
  GaurdCM.sol::197 => uint256 targetSelectorChainId = uint256(uint160(target));
  GaurdCM.sol::369 => address bridgeMediatorL2 = address(uint160(bridgeMediatorL2ChainId));
  GaurdCM.sol::481 => uint256 targetSelectorChainId = uint256(uint160(targets[i]));
  GaurdCM.sol::530 => uint256 bridgeMediatorL2ChainId = uint256(uint160(bridgeMediatorL2s[i]));
  GaurdCM.sol::589 => uint256 targetSelectorChainId = uint256(uint160(target));
  GaurdCM.sol::608 => bridgeMediatorL2 = address(uint160(bridgeMediatorL2ChainId));

  
2.
Block Timestamp
  bridgeERC20.sol::121 => require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED"); 
  Olas.sol::1130 => uint256 numYears = (block.timestamp - timeLaunch) / oneYear; 

Informational Risk

1.
Unsafe ERC20 Operation(s):
  Treasury.sol::161 => function transfer(address to, uint256 amount) external returns (bool);
  Treasury.sol::173 => function approve(address spender, uint256 amount) external returns (bool);
  Treasury.sol::180 => function transferFrom(address from, address to, uint256 amount) external returns (bool);
  Treasury.sol::484 => bool success = IToken(token).transferFrom(account, address(this), tokenAmount);
  Treasury.sol::611 => success = IToken(token).transfer(to, tokenAmount);
  FxERC20RootTunnel.sol::25 => function approve(address spender, uint256 amount) external returns (bool);
  FxERC20RootTunnel.sol::31 => function transfer(address to, uint256 amount) external returns (bool);
  FxERC20RootTunnel.sol::38 => function transferFrom(address from, address to, uint256 amount) external returns (bool);
  FxERC20RootTunnel.sol::991 => bool success = IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
  
2.
Bytes constants are more efficient than string constants:
  TokenomicsConstants.sol::11 => string public constant VERSION = "1.0.1";
  
3.
Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`):
  GenericBondCalculator.sol::359 => /// if (x >= 2 ** 128) {
  GenericBondCalculator.sol::683 => if (xAux >= 2 ** 128) {
  GenericBondCalculator.sol::687 => if (xAux >= 2 ** 64) {
  GenericBondCalculator.sol::691 => if (xAux >= 2 ** 32) {
  GenericBondCalculator.sol::695 => if (xAux >= 2 ** 16) {
  GenericBondCalculator.sol::699 => if (xAux >= 2 ** 8) {
  GenericBondCalculator.sol::703 => if (xAux >= 2 ** 4) {
  GenericBondCalculator.sol::707 => if (xAux >= 2 ** 2) {
  FxERC20RootTunnel.sol::556 => require(index < 2 ** proofHeight, "Leaf index is too big")
  
3.
Using storage instead of memory for structs/arrays saves gas:
  FxERC20RootTunnel.sol::76 => bytes memory data = abi.encode(msg.sender, _receiver, _data);
  FxERC20RootTunnel.sol::119 => RLPItem memory item = self.item;
  FxERC20RootTunnel.sol::168 => RLPItem[] memory result = new RLPItem[](items);
  FxERC20RootTunnel.sol::233 => bytes memory result = new bytes(item.len);
  FxERC20RootTunnel.sol::303 => bytes memory result = new bytes(len);
  FxERC20RootTunnel.sol::436 => bytes memory path = _getNibbleArray(encodedPath);
  FxERC20RootTunnel.sol::499 => bytes memory partialPath = _getNibbleArray(encodedPartialPath);
  FxERC20RootTunnel.sol::500 => bytes memory slicedPath = new bytes(partialPath.length);
  FxERC20RootTunnel.sol::519 => bytes memory nibbles = "";
  FxERC20RootTunnel.sol::666 => bytes memory typedBytes = receipt.raw;
  FxERC20RootTunnel.sol::667 => bytes memory result = new bytes(typedBytes.length - 1);
  FxERC20RootTunnel.sol::797 => bytes memory branchMaskBytes = payload.getBranchMaskAsBytes();
  FxERC20RootTunnel.sol::886 => bytes memory message = _validateAndExtractMessage(inputData);
  FxERC20RootTunnel.sol::986 => bytes memory message = abi.encode(msg.sender, to, amount);  
  
4.
Multiplication by two should use bit shifting:
  FxERC20RootTunnel.sol::524 => nibbles = new bytes(b.length * 2 - 1);
  FxERC20RootTunnel.sol::529 => nibbles = new bytes(b.length * 2 - 2);  
  
5.
Bytes constants are more efficient than string constants:
  wveOlas.sol::136 => string public constant name = "Voting Escrow OLAS";
  wveOlas.sol::138 => string public constant symbol = "veOLAS";  

### Time spent:
36 hours