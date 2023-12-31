## [GAS-1] UNNECESSARY (SSTORE) STORAGE VARIABLE UPDATE.
There is 1 instance of this

- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L291

Since the `_locked` storage variable is used for reentrancy guard, there is no need setting the value of `_locked` in the initialize function of the Tokenomics function. This uses the SSTORE opcode which costs more gas.

```
291: function initializeTokenomics(
292:         address _olas,
293:        address _treasury,
294:        address _depository,
295:        address _dispenser,
296:        address _ve,
297:        uint256 _epochLen,
298:        address _componentRegistry,
299:        address _agentRegistry,
300:        address _serviceRegistry,
301:        address _donatorBlacklist
302:     ) external
303:    {
...
264: _locked = 1;//@audit unnecessary state update since there is no reentrance risk in initialze function.
...
370: }

```
#### Recommendation: remove `_locked` state update above.