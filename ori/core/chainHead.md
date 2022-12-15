# Arquivo core/chainHead.md

## Sobre o objeto ChainHead

A 'Chain Head' ou 'Cabeçalho da Chain' é representada por uma coleção de registros dos blocos, onde transações, contratos, e contas são armazenados nos blocos criados pela própria _Rede Principal_, outros Nodes ou ele mesmo, esses registros não podem ser alterados e estão ali apenas para leitura/consulta.

A gravação de novos registros é feito pelos processos da classe 'Chain Tip' (consulte [chainTip](chainTip.md) para mais detalhes), de forma simplificada recebemos novos blocos (contendo transações), e guardamos eles.

A leitura dos registros é feita em diversos lugares do sistema, pode-se afirmar que o 'Chain Head' junto do 'DB' são o ponto final do historico das operações realizadas no ecossistema.

## Inicialização

O 'chainHead' recebe a instância da base de dados ```DBService dbServer```, durante a inicialização do Node em ```Subnet::initialize```, onde é descarregado todos os registros armazenados em uma inicialização anterior, se a base de dados (DB) estiver vazia é criado o ```Block genesis``` com validadores de teste local.

Após recuperar os dados do DB é inicializado uma rotina de back-up (veja **_periodicSaveToDB_**) a cada 15 segundos.

## Membros da classe ChainHead

Todas as funções com excessão dos métodos **_push_back_**, **_pop_back_** e **_dumpToDB_** são apenas para leitura ou consulta de transações, ou blocos inteiros.

### Chain Head: push_back

Quando um bloco novo é aceito, ele é movido para o fim do 'Chain Head'.

### Chain Head: pop_back

Remove o último bloco aceito pela rede.

### Chain Head: exists [overflow]

Verifica se o bloco existe pelo Hash do bloco ou pela posição dele (```nHeight```).

### Chain Head: getBlock [overflow]

Retorna o bloco (ou se preferir ponteiro do bloco) a partir do Hash do bloco, ou a posição dele (```nHeight```).

### Chain Head: hasTransaction

Verifica se a transação foi processada a partir do Hash da transação.

### Chain Head: getTransaction

Retorna os dados da transação a partir do Hash da transação (utiliza **_hasTransaction_** para verificar se ela foi processada).

### Chain Head: getBlockFromTx

Retorna o bloco que a transação pertence a partir do Hash da transação.

### Chain Head: latest

Retorna o último bloco aprovado pela rede.

### Chain Head: periodicSaveToDB

Rotina de 15 segundos que armazena os dados na base de dados. Essa rotina é inicializada no construtor da classe.

### Chain Head: dumpToDb

É armazenado todo o conteúdo na memória de volta a base de dados (DB), esse método é de uso exclusivo quando o Node inicia o processo de desligamento do mesmo em ```Subnet::stop```.
