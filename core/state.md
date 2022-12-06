# Arquivo core/state.md

## Sobre o objeto State

O objetivo do objeto 'State' é de representar o estado das contas, transações, balanço nativo, balanço de tokens e contratos. Sendo possível apenas a alteração dos dados armazenados a travez do processo de criação de blocos, seja por conta própria ou recebido pela _Rede Principal_ (Veja [**_core/subnet.md_**](subnet.md) para os casos de uso).

## Initialização

O 'State' recebe tanto o acesso da base de dados (DB) quanto um ```grpcClient``` para comunicar com o AvalancheGo, por exemplo; Comunicar ao AvalancheGo que o Node não esta mais processando um bloco.

## Membros da classe State

Todos os membros desta classe diz respeito da atualização da própria Chain que o Node representa e sua sincronização ao ecosistema de "Subnets" conectados a uma _Rede Principal_ e outras pontas de Subnet (Subnetooor).

### State: getNativeBalance

Recupera o saldo nativo da conta fornecida.

### State: getNativeNonce

Recupera o número de transação da conta fornecida.

### State: validateNewBlock

Verifica se a assinatura do bloco é válida, se sua ordem/sequencia está correta com o 'Chain Head', se foi adicionada na 'memory pool', e se não se encontra em 'Chain Tip'.

### State: processNewBlock

Processa o novo bloco e atualiza o 'State', esse método é chamado apenas pela 'Chain Tip' quando a rede solicita que um bloco será aceito (Consulte ```Subnet::acceptBlock``` [*aqui*]("subnet.md#L269")).

### State: createNewBlock

Cria um novo bloco a partir do melhor candidato em 'Chain Tip' quando as seguintes condições são satisfeitas:

1. Deverá existir um candidato em 'Chain Tip' selecionado pela rede.
2. Deverá existir o Hash do bloco em 'Chain Tip'.
3. Deverá existir um bloco correspondente ao Hash em 'Chain Head' já na lista de 'cachedBlocks'.

Se qualquer condição acima falhar será retornado ```nullptr```, e a operação será cancelada.

### State: validateTransactionForBlock

Faz a validação das seguintes condições em uma transação:

1. Se a transação já foi validada.
2. Se a transação se encontra na 'memory pool'.
3. Se a conta existe ou se há ~~qualquer~~ saldo para realizar a transação.
4. Se a conta têm saldo suficiente para realizar a transação e se o Nonce dele é valido.

Se qualquer condição falhar é retornado ```falso```.

**_Atenção_**: Essa operação não altera o State da block-chain.

### State: validateTransactionForRPC

Realiza a mesma validação que **_validateTransactionForBlock_**, porém é utilizado exclusivamente Nodes do ecosistema **Sparq Network**.

**_Atenção_**: Essa operação não altera o State da block-chain.