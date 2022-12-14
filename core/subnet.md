# Arquivo core/subnet.md

### Sumário

* **Sobre o Objeto**
* **O Papel do AvalancheGo Daemon**
* **Por que AvalancheGo?**
* **Pre-inicialização**
* **Inicialização**
* **Membros da classe Subnet**
* **Desligamento**

**_TODO:_** Sempre atualizar o Sumário com os tópicos mais recentes

## Sobre o objeto Subnet

Conforme exemplificado anteriormente no [arquivo ponto de entrada](../src/main.md) a finalidade da classe Subnet é encapsular e redirecionar os módulos dos Nodes onde a Subnet atuará como um intermediador, a Subnet (Subnetooor) também atuará como uma engine de execução para aplicações de smart contracts em [Solidity](https://github.com/ethereum/solidity) com integração de múltiplas vias da Rede Subnets Sparq.

Atualmente a única integração de comunicação dos Nodes é dependente das VM (Virtual Machines, mais em [AvalancheGo](https://github.com/ava-labs/avalanchego/tree/master/vms)) da [AvaLabs](https://www.avalabs.org/), e está sendo desenvolvido uma comunicação de Ponta-A-Ponta entre as Subnets (Subnetooor) e também um modelo de requisições para que outras Subnets realizem a conexão e comuniquem entre sí.

## O Papel do AvalancheGO Daemon

A AvalancheGo e nossa Subnet se comunicam pelo serviço gRPC para se comunicar com os nodes, o mesmo é uma Virtual Machine executando o binário Subnet (Subnetooor*) que servirá como Rede Principal para os Nodes, a própria VM Local faz a leitura da saída do terminal em "``std::cout << "1|19|tcp|" << server_address << "|grpc\n" << std::flush;``" e faz a propagação do IP para os Nodes que estão sendo executados em paralelo (pelo comando ``scripts/AIO-setup.sh``), além disso, o AvalancheGo adiciona a Subnet (Subnetoor*) a lista de chains disponíveis.

De acordo com a [*documentação*](https://docs.avax.network/subnets/create-a-local-subnet) do AvaLabs cada alocação gera as block-chain; X-Chain, P-Chain e C-Chain, e cada "Chain" com suas aplicações e contratos em execução, o papel do AvalancheGO Daemon é de uma Rede Principal Local onde os Nodes façam a conexão com a Subnet desejada fora da rede e internamente as solicitações são validadas.

```mermaid
flowchart LR
label1["AvalancheGo BlackBox"]
label2["AvalancheGo BlackBox"]
label3["AvalancheGo BlackBox"]
subgraph A[AvalancheGo 5a5b6e]
    aX("X-Chain")
    aP("P-Chain")
    aC("C-Chain")
    as1(Subnet)
    subgraph ad1["Node 1"]
        label1
    end
    aX <-.gRPC.-> ad1
    aP <-.gRPC.-> ad1
    aC <-.gRPC.-> ad1
    as1 <-.gRPC.-> ad1
end

subgraph B[AvalancheGo a80982]
    bX("X-Chain")
    bP("P-Chain")
    bC("C-Chain")
    bs1(Subnet)
    subgraph bd1["Node 2"]
        label2
    end
    bX <-.gRPC.-> bd1
    bP <-.gRPC.-> bd1
    bC <-.gRPC.-> bd1
    bs1 <-.gRPC.-> bd1
end

subgraph C[AvalancheGo abd4a3]
    cX("X-Chain")
    cP("P-Chain")
    cC("C-Chain")
    cs1(Subnet)
    subgraph cd1["Node 3"]
        label3
    end
    cX <-.gRPC.-> cd1
    cP <-.gRPC.-> cd1
    cC <-.gRPC.-> cd1
    cs1 <-.gRPC.-> cd1
end

A <-.?? Network Connection ??.-> B
B <-.?? Network Connection ??.-> C
C <-.?? Network Connection ??.-> A
```

#### Atenção

Cada Virtual Machine do AvalancheGo está executando uma cópia de Subnetoord, verifique o arquivo **_debug.txt_** de cada VM em **_$GO_DIRECTORY/src/github.com/ava-labs/avalanchego/node{n}_** para os logs do binário de cada Virtual Machine, o mesmo arquivo se encontra no diretório do projeto para os logs do binário local.

## Por que AvalancheGo?

Um dos pré-requisitos a serem satisfeitos no início do projeto foi a implementação do próprio AvalancheGo, grande feito que surgiu impacto no escopo do projeto más também beneficiou a imagem pública do projeto.

## Pre-inicialização

Após a inicialização das VMs do AvalancheGo temos 60 segundos até a inicialização do binário da Subnet (Subnatooor), pois assim que o tempo limite (60 segundos) for atingido todas as VMs irão receber uma solicitação cURL para o registro da blockChain na rede AvalancheGo.

**cURL em AIO_Setup.sh:**

```json
{
  "jsonrpc": "2.0",
  "method": "platform.createBlockchain",
  "params" : {
      "subnetID"   : "'$SUBNET_ID'",
      "vmID"       : "'$SUBNET_ID'",
      "name"       : "Subnetooor",
      "genesisData": "0x68656c6c6f776f726c648f8f07af",
      "username"   : "'$USERNAME'",
      "password"   : "'$PASSWORD'"
  },
  "id": 1
}
```

Esse tempo limite ocorre porque é um pré-requisito das VMs que sejam "configuradas", isto é receber na rede principal a solicitação de registro do Node Intermediário pelo método ```platform.createBlockchain``` e logo em seguida a inicialização do ```Subnet::start``` do nosso binário com a saída da porta do serviço pelo terminal.

Durante o processo de ```Subnet::start``` é inicializado um servidor gRPC (```shared_ptr Subnet::grpcServer```) que disponibiliza de forma assíncrona um canal de comunicação ao AvalancheGo, os métodos como; inicialização, shutdown e validar blocos, entre outros são recebidos por esse canal e podem ser encontrados em ```proto/vm.proto```.

## Inicialização

Após o AvalancheGo registrar a Subnet em sua lista de Nodes, ele sinalizará o comando ```rpc Initialize(InitializeRequest)``` que corresponde ao método ```Subnet::initialize``` do Subnet (Subnetooor), onde nele irá ser instanciado os membros:

* DB
* grpc Client
* State
* ChainHead
* ChainTip
* BlockManager
* P2PManager
* HttpServer

Além disso, a Subnet armazena no struct ```InitializeRequest``` as informações da rede que ele está conectado como o IP do servidor, chainID, subnetID entre outros.

### Sobre DB

As informações são armazenadas por um par de valores <Chave/Valor> dentro do DB, utilizando LevelDB, a 
'scheme database' atual é herdada das versões anteriores que se comunicava diretamente ao AvalancheGo. 

A escrita e leitura dos registros armazenados na base de dados é necessário o acompanhamento da tabela abaixo.

| Prefixo | Tipo de Dado | Comportamento | Valor                   |
| ------- | ------------ | ------------- | ----------------------- |
| 0001    | Key          | Block Hash    | Block                   |
| 0002    | Key          | Block nHash   | Block Hash              |
| 0003    | Key          | Tx Hash       | Transactions            |
| 0004    | Key          | Address       | Native Balance + nNonce |
| 0005    | ERC20        | -             | Tokens/State            |
| 0006    | ERC721       | -             | Tokens/State            |
| 0007    | Key          | Tx Hash       | Block Hash              |

A transmissão de leitura e escrita do DB é realizada pelo membro ```Subnet::dbServer``` que possui a implementação do protocolo em **_proto/rpcdb.proto_**.

### Sobre gRPC Client

O 'gRPC Client' pode solicitar diretamente ao AvalancheGo os procolos nos arquivos **_proto/aliasreader.proto_**, **_proto/keystore.proto_**, **_proto/metrics.proto_** e **_proto/sharedmemory.proto_**.

### Sobre o State

A classe de 'State' serve para armazenar o estado atual do sistema, atividades como saldo nativo (native balance), estados de contratos, mempool de transações (lista não ordenada de Hash, Base, SafeHash), balanço de tokens e variáveis/processos da block-chain. Somente blocos podem atualizar o 'State', seja por ele mesmo criar um ou receber um novo bloco da rede.

### Sobre Chain Head

A 'Chain Head' realiza o rastreamento da própria block-chain, fáz o armazenamento das confirmações de blocos/transações, essas informações são consultadas em diversas partes do sistema.

### Sobre o Chain Tip

O 'Chain Tip' é similar ao 'Chain Head', porém ele realiza o rastreamento dos blocos rejeitados e blocos sendo processados no momento, o mesmo também rastreia o bloco preferencial (bloco com maior chance) de ser aceito.

### Sobre o Block Manager

**_TODO:_** Aguardando documentação do Itamar

### Sobre o P2PManager

O serviço de ponta-a-ponta 'P2PManager' é feito por web-sockets, esse serviço é responsável pela propagação de transações e informações do bloco processado para as demais Subnets (Subnatooor).

### Sobre o HttpServer

O 'HttpServer' disponibiliza uma conexão direta para serviços dos frameworks web (Web3, ethers, etc) como MetaMask, CoinBase e Frame.

## Membros da classe Subnet

Conforme citado anteriormente nos tópicos '**Pre-Inicialização**' e '**Inicialização**' a _Rede Principal_ solicita de forma assíncrono diversos métodos relacionados ao estado do Block-chain (veja **_proto/vm.proto_** para mais detalhes), seja para perguntar se ambos estão em sinergia ou solicitações ao validador de bloco. A seguir a implementação das demais funcionalidades que a rede pode solicitar.

### Subnet: SetState

Conforme o código-fonte do AvalancheGo que implementa **_proto/vm.proto_**, esse método é chamado quando a _Rede Principal_ precisa sinalizar qual a situação que a rede se encontra (veja [net/grpcserver.md](../net/grpcserver.md) para o ID dos estados).

**_Atenção_**: Atualmente não implementado devido implementação recente por parte da Avalabs.

### Subnet: ParseBlock

A _Rede Principal_ irá solicitar a análise do bloco enviado ao Node, se ele faz parte do 'Chain Head' ou 'Chain Tip', se encontrado independente de ter sido rejeitado ou adicionado ao 'Chain Head' e suas condições sendo:

1. Em duas ocorrências pode retornar **_verdadeiro_**, a primeira quando já foi processada anteriormente, e a segunda quando é adicionado ao 'Chain Tip' para processamento.
2. A única ocorrência de retornar **_falso_** nesse estágio é quando ocorre uma excessão causada pelo 'Chain Tip'.

### Condição 1.

Verificação de se o Bloco já se encontra presente no 'Chain Head' ou em 'Chain Tip':

```mermaid

flowchart LR

MN1(("Main Network"))
MN2(("Main Network"))

MN1 --"c7c316..."--> A

A["Block 'c7c316...'\n"]

IF1{"Is in Chain Head \nor Chain Tip?"}

C1["ChainTip::processBlock"]
R1["Yes"]
R2["No"]
ERR["Exception"]
subgraph SG1["Node 1."]
    subgraph PS1["parseBlock"]
        A --> IF1
        IF1 --> R1
        IF1 --> R2
        R2 --> C1
        C1 --> ERR
    end
end
C1 --"return true"--> MN2
R1 --"return true"--> MN2
ERR --"return false"--> MN2

```

### Condição 2.

Verificando se o Bloco é menor ou o mesmo que o ultimo bloco adicionado pelo 'Chain Head':

```mermaid
flowchart LR

MN1(("Main Network"))
MN2(("Main Network"))

MN1 --"d0c93f..."--> A

A["Block 'd0c93f...'\n"]

IF1{"Block number \n>\n Latest Block number"}

C1["ChainTip::processBlock"]
R1["Yes"]
R2["No"]
R3["BlockStatus::Rejected"]
ERR["Exception"]
subgraph SG1["Node 1"\n.]
    subgraph PS1["parseBlock"]
        A --> IF1
        IF1 --> R2
        IF1 --> R1
        R1 --> C1
        R2 --> R3
        C1 --> ERR
    end
end
C1 --"return true"--> MN2
ERR --> R3
R3 --"return false"--> MN2
```
**_Atenção ¹:_** Não é rejeitado Blocos no futuro (Unknown ou nHeight maior que o esperado), pois o AvalancheGo irá negar o reenvio do mesmo bloco quando em um segundo momento que o bloco é valido para análise.

**_Atenção ²:_** 'ParseBlock' **não realiza verificação da lógica das transações**, apenas se as assinaturas são validas.

### Subnet: acceptBlock

O método ```Subnet::acceptBlock``` recebe da _Rede Principal_ o Hash correspondente a um bloco que está presente no 'Chain Tip', sua solicitação significa que o bloco está pronto para ser adicionado ao 'Chain Head', a rede AvalancheGo nunca deverá enviar outro bloco para ser aceito com o mesmo 'Number Height', se ocorrer acontecerá um erro de excessão não controlado.

### Subnet: rejectBlock

Essa chamada é utilizada pela _Rede Principal_ para rejeitar um bloco enviado posteriormente ao 'Chain Tip'.

### Subnet: setPreference

A _Rede Principal_ envia a Hash do bloco disposto na 'Chain Tip' que ainda não foi processado e define como "preferido", isto é o melhor candidato de ser adicionado ao 'Chain Head'. 

### Subnet: validateTransaction

Faz a validação de uma transação, exclusivo apenas para Nodes da Subnet (Subnetooor), é verificado se a conta têm saldo, se o 'nonce' é valido e se a transação não se encontra na 'memory pool'.

**_Atenção_**: Essa movimentação da transação ao 'memory pool' não altera o 'State' do programa (consulte o arquivo de definição **_state.h_** para saber oque altera o estado da aplicação). 

### Subnet: verifyBlock

Faz a validação das transações do bloco, se o Hash do novo bloco têm como bloco anterior o último bloco da 'Chain Head', se sua altura númérica está sequenciada, e se a assinatura do bloco é coerente ao validador primário na lista de Nodes conectados.

Se todas condições forem satisfeitas o bloco será adicionado ao 'Chain Tip', e o Subnet (Subnatooor) retorna ```Status: Ok```.

### Subnet: getAncestors

Verifica as origens do bloco por sua 'depth', 'size' e 'time', se o bloco existir dentro do 'Chain Head' é feito uma verificação do ponto de partida até o bloco mais recente.

### Subnet: getBlock

Retorna um bloco se ele existir na 'Chain Head' ou no 'Chain Tip', se o bloco já se encontra na 'Chain Head' retorna ```Status: Accepted```, se o bloco se encontra na 'Chain Tip' a situação pode ser ```Status: Unknown | Processing | Rejected```, do contrário o bloco não foi encontrado.

### Subnet: blockRequest

Cria um novo bloco quando há um bloco candidato em 'Chain Tip' (consulte **_setPreference_**).

### Subnet: connectNode

Guarda a conexão de um Node, esse método pode ser chamado também pela _Rede Principal_.

### Subnet: disconnectNode

Remove um Node da lista de Node conectados.

## Desligamento

Similar a inicialização o desligamento é iniciado pela AvalancheGo, esse procedimento ocorre em ```Subnet::stop``` que é executado quando a AvalancheGo envia ```rpc Shutdown(args)```, o processo é realizado na sequencia:

1. Escrever o conteúdo de 'Chain Head' da memória para a base de dados.
2. Escrever o conteúdo de 'State' da memória para a base de dados.
3. Desligar a base de dados.
4. Desligar o 'HttpServer'.
5. Desligamento total com o método ```Subnet::shutdownServer```.
