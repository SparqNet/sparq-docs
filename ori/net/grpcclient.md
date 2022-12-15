# File net/grpcclient.md

## About the grpcClient object

The 'grpcClient' implements the 'interfaces' of **_aliasreader.proto_**, **_appsender.proto_**, **_keystore.proto_**, **_messenger.proto_** and **_sharedmemory.proto_**, allowing the 'Client' or Node make requests directily to AvalancheGo, these requests must be part of the **.proto** files scope.

## Initialization

Initialized at ```Subnet::start```its constructor receives three parameters, the first is the gRPC channel communication ```grpc->CreateChannel``` given the IP of AvalancheGo, the second parameter is the list of Nodes connected to AvalancheGO ```private Subnet::connectedNodes``` (see [core/grpcserver.md](grpcserver.md) in '_Connected_' and '_Disconnect_') and for the last parameter is the list of locks for connected Nodes ```private Subnet::connectedNodesLock```.

## Class members

This class only has one class member, and it's variables correspond to the 'interfaces' although the system not making all the calls the AvalancheGo "server" attends to.

### grpcClient: requestBlock

Requests AvalancheGo a new block to be processed. (See [core/state.md](core/state.md) at _validateTransactionForRPC_).