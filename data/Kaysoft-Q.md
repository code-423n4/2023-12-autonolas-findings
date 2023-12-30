## Security vulnerability with `GovernorCompatibilityBravo` version 4.8.3

There is 1 instance of this
- https://github.com/code-423n4/2023-12-autonolas/blob/2a095eb1f8359be349d23af67089795fb0be4ed1/governance/contracts/GovernorOLAS.sol#L15

According to Openzeppelin's security disclosure here: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-5h3x-9wvq-w4m2

There is a security vulnerability with the `GovernorCompatibilityBravo` contract particularly the `propose(...)` function which is used in the `GovernorOLAS` contract due to the order of the inherited contracts.

```
contract GovernorOLAS is Governor, GovernorSettings, GovernorCompatibilityBravo, GovernorVotes, GovernorVotesQuorumFraction, GovernorTimelockControl {
```
Due to the order of the parent contracts and how Solidity inheritance work, the `super` keyword in the `propose(...)` function below invokes the vulnerable  `GovernorCompatibilityBravo.propose(...)` function.

```
File: GovernorOLAS.sol
45: function propose(
46:        address[] memory targets,
47:        uint256[] memory values,
48:        bytes[] memory calldatas,
49:        string memory description
50:    ) public override(IGovernor, Governor, GovernorCompatibilityBravo) returns (uint256)
51:    {
52:        return super.propose(targets, values, calldatas, description);
53:    }
```
#### References 
- https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-5h3x-9wvq-w4m2

#### Recommendation: Consider updating the openzeppelin library to version 4.9.3.


