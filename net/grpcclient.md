# Arquivo net/grpcclient.md

## Sobre o objeto grpcClient

O 'grpcClient' implementa as ‘interfaces’ de **_aliasreader.proto_**, **_appsender.proto_**, **_keystore.proto_**, **_messenger.proto_** e **_sharedmemory.proto_**, permitindo que o 'Client' ou Node faça solicitações diretamente ao AvalancheGo, essas requisições devem fazer parte do escopo dos arquivos **.proto**.

## Inicialização

É Inicializado em ```Subnet::start```, e seu construtor recebe três parâmetros, o primeiro é o canal de comunicação do gRPC ```grpc->CreateChannel``` que é preciso o endereço de IP do AvalancheGo, segundo parâmetro trata-se da lista de Nodes conectados ao AvalancheGo ```private Subnet::connectedNodes``` (veja [core/grpcserver.md](grpcserver.md) em '_Connected_' e '_Disconnect_'), e por último a lista de travas para os Nodes conectados ```private Subnet::connectedNodesLock```.

## Membros da classe

Essa classe só possúi um membro de classe, e suas variáveis correspondem às 'interfaces' apesar do sistema não realizar todas as chamadas que o "servidor" do AvalancheGo atende.

### grpcClient: requestBlock

Solicita para AvalancheGo um novo bloco para ser processado (veja [core/state.md](core/state.md) em _validateTransactionForRPC_).