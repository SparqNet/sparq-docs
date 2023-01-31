# File net/grpcServer.md

## About the object grpcServer

The main purpose of 'grpcServer' is to receive requests from _MainNet_ (AvalancheGo), the communication between the Node and the network is through [**'Protocol Buffers'**](https://developers.google.com/protocol-buffers) from Google and [gRPC](https://grpc.io/), the message structure implements the methods from ```VMServiceImplementation``` with the expected packets by the VM.

This class doesn't send any request to AvalancheGo by itself, even though some requests may start procedures needing later to request the AvalancheGo (see grpcClient for more information about requesting directly to AvalancheGo).

Examples of ```VMServiceImplementation``` and request forwarding can be found at **_src/subnet.cpp_**.

**_Note:_** The base structure of the expected behaviour of requests can be found at **vm.proto**.

## Initialization

Initialized at ```Subnet::start``` the constructor only inherit the ```Subnet``` instance, after the Node finish initializing itself the binary outputs in terminal the IP and port ```std::cout << "1|20|tcp|" << server_address << "|grpc\n" << std::flush;``` so the VM can register the Node connection.

## Class Members of grpcServer

Every method in code of this class is a direct implementation of ```VMServiceImplementation```, at any time the AvalancheGo can request the implemented methods, even though the **Node doesn't have all API Calls of AvalancheGo**. 

### grpcServer: Initialize

When the _Mainnet_ make this request means the network recognised the Node, and the _complete_ initialization can begin.

### grpcServer: SetState

"_The Mainnet can emit the current stage it is in, the following:_"

```json
{
  "state": {
    "0": "initializing",
    "1": "StateSyncing",
    "2": "Bootstrapping",
    "3": "Normal Op"
  }
}
```

Despite the quote above, the system does not support this request, it always returns the most recent block.

### grpcServer: BuildBlock

Request to create a new block (see [core/subnet.md](../core/subnet.md) section _blockRequest_).

### grpcServer: ParseBlock

Parse the block, must be in 'Chain Head' or 'Chain Tip' (see [core/subnet.md](../core/subnet.md) section _ParseBlock_).

### grpcServer: StateSyncEnabled

Asks if 'StateSync' is enabled, by default it returns ```true```.

**_Caution_**: Documentation of this call was requested, but those responsible did not return.

### grpcServer: SetPreference

Sets the preferred (or best candidate) block in 'Chain Tip', later the function **acceptBlock** will add this block in 'Chain Head' (See [core/chainTip.md](../core/chainTip.md) section _setPreference_).

### grpcServer: Version

Returns the version of 'Block-chain', of the Node, any arbitrary value does not return errors or incompatibility.

### grpcServer: GetBlock

When the _Mainnet_ send the Hash any block its returned the block, either present in both 'Chain Head' or 'Chain Tip' (see [core/subnet.md](../core/subnet.md) section _getBlock_).

### grpcServer: GetAncestors

Return a list of blocks based on the criteria of 'depth', 'size' and 'time' (see [core/subnet.md](../core/subnet.md) section _getAncestors_).

### grpcServer: BlockVerify

The _Mainnet_ sends a block to the Node, if the block's origins matches the current 'Chain Head', it will be added to 'Chain Tip' (see [core/subnet.md](../core/subnet.md) section _verifyBlock_).

### grpcServer: BlockAccept

The _Mainnet_ sends a Hash of a previously in 'Chain Tip' and subsequently added to 'Chain Head' (see [core/subnet.md](../core/subnet.md) section _acceptBlock_).

### grpcServer: BlockReject

Flags a block already present in 'Chain Tip' with the status ```BlockStatus::rejected```.

### grpcServer: BatchedPaseBlock

Same procedure found in 'ParseBlock' but in sequence of the blocks sent by the _Mainnet_.

### grpcServer: Connected

Add the credentials of another Node connected in AvalacheGo to the list of connected Nodes.

### grpcServer: Disconnect

Removes a given Node from the list of connected Nodes.

### grpcServer: Shutdown

The _Mainnet_ emit the shutdown signal, triggering the ```Subnet::shutdown```.

## API Calls not implemented by Subnet

Every item on the following list returns an empty request with ```Status::OK```, because the system does not use those functions or wasn't implemented.


* AppGossip
* VerifyHeightIndex
* CreateHandlers
* CreateStaticHandlers
* Health
* AppRequest
* AppRequestFailed
* AppResponse
* Gather
* CrossChainAppRequest
* CrossChainAppRequestFailed
* CrossChainAppResponse
* GetBlockIDAtHeight
* GetOngoingSyncStateSummary
* GetLastStateSummary
* ParseStateSummary
* GetStateSummary