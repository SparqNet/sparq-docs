# Sparq Overview

## The Problem

Contracts on chains with Virtual Machines (EVMs), like Ethereum and other derived chains, have pretty limited speed and flexibility due to having to share a single, generic computer with limited throughput with the whole world. That is, by default, inefficient by design - constraining the chain to be both generic and decentralized at the same time puts heavy limits on which types of applications you can decentralize and how much you can decentralize them. For example, in the Ethereum EVM, you're *not* able to:

* Loop a function more than 50 times due to block gas limit constraints;
* Have a stack size larger than 16 variables because of the EVM itself;
* Parallelize multiple contract calls (as in, every time a new block has multiple transactions that interact with multiple different contracts, you have to load the contract, parse the changes and save those changes to the database for *each* single one of these contracts, *in order*).

## The Solution

Make native blockchains great again.

By having a natively-coded blockchain, you can apply performance optimizations to the code specifically for your application's implementation, while also providing the flexibility of developing in common performance-driven development languages such as C++, C#, Rust, Go, and others.

Sparq provides developers with tools and documentation to ease the creation of application-specific chains that give them the freedom they need to make decentralized Web2 applications.

## The Caveats (and Solution)

Due to the solution being implementing a **native** and **application-specific** blockchain, this brings another set of problems:

* It's not as easy to push new contract code into the Subnet. If you want to include or exclude logic from your contract in your network, you have to force a mandatory update through all your node operators, which can eventually take *days*
* An application-specific chain is made for *only one* type of application. This would make DeFi impossible as most DeFi projects heavily depend on interaction between each other and each other's contracts. For example, you have deployed a AAA game with NFTs in your Subnet, but forgot to add an NFT marketplace. Now your users have to wait for days until A) you implement a marketplace, and B) everyone in the network updates their nodes to comply with your implementation

But what if you could *natively bridge* these assets between networks? This is what Sparq does. The Subnets use the Sparq Network to push/pull messages from/to each other by using the Sparq Network as a middleman, so neither network has to sync and verify each other completely.

## So what the hell is Sparq?

Sparq is two things:

* An SDK for developers to easily build their application-specific native chains; and
* A network that allows the bridging of data and assets between those chains on the network.

# How it works

## Validators

Validators are nodes set up by the community that have locked at least 10,000 Sparq through the `addValidator` transaction, using the validator's address as an argument.

They are the block creators, seeders of "randomness", and responsible for gathering and signing data on bridging and on blocks.

## Sentinels

Sentinels are similar to Validators, except they cannot *create* blocks nor act on their own. It is required that both randomly selected validators and sentinels send the same data to the requester, otherwise they'll be reported to the network as a malicious node. Sparq Labs Inc. hosts them to ensure that won't happen.
 
## Subnets

Subnets are blockchains built using the Sparq SDK and are able to communicate with the Sparq Network.

## Bridging

More details at [bridge.md](bridge.md).

### Sparq to Sparq Subnet

Bridging on the Sparq Network between two native chains goes like this:

- Subnet A requests Data X from Subnet B to the Sparq Network
- Sparq Network selects Y random Validators and Sentinels from the network using `RandomGen` (same in all nodes in the network)
- Nodes request Data X to Subnet B
- Nodes receive and sign Data X
- Nodes send back Data X + its signature to Subnet A
- Subnet A checks if everything matches, if not, someone is being malicious and can be reported to the network

### Sparq to External Network

**TBD**: multiple edge cases need more discussion with devs. The problem is that you're not able to natively push data into these chains without paying transaction fees, besides them being limited on how much signature verification you can do, external networks might be only bridged by us to ensure safety.

## rdPoS

The biggest problem with the native blockchain is "how the hell are we going to keep consensus if we're going too fast?". The answer is simple: random deterministic block creation only allows a given validator to create a block for a given time, eliminating the risk of a block race condition in the network.

This is implemented in Sparq as **rdPoS**, which stands for "*Random Deterministic Proof of Stake*", and is composed of a *block congestion system* and a *random generator system*. More details at [rdpos.md](rdpos.md)

`RandomGen` (`random.h`) within the Subnetooor code is a deterministic uint256_t generator used for almost everything related to consensus. Deterministic randomness assures that everyone has a chance to answer for a given request (block, randomness, bridging, etc.), while ensuring that the selected nodes from the network are truly random and not malicious nodes selected by an actor.

For `RandomGen` to be useful, it needs to be seeded with a truly random number. Our solution for that is as follows:

* Every time a new block is about to be created within the network, 16 random nodes are selected using `RandomGen` with the previous block randomness seed
* These randomly selected nodes make a 32 bytes random string (RandomnessSeed) and hash it (RandomnessHash), then they sign the hashed data and publish it to the network
* After all nodes have signed and published their hashed data to the network, they can post the true data without having to wonder if anybody is trying to manipulate the end result
* After the data is published and included in the block, the randomness seeds are concatenated and hashed, and the resulting hash is used to seed the next block creation

We have to pay attention to the current state of `RandomGen`, making sure that all nodes are always in the same internal state so they can properly sync with each other.

The genesis block enforces a given fixed randomness to be valid, since there is no previous block before genesis to derive the randomness from.

Several edge cases exist with this approach, for example, what happens if a node goes offline while assigned with something? There should be punishments for offline/deceiving nodes.

Besides that, we need to make rdPoS compatible with an Avalanche Subnet.
