# Cono funciona o blockchain do Sparq

A estrutura do blockchain do Sparq é composta pela interação entre os seguintes componentes:

* Transação (**Tx::Base** - `utils/transaction`)
* Bloco (**Block** - `core/block`)
* Blockchain (**ChainHead** - `core/chainHead`)
* Mempool (**ChainTip** - `core/chainTip`)
* **BlockManager** (`core/blockmanager`) (TODO: cachola do Ita)
* **Validator** (`core/blockmanager`)

Nota do redator (Supra): os nomes são confusos pra quem tá vindo de fora, sinceramente eu buscaria renomear o quanto antes pra ficar consistente:
- **Tx::Base** podia virar só **Tx**, o bagulho é literalmente uma classe dentro de um namespace que não tem NADA a não ser essa uma classe
- **ChainHead** podia virar só **Chain** ou **Blockchain**
- **ChainTip** podia virar **Mempool** (porque é literalmente o que a bagaça é)

## Transação (Tx::Base)

A classe Tx::Base é uma abstração da estrutura de uma transação. A lógica foi adaptada da biblioteca Aleth do Ethereum.

Uma transação contém os seguintes dados:

* Endereço de destino (também chamado de "to")
* (Opcional) Endereço de remessa (também chamado de "from")
* Valor da transação em sua menor unidade (e.g. "satoshi", "wei", etc. - "100000000" em satoshi seria 1.0 BTC)
* (Opcional) Campo de dados arbitrários (data)
* Identificação única do blockchain onde a transação é feita (também chamado de "chainId")
* Número da transação feita pelo endereço de remessa (nonce - começa com 0)
  * No contexto de uma transação, um nonce "4" por exemplo indicaria que essa é a quinta transação feita por tal endereço
* Gas Limit (ou somente "Gas" - limite máximo de unidades de gas que a transação irá gastar, em Wei - e.g. "21000")
  * Caso a transação gaste mais do que isso, ela automaticamente falha, o valor original da transação se mantém, mas o que já foi gasto de gas é perdido
* Gas Price (valor pago para cada unidade de gas, em Gwei - "e.g. 15 = 15000000000 Wei")
  * O valor total da taxa da transação é calculado como (gas * gas price) - e.g. 21000 * 15000000000 = 0.000315 ETH
* Assinatura ECDSA (Elliptic Curva Digital Signature Algorithm) para validação da integridade da transação
  * Dividida em 3 partes: **v** (ID de recuperação - 1 byte), **r** (primeira metade - 32 bytes) e **s** (segunda metade - 32 bytes)
  * Bytes são em formato hexadecimal, logo "32 bytes" seriam "64 caracteres" se convertidos

TODO: conferir com o Ita se os opcionais e o cálculo do gas estão corretos

## Bloco (Block)

A classe Block é uma abstração da estrutura de um bloco. Somente contém os dados do bloco, não faz nenhum tipo de operação de validação ou verificação.

Um bloco contém os seguintes dados:

* Assinatura do Validator
* Hash do bloco anterior
* "Randomness" (TODO: cachola do Ita)
* Árvores Merkle para verificação da integridade das transações do bloco *e* das transações do Validator
  * (TODO: Validator não "faz" transações, apenas as valida - ver com o Ita depois sobre os termos usados)
* Timestamp UNIX do bloco, em nanosegundos
* Altura do bloco (nHeight)
* Número de transações inclusas no bloco *e* número de transações do Validator
* Lista de transações inclusas no bloco *e* lista de transações do Validator

## Blockchain (ChainHead)

A classe ChainHead é uma abstração do blockchain propriamente dito.

O blockchain do Sparq mantém um histórico de até 1000 blocos mais recentes na memória, enquanto os blocos mais antigos vão sendo periodicamente salvos numa database interna.

TODO: talvez melhorar os detalhes aqui, existem escolhas obscuras de design que não foram bem explicadas como:

- A lista interna dos blocos é um `std::deque` (uma fila que dá pra inserir e remover das duas pontas) - eu não lembro o por que o Ita fez assim

## Mempool (ChainTip)

A classe ChainTip é uma abstração do que seria a mempool.

TODO: continuar isso aqui depois
