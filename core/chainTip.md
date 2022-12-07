# Arquivo core/chainTip.md

## Sobre o ChainTip

O objetivo do 'Chain Tip' é de realizar operações relacionadas aos blocos recebidos pela _Rede Principal_, nela é contido informações como o melhor bloco, se um bloco está sendo processado, aceitado ou rejeitado.

A 'Chain Tip' não pode aceitar um bloco que já foi recepcionado anteriormente, essa regra se aplica a todas as situações que um bloco pode estar, e para que isso não ocorra é guardado na lista ```cachedBlockStatus <Hash,BlockStatus, SafeHash>``` a situação de todos os blocos recebidos.

## Initialização

Sem construtor, initialização normal em ```Subnet::initialize``` a partir de um ponteiro.

## Membros da classe Chain Tip

Os membros desta classe podem ou não manipular as variáveis ```Hash preferedBlockHash```, ```unordered_map<Hash, Block*, SafeHash> internalChainTip``` e ```unordered_map<Hash, BlockStatus, SafeHash> cachedBlockStatus```, e apenas essas variaveis.

Posteriormente, no [processo de aceitar o bloco](subnet.md), os dados armazenados serão copiados para um novo bloco e adicionados ao 'Chain Head'.

### Chain Tip: setBlockStatus

Altera o estado de um bloco em ```cachedBlockStatus```.

**_Atenção:_** Esse método não é referenciado em nenhum lugar do sistema.

### Chain Tip: getBlockStatus

Se o bloco existir na 'Chain Tip', recupera o estado de um bloco adicionado ao ```cachedBlockStatus```.

### Chain Tip: processBlock

Adiciona o bloco recebido pela _Rede Principal_ a lista ```internalChainTip``` e ```cachedBlockStatus```, para serem processados com as proximas operações da rede.

### Chain Tip: isProcessing

Verifica se o bloco solicitado está sendo processado, se o bloco não for encontrado retorna ```false```, do contrário é consultado o estado do bloco.

Retorna ```false``` se o estado diferir de ```BlockStatus::Processing```.

### Chain Tip: accept

A _Rede Principal_ solicita ao Node que o bloco verificado  seja aceito, se o bloco foi verificado anteriormente e não está sendo processado o mesmo é enviado ao [State -> processNewBlock](state.md), e adicionado ao 'Chain Head'.

### Chain Tip: reject

A _Rede Principal_ solicita ao Node que o bloco verificado seja rejeitado.

### Chain Tip: exists

Verifica se o bloco está presente na 'Chain Tip' ```internalChainTip```.

### Chain Tip: getBlock

Retorna o bloco adicionado na 'Chain Tip' anteriormente, se não existir retorna uma exceção não controlada.

### Chain Tip: getPreference

Retorna o Hash do melhor bloco candidáto selecionado pela rede, se não houver retorna uma exceção não controlada.

### Chain Tip: setPreference

Define o melhor bloco escolhido pela _Rede Principal_.