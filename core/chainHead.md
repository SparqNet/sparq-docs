# Arquivo core/chainHead.md

## Sobre o ChainHead

A 'Chain Head' ou 'Cabeçalho da Chain' é representada por uma coleção de registros dos blocos, onde transações, contratos, e contas são armazenados nos blocos criados pela própria _Rede Principal_, outros Nodes ou ele mesmo, esses registros não podem ser alterados e estão ali apenas para leitura/consulta.

A gravação de novos registros é feito pelos processos da classe 'Chain Tip' (consulte [chainTip](chainTip.md) para mais detalhes), de forma simplificada recebemos novas transações da rede, conjunto de Nodes ou do próprio Node.

A leitura dos registros é feita em diversos lugares do sistema, pode-se afirmar que o 'Chain Head' junto do 'DB' são o ponto final de todas as operações realizadas no ecossistema.

## Initialização

O 'chainHead' recebe a instância da base de dados ```DBService dbServer```, durante a inicialização do Node em ```Subnet::initialize```, onde é descarregado todos os registros armazenados em uma inicialização anterior, se a base de dados (DB) estiver vazia é criado o ```Block genesis``` com validadores de teste local. 

Após recuperar os dados do DB é inicializado uma rotina de back-up a cada 15 segundos.

## Membros da classe ChainHead

Todas as funções com excessão dos métodos **_push_back_**, **_pop_back_** e **_dumpToDB_** são apenas para leitura ou consulta de transações, ou blocos inteiros.

### Chain Head: push_back 

### Chain Head: pop_back

### Chain Head: exists [overflow]

### Chain Head: getBlock [overflow]

### Chain Head: hasTransaction

### Chain Head: getTransaction

### Chain Head: getBlockFromTx

### Chain Head: latest

### Chain Head: dumpToDb

### Chain Head: periodicSaveToDB
