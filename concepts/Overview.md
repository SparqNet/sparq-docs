# Sparq Overview

### The Problem:

Speed & Flexibility on contracts with Virtual Machine chains like Ethereum or other EVM chains is pretty limited, having to share a single computer that has limited throughput and is generic with the whole world is, by default, inefficient due to the design required to be both generic and decentralized at the same time, some of these constraints put heavy limits on which type of application you can decentralize, for Ethereum in example, you are not able to loop a function more than 50 times due to block gas limit constraints, you are not able to have a stack size larger than 16 variables, because of the EVM itself, besides that, every time a new block has multiple transactions that interact with numerous different contracts, you have to load, the contract, parse the changes and save to the DB for each one of these contracts.


### The Solution

Make native blockchain great again. By being native, you can apply performance optimizations to the code specifically for your application implementation, besides providing the developer the Flexibility of common development languages (such as C++, C#, Rust, Go, and others).

Sparq provides developers with tools and documentation to ease the creation of application-specific chains that give them the freedom they need to make web2 applications decentralized.

### The Problem within the Solution

It says on the name, **native** blockchain, not dynamic, and you are not able to easily push new contract code into the Subnet, if you want to include/exclude logic from your contract in your network, you have to force a mandatory update through all your node operators, which can eventually take days.

Besides that, these are **application-specific chains**, they are made for only one type of application, for that reason, DeFi would be impossible as most DeFi projects heavily depend on each other contracts and interaction with each other.

For example, you have deployed a AAA game in your Subnet that has NFTs but forgot to add an NFT marketplace, now, your users have to wait for days until you implement a marketplace, and everyone in the network updates their nodes!

### Solution of the Solution Problem

But what if you could natively bridge these assets between the networks? That is what Sparq does, the Subnets use the SparqNetwork to push/pull messages from each other by using the SparqNetwork as a middleman, and neither of both networks has to sync and verify each other completely.

### Conclusion: What the hell is Sparq?

Sparq is two things, an SDK for developers to easily build their application-specific native chains and a network that allows the Bridging of data and assets on the network.

# How it works:
### Validators

Validators are community setup nodes in the network that has locked at least 10k Sparq within the addValidator transaction using the validator address as arguments.

They are the block creators, the seeders of the randomness, and who gather the data and signs for bridging data.

### Sentinels

Sentinels are similar to validators, but they are not block creators themselves, Sparq Labs Inc hosts them to ensure that the edge case of all the randomly selected validators is malicious not to occur and be reported to the network.

They commit the same way as validators, bridging or on blocks.

Sentinels cannot create blocks or act on their own, it is required that both randomly selected validators and sentinels send the same data to the requester, otherwise, a malicious node report will occur.
 
### Subnets

Subnets are blockchains built using the Sparq SDK and have the compatibility layer to talk back with the SparqNetwork.

### Bridging

Sparq to Sparq Subnet

Bridging on the SparqNetwork between two native chains is as follows:

- Subnet A requests Data X from Subnet B to the SparqNetwork
- SparqNetwork selects Y random validators and sentinels from the network using RandomGen (same in all nodes in the network)
- Nodes request data X to Subnet B,
- Nodes sign the data
- Nodes sends back signature + data to Subnet A
- Subnet A checks if everything matches, if not, someone is being malicious and can be reported to the network. 

Sparq to External Network:

**TBD**, multiple edge cases need more discussion with devs.
The problem is: you are not able to natively push data into these chains without paying transaction fees, besides them being limited on how much signature verification you can do, external networks might be only bridged by us to ensure safety.

### rdPoS - Random Deterministic Proof of State - Block Congestion System + Random Generator System

The biggest problem with the blockchain is "how the hell are we going to keep consensus if we are going too fast?" simple: random deterministic block creation only allows a given validator to create a block for a given time, eliminating the risk of a block race condition in the network.

RandomGen (random.h) within the Subnetooor code is a deterministic uint256_t generator used through the code for almost everything related to the consensus, deterministic randomness assures that everyone has a chance to answer for a given request (block, randomness, or Bridging)  besides making it that the selected nodes are random nodes from the network and not malicious selected nodes by an actor.

For RandomGen to be useful, it needs to be seeded with a truly random number, our solution for that is as follows:

Every time a new block is going to be created within the network, 16 random nodes are selected using RandomGen with the previous block RandomnessSeed, these nodes make a 32 bytes random string (RandomnessSeed) and hash it (RandomnessHash), then they sign the hashed data and publish into the network after all nodes have signed and published their hashed data into the network, they can post the true data without having to wonder if anybody is trying to manipulate the end result.
After the data is published and included in the block, the randomnessSeeds can be concatenated and hashed, the hash can be used to seed the next block randomness validators and the block randomness for the transactions.

We have to pay attention heavily to what is the current state of the RandomGen, always to make sure that all nodes are in the same internal state to sync with each other properly.

The genesis block enforces a given randomness to be valid (there is no previous block before block 0 to derive the randomness from)

Several edge cases exist with this approach, for example, what happens if a node goes offline while assigned with something? there should be punishments for offline/deceiving nodes.

Besides that, we need to make rdPoS compatible with an Avalanche Subnet.

