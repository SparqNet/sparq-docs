# File core/chainTip.md

## About the object ChainTip

The purpose of the 'ChainTip' is to keep track of the blocks received by the _Mainnet_, it contains information such as the preferred block (chosen by the Main Network), if the block was accepted, being processed or rejected.

Simplifying the 'ChainTip' acts similar to a "mempool", when a block reaches a consensus (go from "processing" to either "accepted" or "rejected" state), the block does not belong in 'ChainTip' anymore, and is discarted (if rejected) or moved to the 'Chain Head' (if accepted).

The 'ChainTip' cannot accept a previously added block, this rule applies to every state a block can be, and to prevent we save a reference in ```cachedBlockStatus <Hash, BlockStatus, SafeHash>``` the state of every received block.

## Initialization

Normal initialization in ```Subnet::initialize```, with a pointer.

## Class members of ChainTip

The members of this class are limited to access the variables ```Hash preferedBlockHash```, ```unordered_map<Hash, Block*, SafeHash> internalChainTip``` and ```unordered_map<Hash, BlockStatus, SafeHash> cachedBlockStatus```.

In the future at the [process of accept a block](subnet.md), the stored data will be copied to a new block and added to 'Chain Head'.

### ChainTip: setBlockStatus

Set the block status present in ```cachedBlockStatus```.[chainHead.md](..%2F..%2F..%2F..%2F..%2FDownloads%2FchainHead.md)

**_Caution:_** This method is not used anywhere in the system.

### ChainTip: getBlockStatus

If the block is present at 'ChainTip', recover the state of a block in ```cachedBlockStatus```. 

### ChainTip: processBlock

Adds a given block to both ```internalChainTip``` and ```cachedBlockStatus``` to be processed by the next operations of the _Mainnet_.

### ChainTip: isProcessing

Verifies if the requested block is being processed, if the block wasn't found returns ```false```, otherwise returns the query.

Return ```false``` if the state is different of ```BlockStatus::Processing```.

### ChainTip: accept

The _Mainnet_ sends a block to the Node to be accepted, if the block was previously verified and is not being processed the block will be redirected to [State::processNewBlock](state.md), and added to 'Chain Head'.

### ChainTip: reject

The _Mainnet_ sends a block previously verified to be rejected.

### ChainTip: exists

Verifies if the block is in 'ChainTip' ```internalChainTip```.

### ChainTip: getBlock

Returns the block added previously in 'ChainTip', if it does not exist an exception will be thrown.

### ChainTip: getPreference

Returns the Hash of the preferred block, if the network didn't decide the best candidate/preferred block an exception will be thrown.

### ChainTip: setPreference

Set the preferred block.