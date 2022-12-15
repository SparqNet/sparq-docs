# File core/chainHead.md

## About the object ChainHead

The 'ChainHead' or 'Header of the Blockchain' is represented by a collection of records of blocks, where transactions, contracts and accounts are stored in blocks created by the Mainnet, other Nodes or by itself, those records cannot be changed and is accessible only to read/query.

The creation of new records are made through the processes in the class 'Chain Tip' (see [chainTip](chainTip.md) for more details), in a simplified new blocks are created (containing data about transactions, contract data and accounts), and appended to.

The querying of the data is made in multiple functions of the system, it can be affirmed the 'ChainHead' together with 'DB' are the final goal of the procedures done in the ecosystem.

## Initialization

The 'ChainHead' receives an instance of the database ```DBService dbServer``` during the initialization of the Node in ```Subnet::inititalize```, where every record is restored from a previous initialization, if the database (DB) is empty a ```Block genesis``` is created with testing local validators.

After the initializatio is done, a back-up routine is started every 15 seconds the data is saved (see **_periodicSaveToDB_**).

## Class members of ChainHead

All the functions with exception to the methods **_push_back_**, **_pop_back_** and **_dumpToDB_** are read-only.

### ChainHead: push_back

When a new block is accepted, it is moved to the end of 'ChainHead'.

### ChainHead: pop_back

Removes the last block accepted by the network.

### ChainHead: exists [overflow]

Verifies if the block exists by the Hash or its 'number height' (```nHeight```).

### ChainHead: getBlock [overflow]

Returns the block (more especifically the block's pointer in memory) based on the Hash of the block, or its 'number height' (```nHeight```).

### ChainHead: hasTransaction

Verifies if the transaction was processed based on the Hash's transaction.

### ChainHead: getTransaction

Returns the transaction's data based on the Hash of transaction (uses **_hasTransaction_** to verify if it was processed).

### ChainHead: getBlockFromTx

Returns the block of a given Hash from the Transaction.

### ChainHead: latest

Returns the latest block approved by the _Mainnet_.

### ChainHead: periodicSaveToDB

A 15 seconds periodic save of the records in the database. This routine is initialized by the class constructor. 

### ChainHead: dumpToDb

Copy all the content in memory back to the database (DB), this method is only used when the Node start the shutdown procedure in ```Subnet::stop```.