# Tightly pack storage variables
When defining storage variables, make sure to declare them in ascending order, according to size. When multiple variables are able to fit into one 256-bit slot, this will save storage size and gas during runtime. For example, if you have a `bool`, `uint256` and a `bool`, instead of defining the variables in the previously mentioned order, defining the two boolean variables first will pack them both into one storage slot since they only take up one byte of storage.

## 1.  Inside contract **veOLAS** at veOLAS.sol 

line 101, 110 & 115


```
 
    uint64 internal constant WEEK = 1 weeks;
    uint256 internal constant MAXTIME = 4 * 365 * 86400; //-----> uint256
    int128 internal constant IMAXTIME = 4 * 365 * 86400;
    uint8 public constant decimals = 18;

    address public immutable token;
    
    uint256 public supply; //-------> uint256

    mapping(address => LockedBalance) public mapLockedBalances;


    uint256 public totalNumPoints;  //-------> uint256

    mapping(uint256 => PointVoting) public mapSupplyPoints;
  
    mapping(address => PointVoting[]) public mapUserPoints;
   
    mapping(uint64 => int128) public mapSlopeChanges;

    
    string public name;
 
    string public symbol;
```

**This could be re-written as** 
```
 uint8 public constant decimals = 18;     ---> 1st
 uint64 internal constant WEEK = 1 weeks; ---> 2nd 
 string public name;   -----------> 3rd
 string public symbol; -----------> 4th
 int128 internal constant IMAXTIME = 4 * 365 * 86400; ---> 5th
 address public immutable token;   ----------> 6th
``` 


## 2. Inside Contract wveOLAS.sol:130

```
    address public immutable ve;
    
    address public immutable token;
    
    string public constant name = "Voting Escrow OLAS";
    
    string public constant symbol = "veOLAS";
    
    uint8 public constant decimals = 18;
```
This code can be further re-written to save gas by initializing the variables from lowest to highest.

starting with uint8, strings and then the address.


