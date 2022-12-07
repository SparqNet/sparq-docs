# Cono funciona o blockchain do Sparq

A estrutura do blockchain do Sparq é composta pela interação entre os seguintes componentes:

* [Transação](../utils/transaction.md) (**Tx::Base** em `utils/transaction`)
* [Bloco](block.md) (**Block** em `core/block`)
* Nó Validador (**Validator** em `core/blockmanager`)
* Gerenciador de Blocos (**BlockManager** em `core/blockmanager`)
* [Mempool](chainTip.md) (**ChainTip** em `core/chainTip`)
* Blockchain (**ChainHead** em `core/chainHead`)

TODO: Nota do redator (Supra): os nomes são confusos pra quem tá vindo de fora, sinceramente eu buscaria renomear o quanto antes pra ficar consistente:
- **Tx::Base** podia virar só **Tx**, o bagulho é literalmente uma classe dentro de um namespace que não tem NADA a não ser essa uma classe
- **ChainHead** podia virar só **Chain** ou **Blockchain**
- **ChainTip** podia virar **Mempool** (porque é literalmente o que a bagaça é)

TODO: eliminar os tópicos abaixo conforme forem tomando forma em arquivos separados

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

