# Arquivo net/grpcServer.md

## Sobre o objeto grpcServer

O objeto 'grpcServer' têm o objetivo principal de receber as solicitações da _Rede Principal_ do AvalancheGo, a comunicação entre o Node e a rede é feita pelo mecanismo [**'Protocol Buffers'**](https://developers.google.com/protocol-buffers) do Google e [GRPC], as mensagens são recepcionadas a partir da implementação dos métodos de ```VMServiceImplementation``` que retornam pacotes esperados pela VM.

Esta classe não inicializa nenhuma solicitação direta ao AvalancheGo, apesar de algumas solicitações darem inicio a processos que podem solicitar o AvalancheGo diretamente (veja grpcClient para mais informações sobre solicitar dados ao AvalancheGo).

Exemplos de ```VMServiceImplementation``` e redirecionamento da rede podem ser encontradas em **_src/subnet.cpp_**.

**_Atenção:_** A estrutura de como é trafegado as solicitações podem ser encontradas em **vm.proto**.

## Initialização

É Inicializado em ```Subnet::start``` e seu construtor recebe apenas o ponteiro da classe ```Subnet```, após o Node terminar o processo de inicialização do mesmo, é emitido em terminal o IP e Porta no trecho ```std::cout << "1|20|tcp|" << server_address << "|grpc\n" << std::flush;``` para que a máquina virtual capture e registre a subnet.

## Membros da classe grpcServer

Todos os membros desta classe correspondem a implementação de ```VMServiceImplementation``` que em algum momento a AvalancheGo pode solicitar, porem o Node **não utiliza todas as chamadas do AvalancheGo**, e para esses casos é retornado ```Status::OK```.

### grpcServer: Initialize

Quando a _Rede Principal_ solicita este método significa que o Node faz parte da rede, e que a inicialização seja finalizada. 

### grpcServer: SetState

"_É sinalizado ao Node através do AvalancheGo a situação que o Node se encontra, sendo os estados de:_"

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

Apesar da citação acima o sistema não possue suporte para esta solicitação, em toda via retorna o bloco mais recente.

### grpcServer: BuildBlock

Solicitação de criação para um novo bloco (veja [core/subnet.md](../core/subnet.md) no método _blockRequest_).

### grpcServer: ParseBlock

Faz a análise do bloco, esteja ele tanto no 'Chain Head' quanto 'Chain tip' (veja [core/subnet.md](../core/subnet.md) no método _ParseBlock_).

### grpcServer: StateSyncEnabled

Solicita se o 'StateSync' está habilitado, por padrão é retornado ```true```.

**_Atenção:_** Foi solicitado a documentação dessa chamada, porém os responsáveis não retornaram.

### grpcServer: SetPreference

Ajusta a preferência (ou melhor candidáto) do bloco presente em 'Chain Tip', posteriormente o **_acceptBlock_** irá dar início a adição do bloco ao 'Chain Head' (veja [core/chainTip.md](../core/chainTip.md) no método _setPreference_).

### grpcServer: Version

Retorna a versão da 'Block-chain' que o Node pode ser representado, qualquer valor abritário de versionamento não retorna erros ou incompatibilidade.

### grpcServer: GetBlock

Quando informado o Hash do bloco pela _Rede Principal_, é retornado o bloco esteja ele na 'Chain Head' ou 'Chain Tip' (veja [core/subnet.md](../core/subnet.md) no método _getBlock_).

### grpcServer: GetAncestors

Retorna uma lista de blocos baseando-se nos critérios de 'depth', 'size' e 'time' (veja [core/subnet.md](../core/subnet.md) no método _getAncestors_).

### grpcServer: BlockVerify

A _rede principal_ envia um bloco ao Node, se o bloco fazer parte da sequencia será adicionado ao 'Chain Tip' (veja [core/subnet.md](../core/subnet.md) no método _verifyBlock_).

### grpcServer: BlockAccept

É enviado o Hash do bloco já presente em 'Chain Tip' e em sequência adicionado ao 'Chain Head' (veja [core/subnet.md](../core/subnet.md) no método _acceptBlock_).

### grpcServer: BlockReject

Marca um bloco já presente na 'Chain Tip' com ```BlockStatus::rejected```.

### grpcServer: BatchedParseBlock

Realiza o processo de 'ParseBlock' na sequencia que os blocos foram enviados pela _Rede Principal_.

### grpcServer: Connected

Adiciona as credênciais de outro Node a lista de Nodes conectados acessíveis do AvalancheGo.

### grpcServer: Disconnected

Remove um Node da lista de Nodes conectados acessíveis do AvalancheGo.

### grpcServer: Shutdown

Sinal de desligamento enviado pela _Rede Principal_, dá início ao processo em ```Subnet::shutdown```.

## Funções não utilizadas pela Subnet

A lista a seguir corresponde a todas as funções com resposta vazia ```Status::OK```, pois o sistema não utiliza dessas funções ou não foi implementado.

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