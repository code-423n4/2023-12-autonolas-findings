# QA Report

##

## [L-1] A year is not always 365 days

On leap years, the number of days is 366, so calculations during those years will return the wrong value

```solidity
FILE: 2023-12-autonolas/governance/contracts/OLAS.sol

21:  // One year interval
22:    uint256 public constant oneYear = 1 days * 365;

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/OLAS.sol#L21-L22

```solidity
FILE: 2023-12-autonolas/tokenomics/contracts/TokenomicsConstants.sol

// One year in seconds
    uint256 public constant ONE_YEAR = 1 days * 365;

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/TokenomicsConstants.sol#L15-L16

##

## [L-2] Implications of Miner-Controlled Timestamps on Bond Maturity Calculations

Contract uses block.timestamp to determine when bonds mature.

Minor Manipulation Possible: Miners have some leeway to manipulate the block timestamp. They can't deviate wildly from the real time, but they can adjust the timestamp by small amounts (usually on the order of seconds to a few minutes).

Low Incentive for Abuse: In most cases, miners have little incentive to manipulate the timestamp in a way that would significantly impact your contract. The potential gain from slightly adjusting bond maturity times is generally low.

```solidity
FILE: 2023-12-autonolas/tokenomics/contracts/Depository.sol

215: uint256 maturity = block.timestamp + vesting;

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Depository.sol#L215

##

## [L-3] Risks with Arbitrary Payload Execution in Multisig State Modification within a Gnosis Safe Interaction

Since the contract executes the payload without internal validation, any vulnerabilities or flaws in the payload could be exploited, potentially compromising the multisig wallet.

If the payload is malformed or contains errors, the execution might fail, or worse, it could execute unintended modifications (like removing an existing owner instead of adding a new one).

### POC

- Suppose there are three owners (A, B, and C) of a Gnosis Safe multisig wallet, with a threshold of 2 (meaning any transaction requires approval from at least two of the three owners).
- The team decides to add a new owner D and increase the threshold to 3.
- They generate a payload that, when executed, would add D as an owner and update the threshold.
- They pass this payload to the GnosisSafeSameAddressMultisig contract, which then forwards it to the Gnosis Safe contract.

```solidity
FILE: 2023-12-autonolas/registries/contracts/multisigs
/GnosisSafeSameAddressMultisig.sol

 function create(
        address[] memory owners,
        uint256 threshold,
        bytes memory data
    ) external returns (address multisig)
    {
        // Check for the correct data length
        uint256 dataLength = data.length;
        if (dataLength < DEFAULT_DATA_LENGTH) {
            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);
        }

        // Read the proxy multisig address (20 bytes) and the multisig-related data
        assembly {
            multisig := mload(add(data, DEFAULT_DATA_LENGTH))
        }

        // Check that the multisig address corresponds to the authorized multisig proxy bytecode hash
        bytes32 multisigProxyHash = keccak256(multisig.code);
        if (proxyHash != multisigProxyHash) {
            revert UnauthorizedMultisig(multisig);
        }

        // If provided, read the payload that is going to change the multisig ownership and threshold
        // The payload is expected to be the `execTransaction()` function call with all its arguments and signature(s)
        if (dataLength > DEFAULT_DATA_LENGTH) {
            uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
            bytes memory payload = new bytes(payloadLength);
            for (uint256 i = 0; i < payloadLength; ++i) {
                payload[i] = data[i + DEFAULT_DATA_LENGTH];
            }

            // Call the multisig with the provided payload
            (bool success, ) = multisig.call(payload);
            if (!success) {
                revert MultisigExecFailed(multisig);
            }
        }

        // Get the provided proxy multisig owners and threshold
        address[] memory checkOwners = IGnosisSafe(multisig).getOwners();
        uint256 checkThreshold = IGnosisSafe(multisig).getThreshold();

        // Verify updated multisig proxy for provided owners and threshold
        if (threshold != checkThreshold) {
            revert WrongThreshold(checkThreshold, threshold);
        }
        uint256 numOwners = owners.length;
        if (numOwners != checkOwners.length) {
            revert WrongNumOwners(checkOwners.length, numOwners);
        }
        // The owners' addresses in the multisig itself are stored in reverse order compared to how they were added:
        // https://etherscan.io/address/0xd9db270c1b5e3bd161e8c8503c55ceabee709552#code#F6#L56
        // Thus, the check must be carried out accordingly.
        for (uint256 i = 0; i < numOwners; ++i) {
            if (owners[i] != checkOwners[numOwners - i - 1]) {
                revert WrongOwner(owners[i]);
            }
        }
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L85-L144

### Recommended Mitigation

- Implement a validation mechanism within the GnosisSafeSameAddressMultisig contract to verify the payload's structure and intent before execution. This could include checks to ensure the payload aligns with expected changes.

Dry-Run Simulation: Before executing the payload on the actual Gnosis Safe contract, run a simulation or a "dry-run" of the payload in a test environment or a separate function to predict its effects.

##

## [L-4]  Risks of Gnosis Safe's Reverse Order Owner Storage on the Verification Logic of GnosisSafeSameAddressMultisig Contract

- The GnosisSafeSameAddressMultisig contract's correctness in verifying owners depends entirely on this specific storage order of Gnosis Safe.

- If Gnosis Safe ever changes this detail (e.g., owners are stored in the order they were added), the verification logic in GnosisSafeSameAddressMultisig would fail to correctly verify the owners.

-  This reliance on an external contract's specific implementation detail, albeit stable for now, introduces a point of fragility in the GnosisSafeSameAddressMultisig contract.

### Example Scenario

``Initial Owners``: Imagine a Gnosis Safe with two owners, Alice and Bob.
``Adding a New Owner``: A new owner, Charlie, is added, making the owner list Alice, Bob, Charlie.
``Gnosis Safe Storage``: Internally, Gnosis Safe stores them as Charlie, Bob, Alice.
``Verification in GnosisSafeSameAddressMultisig``: If GnosisSafeSameAddressMultisig is verifying the owners, it should expect Charlie, Bob, Alice, not Alice, Bob, Charlie.

```solidity
FILE: 2023-12-autonolas/tokenomics/contracts
/Depository.sol

function create(
        address[] memory owners,
        uint256 threshold,
        bytes memory data
    ) external returns (address multisig)
    {
        // Check for the correct data length
        uint256 dataLength = data.length;
        if (dataLength < DEFAULT_DATA_LENGTH) {
            revert IncorrectDataLength(DEFAULT_DATA_LENGTH, data.length);
        }

        // Read the proxy multisig address (20 bytes) and the multisig-related data
        assembly {
            multisig := mload(add(data, DEFAULT_DATA_LENGTH))
        }

        // Check that the multisig address corresponds to the authorized multisig proxy bytecode hash
        bytes32 multisigProxyHash = keccak256(multisig.code);
        if (proxyHash != multisigProxyHash) {
            revert UnauthorizedMultisig(multisig);
        }

        // If provided, read the payload that is going to change the multisig ownership and threshold
        // The payload is expected to be the `execTransaction()` function call with all its arguments and signature(s)
        if (dataLength > DEFAULT_DATA_LENGTH) {
            uint256 payloadLength = dataLength - DEFAULT_DATA_LENGTH;
            bytes memory payload = new bytes(payloadLength);
            for (uint256 i = 0; i < payloadLength; ++i) {
                payload[i] = data[i + DEFAULT_DATA_LENGTH];
            }

            // Call the multisig with the provided payload
            (bool success, ) = multisig.call(payload);
            if (!success) {
                revert MultisigExecFailed(multisig);
            }
        }

        // Get the provided proxy multisig owners and threshold
        address[] memory checkOwners = IGnosisSafe(multisig).getOwners();
        uint256 checkThreshold = IGnosisSafe(multisig).getThreshold();

        // Verify updated multisig proxy for provided owners and threshold
        if (threshold != checkThreshold) {
            revert WrongThreshold(checkThreshold, threshold);
        }
        uint256 numOwners = owners.length;
        if (numOwners != checkOwners.length) {
            revert WrongNumOwners(checkOwners.length, numOwners);
        }
        // The owners' addresses in the multisig itself are stored in reverse order compared to how they were added:
        // https://etherscan.io/address/0xd9db270c1b5e3bd161e8c8503c55ceabee709552#code#F6#L56
        // Thus, the check must be carried out accordingly.
        for (uint256 i = 0; i < numOwners; ++i) {
            if (owners[i] != checkOwners[numOwners - i - 1]) {
                revert WrongOwner(owners[i]);
            }
        }
    }

```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/registries/contracts/multisigs/GnosisSafeSameAddressMultisig.sol#L85-L144

### Recommended Mitigation
Implement a mechanism to verify the version or specific implementation of the Gnosis Safe contract being interacted with, ensuring it matches the expected logic.

##

## [L-5] GnosisSafeMultisig contract, there is no implementation of event logging for significant operations, particularly for the creation of a new multisig instance

The absence of events does not directly impact the contract’s core functionality. The contract operations (like creating a multisig) will execute as intended, regardless of whether events are emitted.

The primary impact is on transparency and traceability. For users or developers trying to track the contract's activities, the absence of events makes it more difficult to see the history and results of transactions, especially in a decentralize.

```solidity
FILE: 2023-12-autonolas/registries/contracts/multisigs
/GnosisSafeMultisig.sol

/// @dev Creates a gnosis safe multisig.
    /// @param owners Set of multisig owners.
    /// @param threshold Number of required confirmations for a multisig transaction.
    /// @param data Packed data related to the creation of a chosen multisig.
    /// @return multisig Address of a created multisig.
    function create(
        address[] memory owners,
        uint256 threshold,
        bytes memory data
    ) external returns (address multisig)
    {
        // Parse the data into gnosis-specific set of variables
        (address to, address fallbackHandler, address paymentToken, address payable paymentReceiver, uint256 payment,
            uint256 nonce, bytes memory payload) = _parseData(data);

        // Encode the gnosis setup function parameters
        bytes memory safeParams = abi.encodeWithSelector(GNOSIS_SAFE_SETUP_SELECTOR, owners, threshold,
            to, payload, fallbackHandler, paymentToken, payment, paymentReceiver);

        // Create a gnosis safe multisig via the proxy factory
        multisig = IGnosisSafeProxyFactory(gnosisSafeProxyFactory).createProxyWithNonce(gnosisSafe, safeParams, nonce);
    }

```

### Recommended Mitigation

Add event 

```solidity

event MultisigCreated(address indexed multisig, address[] owners, uint256 threshold);

```

##

## [L-6] Potential precision lose in checkpoint() function

Function: checkpoint()

```solidity
FILE: 2023-12-autonolas/tokenomics/contracts
/Tokenomics.sol

 // Bonding and top-ups in OLAS are recalculated based on the inflation schedule per epoch
        // Actual maxBond of the epoch
        tp.epochPoint.totalTopUpsOLAS = uint96(inflationPerEpoch);
        incentives[4] = (inflationPerEpoch * tp.epochPoint.maxBondFraction) / 100;


```
https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/tokenomics/contracts/Tokenomics.sol#L951

### Explanation:
The calculation of inflationPerEpoch multiplies curInflationPerSecond (a uint96 value) by the difference in seconds (diffNumSeconds), which is likely a large number.
The result is then used in a percentage-based calculation for incentives[4].
The division by 100 can lead to precision loss since Solidity does not support floating-point arithmetic.










