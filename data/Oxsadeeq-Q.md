In the Autonolas protocol dao members  are allowed to make proposals,A proposal is represented as (address target,uint256 value,bytes payload).There are curently two encoding for encoding parameters which are 1)abi.encode 2)abi.encodePacked..The  issue is that payloads encoded with abi.encode instead of abi.encodePacked would be decoded wrongly these  issue is due to the decoding technique implemented by the autonolas which doesn't account for padding in the encoded payload.
Contracts affected ::
FxGovernorTunnel.sol
processMessageFromRoot() .
HomeMediator.sol
processMessageFromForeign()
