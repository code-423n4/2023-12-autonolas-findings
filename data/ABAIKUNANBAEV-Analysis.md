## Protocol overview

The purpose of the protocol is to make the system where users can represent their code by NFT on-chain via creating agents and components forming services. Additionally, the users can participate in the system by depositing LP tokens and minting OLAS tokens with discount in return via bonding mechanism. OLAS token, in its turn, is strictly limited by certain amounts that can be minted every year and 

This includes several processes:

 1. Creating registries: users can use `AgentRegistry` and `ComponentRegistry` contracts to represent their code on-chain and become part of a service.
 2. Bonding: there is an opportunity to buy OLAS tokens with discount to the market price by providing OLAS tokens in `Depository` smart contract
3. Donations: services can make donations to its units by using `Treasury` smart contracts


## Main contracts flow overview


### Depository.sol

The contract is used by users who wants to buy OLAS tokens with discount by providing LP tokens. The owner of the contract can create products with a certain supply (limited by `effectiveBond` amount from the `Tokenomics` smart contract) and users can deposit tokens and create bonds. The tokens are not immediately transferred, there is a waiting period (maturity) that needs to pass first.

### Treasury.sol
The contract where services can make donations in ETH to its units and update their rewards in `Tokenomics` contract. Except for that, inside of this contract some arbitrary donations can be made. There is also a function that drains ETH from the slashed services. When `unitIds` claim their incentives in `Tokenomics`, treasury is used to withdraw the funds.


### Dispenser.sol

The contract serves as a place where `unitIds` call `claimOwnerIncentives()` and get their rewards.


### GenericBondCalculator.sol
Here the OLAS amounts are calculated (taking into account discount factor parameter) for the users who made deposits in `Depository`.


### Tokenomics.sol

The contract where the inflation parameters (`inflationPerEpoch` and `inflationPerSecond`) for OLAS token is calculated and rewards are updated inside of the mappings according to their fractions depending on the type of `unitId` (component or agent or service). There is also `checkpoint()` function that is called after an epoch ends and updates the inflation and bond parameters if needed.


### DonatorBlacklist.sol
The contract sets the addresses of accounts that cannot make service donations inside of `Treasury`.

The diagramming of the main mechanisms were made to better understand the system:


### Overview of the tokenomics contracts


![image](https://github.com/rodiontr/fdkfd/assets/109477234/567936f5-f297-4291-bb58-1635c52056a4)



### Service donations mechanism


![image](https://github.com/rodiontr/fdkfd/assets/109477234/621ecb40-5db5-4896-a952-7ca8b54cd986)



### Bonding mechanism


![image](https://github.com/rodiontr/fdkfd/assets/109477234/5869ae98-11c4-4db3-848a-d4702152dcf5)



### Unit id reward mechanism


![image](https://github.com/rodiontr/fdkfd/assets/109477234/7e8c8584-dfb2-47df-88d6-f067934a70cf)


## Systemic risks

Inflation: there are several inflation parameters that needed to be closely followed as there is fixed amount of tokens that can be minted every year. In the case of a vulnerability, the tokenomics contracts can be severely damaged.


## Centralization risks

Centralized parameters: the developers have an ability to set different parameters such as parameters in `Tokenomics` smart contract that may affect the entire system. It's important to make sure that all those variables have reasonable values. Moreover, only the owners may pause the contracts and change the state of the system. 


## Security approach of the protocol

The protocol had multiple audits (both internal and external) in the past where all main contracts were audited. However, it's highly important to audit contracts comprehensively to be confident that nothing is missed. This can be done via private audits, audit contests and more audits from the firms.


## Approach taken when evaluating the codebase

 1 Overview of the protocol: at first stage, I looked into the protocol's contract very fast making some notes regarding of how the system is functioning overall.
 2 Deep dive: after the quick look, I started to review contracts separately trying to form the clear picture in my head of how every action in the system happens and what are the main actors who initiate the system state changes.
 3 Running tests: by setting up the testing environment, I made sure that all the tests passed and nothing remained uncovered.
 4 Complex attack vectors: finally, after getting the full understanding of the protocol, I began to create some complex attack scenarios in my head trying to find new vulnerabilities.



## Conclusion


In conclusion, the protocol presented a comprehensive and intricate system aimed at enabling users to represent their code as NFTs on-chain, participate through bonding mechanisms, and engage in various processes such as creating registries, bonding, and making donations to services. 

The main contracts, including `Depository`, `Treasury`, `Dispenser`, `GenericBondCalculator`, `Tokenomics`, and also registries contracts together form a sophisticated ecosystem with well-defined roles and functionalities.
The systemic risks, particularly related to inflation parameters, were highlighted, emphasizing the need for vigilant monitoring to prevent potential vulnerabilities that could adversely impact the tokenomics contracts. Additionally, centralization risks were identified, underscoring the importance of carefully managing parameters and ensuring responsible ownership to avoid undue influence on the system.

The security approach adopted by the protocol involved multiple audits, both internal and external, demonstrating a commitment to robust security practices. However, the acknowledgment of the need for ongoing comprehensive audits and exploration of various audit methods, including private audits and contests, reflects a proactive stance towards ensuring the continued security and resilience of the protocol.

The evaluation approach involved a phased process, starting with an overview of the protocol, followed by a deep dive into individual contracts, rigorous testing, and consideration of complex attack vectors. This methodical approach demonstrated a thorough understanding of the protocol's intricacies and a proactive effort to identify and address potential vulnerabilities.

In summary, the protocol exhibits a well-designed and carefully considered architecture, supported by a diligent security approach. The outlined evaluation process reflects a commitment to maintaining the integrity and security of the system, providing a solid foundation for users to confidently engage with the protocol and its associated functionalities.


### Time spent:
25 hours