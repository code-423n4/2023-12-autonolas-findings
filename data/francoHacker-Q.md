###https://github.com/code-423n4/2023-12-autonolas/blob/main/governance/contracts/multisigs/GuardCM.sol###

1.Create an access modifier for the statement:

    if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

and use this modifier in all functions where this 'if' statement is implemented.
         
