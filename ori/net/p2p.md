# Comunicações P2P (Peer-To-Peer)

A rede Sparq conta com uma estrutura P2P para comunicações RPC entre nós. As mensagens possuem um determinado formato customizado.

## Estrutura da mensagem

Uma mensagem P2P é formada por uma sequência de bytes, nessa ordem:

* ID do request - **8 bytes** (0x0000000000000000)
* Comando - **2 bytes** (0x0000)
* Dados da mensagem - varia de acordo com o comando, ver abaixo

Os retornos omitem os 2 bytes de comando mas permanecem com o ID. O ID é obrigatório para todas as mensagens, mesmo que não venha a ser usado.

## Lista de comandos

A rede P2P suporta os seguintes functors/comandos:

* **0x0000** - Info
* **0x0001** - SendTransaction
* **0x0002** - SendBulkTransactions
* **0x0003** - RequestBlockByNumber
* **0x0004** - RequestBlockByHash
* **0x0005** - RequestBlockRange
* **0x0006** - NewBestBlock
* **0x0007** - SendValidatorTransaction
* **0x0008** - SendBulkValidatorTransaction
* **0x0009** - RequestValidatorTxs
* **0x000a** - GetConnectedNodes

Abaixo há uma lista com a explicação e os envios e retornos de cada comando. Não esqueça que todas as mensagens sempre começam com o ID e o functor, os dados sempre vem depois.

## 0x0000 - Info

Envia informações sobre o próprio nó e retorna informações sobre o nó conectado.

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| version | 8 | Versão do nó |
| epoch | 8 | Timestamp UNIX em microssegundos |
| nHeight | 8 | Altura do bloco mais recente |
| nHash | 32 | Hash do bloco mais recente |
| connectedNodes | 8 | Número de nós conectados |

Retorno:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| version | 8 | Versão do nó |
| epoch | 8 | Timestamp UNIX em microssegundos |
| nHeight | 8 | Altura do bloco mais recente |
| nHash | 32 | Hash do bloco mais recente |
| connectedNodes | 8 | Número de nós conectados |

## 0x0001 - SendTransaction

Envia uma transação para a rede.

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| tx | N | Bytes raw da transação |

Retorno: não há.

Não é necessário enviar o tamanho da transação, já que é possível saber a mesma com `data.substr(2, data.size())`.

## 0x0002 - SendBulkTransactions

Envia uma série de transações para a rede de uma vez.

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| txCount | 8 | Quantidade de transações |
|
| txSize | 8 | Tamanho da transação |
| tx | N | Bytes raw da transação |
| ... | ... | ... |

Retorno: não há.

A segunda parte do envio é um array, é necessário enviar junto o tamanho de cada transação, assim é possível iterar por cada uma delas dentro da mensagem com a seguinte linha de código:

`for (auto tx : txList) message += Utils::uint64ToBytes(tx.rlpSerialize(true).size()) + tx.rlpSerialize(true)`

## 0x0003 - RequestBlockByNumber

Requisita um bloco pela sua altura (nHeight).

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| nHeight | 8 | Altura do bloco |

Retorno:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| block | N | Bytes raw do bloco |

Caso o bloco não exista, não há retorno.

## 0x0004 - RequestBlockByHash

Requisita um bloco pelo seu hash.

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| hash | 32 | Hash do bloco |

Retorno:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| block | N | Bytes raw do bloco |

Caso o bloco não exista, não há retorno.

## 0x0005 - RequestBlockRange

Requisita um range de blocos de acordo com uma altura inicial e final.

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| startHeight | 8 | Altura do bloco inicial |
| endHeight | 8 | Altura do bloco final |

Retorno:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
|
| blockSize | 8 | Tamanho do bloco |
| blockBytes | N | Bytes raw do bloco |
| ... | ... | ... | ... |

A segunda parte do retorno é um array, é necessário enviar junto o tamanho de cada bloco, assim é possível iterar por cada um deles dentro da mensagem.

**A ordem importa**, do bloco inicial ao final. O bloco final deve sempre ser maior que o bloco inicial.

Caso o range saia do escopo (out of range), não há retorno.

## 0x0006 - NewBestBlock

Propaga um novo "melhor bloco" para a rede (mesma coisa que "bloco mais recente").

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| block | N | Bytes raw do bloco |

Retorno: não há.

## 0x0007 - SendValidatorTransaction

Envia uma transação de um Validator para a rede.

TODO: o envio e retorno NÃO são os mesmos, esse comando vai tomar um Validator de argumento (mandar o Ita preencher isso depois)

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| tx | N | Bytes raw da transação |

Retorno: não há.

Não é necessário enviar o tamanho da transação, já que é possível saber a mesma com `data.substr(2, data.size())`.

## 0x0008 - SendBulkValidatorTransaction

Envia uma série de transações de um Validator para a rede de uma vez.

TODO: o envio e retorno NÃO são os mesmos, esse comando vai tomar um Validator de argumento (mandar o Ita preencher isso depois)

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |
| txCount | 8 | Quantidade de transações |
|
| txSize | 8 | Tamanho da transação |
| tx | N | Bytes raw da transação |
| ... | ... | ... |

Retorno: não há.

A segunda parte do envio é um array, é necessário enviar junto o tamanho de cada transação, assim é possível iterar por cada uma delas dentro da mensagem com a seguinte linha de código:

`for (auto tx : txList) message += Utils::uint64ToBytes(tx.rlpSerialize(true).size()) + tx.rlpSerialize(true)`

## 0x0009 - RequestValidatorTxs

Requisita as transações feitas por Validators.

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |

Retorno:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| txCount | 8 | Quantidade de transações |
|
| txSize | 8 | Tamanho da transação |
| tx | N | Bytes raw da transação |
| ... | ... | ... |

A segunda parte do retorno é um array, é necessário enviar junto o tamanho de cada transação, assim é possível iterar por cada uma delas dentro da mensagem com a seguinte linha de código:

`for (auto tx : txList) message += Utils::uint64ToBytes(tx.rlpSerialize(true).size()) + tx.rlpSerialize(true)`

## 0x000a - GetConnectedNodes

Requisita a lista de nós conectados.

Envio:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| functor | 2 | ID do comando |

Retorno:

| Var | Bytes | Info |
| --- | ---- | ---- |
| id | 8 | ID da mensagem |
| nCount | 8 | Quantidade de nós |
|
| nIp | 4 | IP do nó |
| nPort | 4 | Porta do nó |
| ... | ... | ... |

A segunda parte do retorno é um array, iterando por cada par de IP e porta. Como os tamanhos são fixos é fácil de serializar/deserializar.

