# Cono funciona o blockchain do Sparq

A estrutura do blockchain do Sparq é composta pela interação entre os seguintes componentes:

* Transação (**Tx::Base** em `utils/transaction`)
* Bloco (**Block** em `core/block`)
* Nó Validador (**Validator** em `core/blockmanager`)
* Gerenciador de Blocos (**BlockManager** em `core/blockmanager`)
* Mempool (**ChainTip** em `core/chainTip`)
* Blockchain (**ChainHead** em `core/chainHead`)

TODO: Nota do redator (Supra): os nomes são confusos pra quem tá vindo de fora, sinceramente eu buscaria renomear o quanto antes pra ficar consistente:
- **Tx::Base** podia virar só **Tx**, o bagulho é literalmente uma classe dentro de um namespace que não tem NADA a não ser essa uma classe
- **ChainHead** podia virar só **Chain** ou **Blockchain**
- **ChainTip** podia virar **Mempool** (porque é literalmente o que a bagaça é)

## Transação (Tx::Base)

A classe Tx::Base é uma abstração da estrutura de uma transação. A lógica foi adaptada da biblioteca Aleth do Ethereum.

Uma transação contém os seguintes dados:

* **to** - Endereço de destino
* **from** (opcional) - Endereço de remessa
* **value** - Valor da transação em sua menor unidade
  * e.g. "satoshi", "wei", etc. - "100000000" em satoshi seria 1.0 BTC, "5000000000" em wei seria 0.000000005 ETH (ou 5 gwei)
* **data** (opcional) - Campo de dados arbitrários, geralmente usado em contratos
* **chainId** - Identificação única do blockchain onde a transação é feita
  * e.g. "43114" = Avalanche C-Chain, "43113" = Avalanche Fuji Testnet
* **nonce** - Número da transação feita pelo endereço de remessa (nonce - começa com 0)
  * Começa sempre no 0, isso quer dizer que um nonce "4" por exemplo indicaria que essa é a *quinta* transação feita por tal endereço
* **gas** (também chamado de "Gas Limit") - limite máximo de unidades de gas que a transação irá gastar, em Wei (e.g. "21000")
  * Caso a transação gaste mais do que isso, ela automaticamente falha, o valor original da transação se mantém, mas o que já foi gasto de gas é perdido
* **gasPrice** - valor pago para cada unidade de gas, em Gwei (e.g. "15" = 15000000000 Wei)
  * O valor total da taxa da transação é calculado como (gas * gasPrice) - e.g. 21000 * 15000000000 = 0.000315 ETH
* Assinatura ECDSA (Elliptic Curva Digital Signature Algorithm) para validação da integridade da transação, dividida em 3 partes:
  * **v** - ID de recuperação (1 byte hex)
  * **r** - primeira metade da assinatura ECDSA (32 bytes hex)
  * **s** - segunda metade da assinatura ECDSA (32 bytes hex)

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

## Nó Validador (Validator)

A classe Validator é uma abstração de um nó validador. Um Validator é um nó que valida os blocos e suas transações vindas da rede.

TODO: isso aqui também tá mais na cachola do Ita, falta documentação a mais sobre o que/como o Validator faz

## Gerenciador de Blocos (BlockManager)

A classe BlockManager gerencia a criação, congestão e validação de blocos.

O processo é feito por meio de uma lista interna de Validators, dos quais um é escolhido para criar o bloco e os outros validam o mesmo por meio de assinaturas.

TODO: sinceramente isso aqui tá mais na cachola do Ita, além de que algumas coisas não estão muito claras:

- "BlockManager is also considered a contract, but remains part of the core protocol of Sparq"- ???
- A escolha dos Validators é realmente aleatória? O código não diz muito
- `processBlock()` tá implementado no ChainTip mas não aqui (code cruft?????)

## Blockchain (ChainHead)

A classe ChainHead é uma abstração do blockchain propriamente dito. Mantém blocos aprovados e validados.

Ao iniciar a Subnet, o ChainHead carrega na memória um histórico de até 1000 transações mais recentes de uma database interna. Caso não hajam blocos (ou seja, o blockchain acabou de ser implantado e iniciado pela primeira vez), um bloco genesis é automaticamente criado e carregado na memória.

Ao atingir o limite de 1000 blocos recentes na memória, ou depois de certo tempo, os blocos antigos são periodicamente salvos na database. Isso mantém o blockchain leve no uso de memória e extremamente responsivo.

A classe tambem possui vários `std::unordered_map`s para consulta e cache de blocos e transações.

TODO: existem escolhas obscuras de design que não foram bem explicadas como:

- A lista interna dos blocos é um `std::deque` (uma fila que dá pra inserir e remover das duas pontas) - eu não lembro o por que o Ita fez assim

## Mempool (ChainTip)

A classe ChainTip é uma abstração do que seria a mempool. Mantém blocos novos e/ou em processamento, aprovando-os ou rejeitando-os de acordo com as regras de consenso.

Ao chegar num consenso sobre um bloco, ele é automaticamente apagado da mempool, migrando antes para o ChainHead se tiver sido aprovado.

