# Quality Assurance Report

| QA Issues                                                                            | Issues                                                                     | Instances |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------- | --------- |
| [[L-01](#l-01-address0-check-missed-in-constructor)]                                 | `address(0)` check missed in `constructor`                                 | 1         |
| [[L-02](#l-02-check-value-for-zero-before-dividing)]                                 | Check value for `zero` before dividing                                     | 2         |
| [[N-01](#n-01-public-function-should-be-make-external-if-it-is-not-used-internally)] | `Public` Function should be make `external` if it is not used `internally` | 1         |
| [[N-02](#n-02-use-delete-instead-of-assigning-zero)]                                 | Use `delete` instead of assigning `zero`                                   | 3         |
| [[N-03](#n-03-unnecessary-type-casting)]                                             | Unnecessary Type casting                                                   | 1         |

# Low-Severity

## [L-01] `address(0)` check missed in `constructor`

In a smart contract, an address might be empty or "zero" at the start. If you don't check whether an address is empty before using it, the contract might get confused or make mistakes.

**Note: Missed by bot-report**

_1 instance_

```solidity
File : governance/contracts/veOLAS.sol

132:    constructor(address _token, string memory _name, string memory _symbol)
133:     {
134:         token = _token; //@audit Check _token for address(0)
135:         name = _name;
136:         symbol = _symbol;
137:         // Create initial point such that default timestamp and block number are not zero
138:         // See cast specification in the PointVoting structure
139:         mapSupplyPoints[0] = PointVoting(0, 0, uint64(block.timestamp), uint64(block.number), 0);
140:     }

```

[132-140](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L132C5-L140C6)

## [L-02] Check value for `zero` before dividing

Checking for zero before performing a division operation is crucial to prevent division by zero errors. If you attempt to divide a number by zero in most programming languages, it will result in an error or an exception. In Solidity and smart contract development, encountering such errors can have severe consequences, potentially leading to a halt in the execution of the smart contract or unexpected behavior.

**Note: Missed by bot-report**

_2 instances_

```solidity
File : tokenomics/contracts/Tokenomics.sol

         //@audit Check `sumUnitIncentives` for 0
675:     totalIncentives = mapUnitIncentives[unitType][unitId].topUp + totalIncentives / sumUnitIncentives;

         //@audit Check `sumUnitIncentives` for 0
1224:    topUp += totalIncentives / sumUnitIncentives;

```

[675](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L675C13-L675C111), [1224](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L1224C21-L1224C66)

# Non-Critical

## [N-01] `Public` Function should be make `external` if it is not used `internally`

**Note: Missed by bot-report**

_1 instance_

```solidity
File : governance/contracts/veOLAS.sol

76:    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

```

[76](https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/veOLAS.sol#L761C5-L761C97)

## [N-02] Use `delete` instead of assigning `zero`

**Note: Missed by bot-report**

_3 instances_

```solidity
File : lockbox-solana/solidity/liquidity_lockbox.sol

345:    uint64 liquiditySum = 0;
346:    uint32 numPositions = 0;
347:    uint64 amountLeft = 0;

```

[345-347](https://github.com/code-423n4/2023-12-autonolas/blob/main/lockbox-solana/solidity/liquidity_lockbox.sol#L345C8-L347C31)

## [N-03] Unnecessary Type casting

**Note: Missed by bot-report**

_1 instance_

Here, `unitIds` is already type of `uint32`

```solidity
File : registries/contracts/UnitRegistry.sol

203:    uint32 numUnits = uint32(unitIds.length);

```

[203](https://github.com/code-423n4/2023-12-autonolas/blob/main/registries/contracts/UnitRegistry.sol#L203C8-L203C50)
