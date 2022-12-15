# File core/state.md

## About the object 'State'

The objective of 'State' object is to represent the state of accounts, transactions, native balance, token balance and contracts. Only being possible to change the data stored through the process of block creation, either on your own or received by the _Mainnet_ (See [**_core/subnet.md_**](subnet.md) for the use cases).

## Initialization

The 'State' receives both database access (DB) and ```grpcClient``` to communicate with AvalancheGo, for example: Request AvalancheGo a new block.

## Class members of State

All the class members behave to represent the current State of the Chain or Node and to keep a synchronized state in the ecosystem of "Subnets" between all the Nodes connected to the _Mainnet_.

### State: getNativeBalance

Returns requested account native balance.

### State: getNativeNonce

Returns requested account transaction number.

### State: validateNewBlock

Verifies if new block given as argument is valid to continue the blockchain, checking signatures, transactions and other obrigatory information.

### State: processNewBlock

Process the new block and updates 'State', this method is called only by the 'Chain Tip' when the network requests that a new block will be accepted (Check ```Subnet::acceptBlock``` [here](subnet.md)).

### State: createNewBlock

Creates a new block from the preferred (or best candidate) block on 'Chain Tip' when the following conditions are satisfied:

1. There is a candidate in 'Chain Tip' selected by the network
2. There is the Block's Hash in 'Chain Tip' 
3. There is a block corresponding to the Hash in the 'Chain Head'.

If any condition above fails ```nullptr``` will be thrown, and the operation will be canceled.

### State: validateTransactionForBlock

Validates the following conditions in a transaction:

1. The transaction has already been validated.
2. The transaction is present  in 'memory pool'.
3. Account exists and if the account has ~~any~~ funds to make any transaction.
4. The account has enough balance to complete the transaction, and its Nonce is valid.

If any condition fails ```false``` is returned.

**_Warning_**: This operation does not change the Block-chain State

### State: validateTransactionForRPC

Performs the same validations that **_validateTransactionForBlock_** however it's exclusively used Nodes from **Sparq Network** ecosystem.

**_Warning_**: This operation does not change the Block-chain State.