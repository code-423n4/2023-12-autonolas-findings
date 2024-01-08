**Index of table**
| Sl.no  | Particulars  |
|---|---|
| 1   | Overview  |
| 2  | Architecture view(Diagram)  |
| 3  | Approach taken in evaluating the codebase  |
| 4  | Systematic risks  |
| 5 | Mechanism review  |
| 6  | Recommendation   |
| 7  | Hours spend  |



## 1. OverView

Autonolas is a innovative protocol (marketplace) that converges cutting-edge technologies like decentralized autonomous AI agents, modular components, smart contracts, and DAOs into a unified platform. This isn't merely a technological innovation; it's a paradigm shift in how we envision and build applications.It empowers developers to unleash the potential of services that operate continuously, make autonomous decisions, and interact with the real world independently. It's like setting machines free to work for us, but not in the dystopian sense.Agents can collaborating to manage your finances, optimize supply chains, or drive self-governing DAOs with automated actions and transparent decision-making. The OLAS token holders can participate in the governance phase by locking their tokens and receiving non-transferable veOLAS tokens (virtualized tokens) in return. Their voting power is determined by the duration of their locked tokens and the amount locked.



## 2. Architecture View(Diagram)

![OLAS DIAGRAM](https://user-images.githubusercontent.com/69415766/294666223-0aea3128-56e5-4a2f-b8a8-34116b9cfea7.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDQ1MzY0MjIsIm5iZiI6MTcwNDUzNjEyMiwicGF0aCI6Ii82OTQxNTc2Ni8yOTQ2NjYyMjMtMGFlYTMxMjgtNTZlNS00YTJmLWI4YTgtMzQxMTZiOWNmZWE3LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAxMDYlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMTA2VDEwMTUyMlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTVlM2E0MmQ3N2NhODU2NDZlYTgwZTcyNjM3MGI2MTAxOTA5NTM1NzdhMTRmNTlhYzQ4YTc4ZDk5M2RiNGM5YWQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.1YL0T6m85tVSGSnzlXIAGcnLDCBd_bxp8GEC8BtoXW8)

Due to the free software we represented only core function call and contracts.

## 3. Approach taken in evaluating the codebase

We approached manual code review by looking into each space in the scoped contract and decided to explain some core functional contracts.It is important to note that manual code reviews can be very time-consuming.

**Note :-
We usually do function by function and contract analysis but autonolas have done multiple external and internal audits and pdfs have enough information about that. So here we overview the Core contracts only.**


**Core Contract**

**Governance**

 1. veOLAS.sol
 
 This contract enables OLAS token holders to lock their tokens for a specified duration and amount, granting them entry into the governance phase. This signifies the ability to propose changes and upgrades to the protocol, participate in decisions regarding fund allocation and parameter adjustment, and exercise veto power to cancel potentially risky protocol modifications.


 2. GuardCM.sol

This contract relies on the Gnosis sidechain as a multisig. A group of trusted individuals or a community can propose and reject any decision without requiring governance permission. It acts as a bridge between the Gnosis Safe and Autonolas, enabling guardians to perform specific actions within the platform. It helps to upgrade the protocol, approve and reject changes in the Autonolas platform, and can trigger an emergency shutdown. This contract has functionalities like verifying and processing bridge data, and changing bridge metadata to enhance flexibility between other chains.
 
 
 3. TimeLock 
 
 This contract is backed by the openzepplin time controller contract as we know it helps in delaying the execution of specific operation on protocol by governance and authorised bodies.
  

**Registries**

 1. UnitRegistries.sol
  
 This abstract contract is used for adding new agents and components to the protocol which deployed by component registry contract and agent registry. It has functions such as creating units (agents/components), updating unit hashes, and calculating subcomponents.


 2. RegistriesManager.sol
 
This contract serves as an interface, enabling users to create and update hashes for agents and components within the system and in return they will get ERC-721 NFT with a reference to the IPFS hash of the metadata.

 
**Tokenomics**

 1. Depository.sol
 
 This contract plays a vital role in the Autonolas protocol. It enables an interface where users can create bonds by staking whitelisted LP tokens and get OLAS in return at a slightly discounted rate compared to the market.This contract functionality extends beyond initial bond creation. It empowers users to manage their investments, offering capabilities like bond closure, token deposit for specific bonds, and token redemption upon bond maturity. This flexibility allows users to adjust their positions within the protocol as needed, optimizing their returns and participation within the evolving ecosystem. This contract will enables economic activity within the protocol. This contract have functionality are creation and closing the bond , deposit and redeeming the tokens for specific bonds.
 

 2. Dispenser.sol
 
This contract is used to distribute incentives for the code. It have a function called `claimOwnerIncentives` that enables the claiming of incentives for components or agents by interacting with tokenomics and treasury functions.
 
 
 3. Tokenomics.sol 

This contract plays vital role and have crucial functionalities, including modifying parameters for registries and managers. Its effective bond concept utilizes leftover funds from the previous epoch ("effectiveBond"), automatically deducting them before each new epoch and reincorporating them into product bonding during deposits and redemptions. Additionally, it tracks donated ETH for services and tokens for units (agents/components), and its determination of the "Inverse Discount Factor" dictates OLAS token payouts during bond creation. Finally, this contract serves as a vital mechanism checkpoint, enabling adjustments to incentives, inflation, account rewards, and treasury rebalancing.
 
 
 4. Treasury.sol
 
This contract holds the responsibility of minting OLAS tokens. It also provides an interface where users can make ETH donations for agents and components. Additionally, it plays a role in rebalancing treasury funds when called upon by the Tokenomics contract, updating the `amountETHFromServices` and `amountETHOwned` variables accordingly. Furthermore, it enables the owner to transfer tokens from the treasury to specified addresses. Finally, it possesses a function that facilitates the deposit of LP tokens and the minting of OLAS tokens during bond creation.
 
  
 5. GenericBondCalculator.sol
 
This contract plays a crucial role in determining amount of OLAS token bond creation. It have key functions, including calculating the amount of OLAS tokens to be minted and fetching the current liquidity pool price via the Pairs contract from Uniswap. This integration with Uniswap ensures accurate pricing and smooth bonding execution and utilises the curve finance equation.



## 4. Systematic Risks.

After manual code review of autonolas protocol by looking each space we are concluded by following risks.
Systemic risk is the possibility that a single event could cause an industry or the economy to collapse.

 1. Contracts `Tokenomics` ,`Depository` and `Treasury`  are highly depend on the single owner address as authorisation . If owner address compromised by any factor then bonds , agents and components associated with that will be landed on risk. There is no other method to change the existed owner address by any governance protocol.
 
 2. The [`Create()`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L183) function is used to initiate a bond with LP tokens with specific terms, while the [`deposit()`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L291) function is used to deposit LP tokens for specific bonds and by this functions user enters to economy of the protocol. If protocol bearing risk still this two function allows users to create and deposit the bonds and LP tokens. 
 
 3. Autonolas utilizes the uniswap v2 to fetch and determines the LP token price risk associated are following :-
 
    a. Uniswap V2 relies on time-weighted average prices (TWAPs) for price updates. These TWAPs can lag behind real-time market prices, especially during periods of high volatility which leads to  inaccurate LP token price estimations. 
    
    b. Uniswap v2 associated with the impermanent loss which offically mentioned in the docs. If one asset significantly outperforms the other, LPs can experience impermanent loss, meaning their token value might decrease compared to simply holding the assets individually. Their value fluctuates based on the relative price changes of those assets.
 
 
 4. Determining the amount of OLAS token payout for users may become an issue in the future. This is because each pool and protocol has its own unique equational calculation for minting LP tokens based on user deposits. However, Autonolas protocol factors in the price of LP tokens, the amount of LP tokens, and the IDF factor, potentially leading to users receiving either lower or higher amounts of OLAS tokens in the future. 
 
 Consider this scenario: A user deposits a minimum amount of tokens into a new pool and receives a higher amount of LP tokens in return. They then deposit these LP tokens into Autonolas, potentially receiving OLAS tokens at a discounted rate. However, if the pool faces any risks that cause the price of LP tokens to decline, the user would still receive (LP tokens or vise-versa tokens) based on the historical price within Autonolas by `redeem() or deposit()` functions  .
 

## 5. Mechanism review 

Autonolas is forging a groundbreaking path in DeFi by seamlessly integrating various innovative concepts into its protocol, ushering in a new era of automation and decentralized intelligence.

 a. Decentralized Autonomous Agents (DAAs):

Agent Architecture:- 

DAAs can be built with various architectures, including modular designs with specialized modules for tasks like decision-making, resource management, and interaction with the blockchain.

Consensus Mechanisms:-

DAAs may utilize on-chain or off-chain consensus mechanisms to reach collective decisions. On-chain options include delegated proof-of-stake (DPoS) or Byzantine Fault Tolerance (BFT) protocols, while off-chain methods involve agent-to-agent communication and agreement algorithms.

Incentive Mechanisms:-

Tokenized reward systems or reputation protocols can incentivize DAAs to behave in accordance with the DAO's goals and contribute effectively.


 b. Co-owned AI:-

Model Training and Governance:-

AI models used by DAOs and DAPPs can be collaboratively trained and improved by community members, possibly using federated learning approaches to protect sensitive data. Governance mechanisms like on-chain voting or reputation-based systems can determine model updates and usage. Autonolas unleashes a wave of innovation by empowering developers to create their own custom agents and component services using Python. These intelligent agents seamlessly interact with various dApps across the Autonolas platform, fostering a vibrant ecosystem of decentralized services and economic activity.

Data Access and Security:-

 Secure data access protocols ensure authorized DAAs can access relevant off-chain data while upholding privacy and preventing unauthorized data manipulation.


 c. Inter-DAO and DApp Integration:-

Communication Protocols:- DAAs can communicate with each other and external applications through standardized protocols, enabling collaboration and data exchange across DAOs and decentralized applications (dApps).

Interoperability Standards: Utilizing common data formats and communication standards facilitates seamless integration between different DAOs and dApps built on the Autonolas platform.

 d. Bonding mechanism :-

Bonding and Discount factor :-

Autonolas has adopted a bonding mechanism to increase the diversified liquidity of various LP tokens. This mechanism involves users depositing whitelisted liquidity pool tokens on Autonolas and creating a bonding term. In return, users receive OLAS tokens at a slight discount compared to the market price, offering a potentially higher return. A discount factor is used to control the amount of capital that can be bonded and determines the payout for the bonds.

Locking :-

Users who hold OLAS tokens can enter the governance phase by staking their tokens. In return, they receive non-transferable veOLAS governance tokens, which enable them to propose new changes to the protocol, including parameter changes for rewards distribution, and participate in various decision-making processes.

Governance :-

veOLAS token holders play a pivotal role in shaping the Autonolas protocol. They have the power to propose and cancel changes, ensuring active community participation in governance. In addition, a trusted multi-sig contract exists, enabling the community to propose or cancel ongoing processes without requiring traditional governance permissions.



Comparison table with other protocol.


| Feature  | Autonolas  | MakerDAO  | Morpheus  |
|---|---|---|---|
| Focus  | Decentralized insurance on smart contracts , automation and AI  | Stablecoin issuance and decentralized finance (DeFi) | Decentralized open-source personal AIs, connecting them directly to wallets, dApps, and smart contracts.|
| Protocol Token  | OLAS  | MKR and DAI | MOR |
| Target Market	  | Developers and users of smart contracts   | Anyone seeking to borrow or lend cryptocurrencies  | Developers and users of smart contracts   |
| Insurance Model  | Bond and Locking Model  | Algorithmic and collateralized debt positions (CDPs)  | Block reward distribution and  |
| Claims Resolution   |  Automated through oracles , service and on-chain data  | Depends on governance vote by MKR holders  | Depends on the product contributors and Agents  |
| Capital Efficiency  | Pools capital from multiple bonds for claims  | Uses CDPs to collateralize loans and back the DAI stablecoin  | Protocol capital from code, community and capital |
| Governance   | DAO with governance proposals voted on by veOLAS holders and multisig trusted accounts  | DAO with governance proposals voted on by MKR holders  |  Morpheus community  agents will regulate the suspicious activites  |
| Decentralization Level  | Highly decentralized, no reliance on third-party oracles  | Partially decentralized, relies on oracles for some data feeds  | Morpheus Agent Store  |
| Risk Management  | Relies on parametric and tokenomics triggers, automated claims processing  | Uses dynamic interest rates and liquidation mechanisms to manage risk  | Utilises the smart contract ranking mechanism to control the risk  |
| Transparency  | All data and transactions are publicly viewable on-chain and off-chain depend on visibility  | Some data and transactions may be obscured by CDPs and governance proposals  | All data and transaction are publicly viewable on-chain and leveraged encryption of LLM   |
| Scalability   | Potentially more scalable due to bonding nature of insurance   | Scalability challenges exist due to reliance on CDPs and on-chain governance  | Scalability inherited by public on-chains and Zk-tech  |
| Innovation  | Pioneering bonding mechanism, Co-owned AI ,Decentralized Autonomous Agents (DAAs) and DAO's  | Leading innovator in DeFi, constantly developing new lending and borrowing products  | The Smart Agent Protocol brings together the right mix of capabilities with LLMs, Agents,  Web3 and AI |
| Adoption   | Early stage, still gaining traction  | Widely adopted, with DAI being one of the leading stablecoins  | Early stage, still in development stage  |
| Overall  | A promising new approach to decentralized bonding mechanism with potential for wider adoption and automation model (AI) | A well-established DeFi platform with proven track record, but facing scalability challenges | A new innovation of AI-powered DeFi agents could usher in hyper-personalized finance, creating a dynamic system that bends to individual needs.  |



## 6. Recommendation
After manual observation of code and mechanism we recommend a simple changes which make more robustness to Autonolas protocol.

1. Contracts which are highly depent one the single owner address which can be land on risk if owner address get compromised by private key or any other factors and bonds , agents and components associated with that also landed on the risk as well.

Contract list are :-


 a.`Tokenomics`
 
 b. `Depository`
 
 c. `Treasury`
 
 d. `GuardCM`


The contract have the single owner address and function like below which means that only owner can add the new owner if any case owner address compromised or in-solvent then bonds , agent and components associated with this contract will landed on risk . Tokenomics can be updated with new implementation address but existed users will bear the huge loss.

```solidity
    /// @dev Changes the owner address.
    /// @param newOwner Address of a new owner.
    function changeOwner(address newOwner) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }


``` 

**Recommendation**

Our recommendation is to add setter function where governance or any trusted multi-sig contract can change the existed(compromised) owner address. Which will safe guard the protocol even owner address compromised.

```solidity
    /// @dev Changes the owner address.
    /// @param newOwner Address of a new owner.
    function setNewOwner(address newOwner) external {
        // Check for the contract ownership
        
        if(msg.sender != governor || msg.sender != MS) revert Error();

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }

```

2. Overflow check be removed while users donating ETH to service.

```solidity
        // Check for the overflow values, specifically when fuzzing, since practically these amounts are not realistic
        if (amount + ETHFromServices > type(uint96).max) {
            revert Overflow(amount, type(uint96).max);
        }
        ETHOwned = uint96(amount);

```

The above check in the function [depositServiceDoantionsETH()](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Treasury.sol#L127C1-L131C35) function, there's a check that reverts the transaction if the sum of the user-specified amount and the existing ETHFromServices exceeds uint96. From an economic perspective, we recommend removing this restriction and accepting amounts even beyond uint96. Users are unlikely to send excessively large donations, and even if they do, the additional funds can be held for protocol purposes only.


3. As we noticed that Autonolas encourage to deposit various whitelisted liquidity pool(LP) tokens and get in exchange OLAS tokens slightly discounted amount than market to increase the capitalisation of various protocol lp tokens which could create other investment opporunities and diversification of assets. As like that currently autonolas working with Gnosis chain and polygon chain we recommend to adopt arbitrum chain also which creates opporunites for users to conduct economics activities through arbitrum also. 


4. As we noticed that Autonolas have multiple internal and external audits with well detail oriented analysis reports but we recommend the Unit test code which means testing and verfying the each unit of the code base and formal verification the process of mathematically checking that the behavior of a system, described using a formal model, satisfies a given property, also described using a formal model.

5. Add pause and unpause mechanism for crucial functions on depository contract .

The [`Create()`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L183) function is used to initiate a bond with LP tokens with specific terms, while the [`deposit()`](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Depository.sol#L291) function is used to deposit LP tokens for specific bonds and by this functions user enters to economy of the protocol. In case the protocol faces any risk, further operations are halted until the risk is mitigated. By adding pause and unpause mechanism safeGuard the users funds and helps to build good will of the protocol.

6. Autonolas depend on the Uniswap v2 to fetch and determines the price of LP token . Uniswap itself mentioned associated risk like impermanent loss , TWAP's delays and other

```solidity

  IUniswapV2Pair pair = IUniswapV2Pair(token);
        uint256 totalSupply = pair.totalSupply();
        if (totalSupply > 0) {
            address token0 = pair.token0();
            address token1 = pair.token1();
            uint256 reserve0;
            uint256 reserve1;
            // requires low gas
            (reserve0, reserve1, ) = pair.getReserves();

```

The above code in function [getCurrentPriceLP()](https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/GenericBondCalculator.sol#L72C7-L80C57) which fetch the pairs address on Uniswap v2 and determines the price of lp tokens.

  Recommendation :-
  Our recommendation is to implement the chainlink 
  oracle to fetch the LP token price when Uniswap v2 
  pairs associated with risk through the comparison 
  statements.
 
 
7. Implement an emergency shutdown mechanism, or an ideal mechanism to halt crucial processes in the Autonolas protocol when major economic (market) risks and security risks arise. This mechanism could halt bond creation, agent/component creation, and oracle updates. It could be initiated by locking a certain large amount of OLAS tokens into governance (burned upon deposit) or through the exclusive voting mecahnism with help of `ERC20Permit` contract with verfied signature.This could safeguard user funds, limit potential losses, and help build integrity. Consider the emergency shutdown mechanism implemented by MakerDAO as a reference.


8. By adding delegation voting system  allows token holders to transfer their voting power to another address (the delegatee) without transferring the tokens themselves by `ERCVotes.sol` . 


Benefits are follws :-


Passive holders:-
People who hold tokens but may not be actively involved in the project can still participate in governance by delegating their votes to someone they trust.

Small holders:-
 Accounts with a small number of tokens can pool their voting power with others through delegation, giving them a stronger voice in governance decisions.


Reduced gas costs:-
 Voting directly can be expensive due to gas fees. Delegation allows users to avoid these fees by letting their delegatee do the voting.
 
Choose expertise:-
 Token holders can delegate their votes to individuals or groups with more knowledge and understanding of the project, potentially leading to better governance decisions.
 
Dynamic representation:- 
 Users can change their delegate at any time, allowing them to adjust their voting power based on the specific proposals or issues being considered.
 


9.Function `reserveAmountForBondProgram` allows to make `effectiveBond` as ZERO which breaks the invariant of the protocol could lead various issues that why we would like to mention in systematic risk section. Our recommendation is to change accoriding the comments which explictly mentioned.
 
 recommendation
 
 ```solidity
 // Effective bond must be bigger than the requested amount
        uint256 eBond = effectiveBond;
        if (eBond > amount) { //@audit changed here
```
Code snippet:-
https://github.com/code-423n4/2023-12-autonolas/blob/main/tokenomics/contracts/Tokenomics.sol#L615C1-L617C31



## 7. Hours spend 
  
 50 hours 


## 8. Comments for judge 

1.Representing the smart contract working flow through the diagram. 

2.Extensive analysis of contract by contract analysis. 

3.Best representing systematic risk.

3.Comparison table with other competitive protocol.

4.Best recommendation which could safely implemented.



### Time spent:
53 hours