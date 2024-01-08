## Using existing public libraries rather than inventing the wheel
The project uses an ownable reentrancy guard and proxy pattern.
For each one of those the project have written them from scratch.
It might be better to just use existing libraries rather then reinventing the wheel.

## Using modifiers
For the ownable and and reentrancy guard the project didn't use modifiers but wrote the code in the body of the function.
I think modifiers would make the code more readable and easier to maintain.

## Code reusability
For the Ownable and Reentrancy-Guard it seems that the code is written again for each contract, for the sake of simplicity and cleaner code it'd be better to write it once in an abstract contract and then inherent them.

## Placing the proxy upgrade function in the implementation contract
Tokenomics is being used with a proxy, the function to upgrade the proxy is placed on the implementation contract.
This seems a bit of an odd design, that would require to copy this function to every future upgrade as well.
And if it's copied incorrectly then the contract can't be upgraded anymore.
It seems much more simple and secure to place it on the proxy's side (it'd also save some bytescode on the implementation side in case it's ever needed).


### Time spent:
28 hours