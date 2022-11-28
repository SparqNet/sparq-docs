# Arquivo core/subnet.md

### Objetivo do Objeto Subnet

**_TODO_**

### Sobre o objeto Subnet

Conforme exemplificado anteriormente no [arquivo ponto de entrada](../src/main.md) a finalidade da classe Subnet é encapsular e redirecionar os módulos dos Nodes onde a Subnet atuará como um intermediador, a Subnet (Subnatooor) também atuará como uma engine de execução para aplicações de smart contracts em [Solidity](https://github.com/ethereum/solidity) com integração de múltiplas vias da Rede Subnets Sparq.

Atualmente a única integração de comunicação dos Nodes é dependente das VM (Virtual Machines, mais em [AvalancheGo](https://github.com/ava-labs/avalanchego/tree/master/vms)) da [AvaLabs](https://www.avalabs.org/), e está sendo desenvolvido uma comunicação de Ponta-A-Ponta entre as Subnets (Subnatooor) e também um modelo de requisições para que outras Subnets realizem a conexão e comuniquem entre sí.

### O Papel do AvalancheGO Daemon

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

**_Nota:_**  Extrair mais informações do Itamar.

### Por que AvalancheGo?

Um dos pré-requisitos a serem satisfeitos no início do projeto foi a implementação do próprio AvalancheGo, grande feito que surgiu impacto no escopo do projeto más também beneficiou a imagem pública do projeto.
