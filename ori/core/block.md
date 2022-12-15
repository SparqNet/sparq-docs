# Blocos

Blocos na rede Sparq são abstraídos pela classe **Block**. Essa classe somente possui a estrutura e os dados de um bloco, não faz nenhum tipo de operação de validação ou verificação nos mesmos.

## Estrutura de um bloco

Conceitualmente, um bloco possui a seguinte estrutura:

* Assinatura do Validator
* Header:
  * Hash do bloco anterior (= um hash do header como um todo, assinado pelo Validator)
  * "Randomness" (TODO: cachola do Ita)
  * Árvore Merkle do Validator (para verificação da integridade das transações do Validator)
  * Árvore Merkle do bloco (para verificação da integridade das transações do bloco)
  * Timestamp UNIX do bloco, em nanosegundos
  * Altura do bloco (nHeight)
* Conteúdo:
  * Número de transações do Validator
  * Número de transações do bloco
  * Lista de transações do Validator
  * Lista de transações do bloco

Na prática, um bloco é simplesmente um string de bytes serializado, transmitido pela rede e guardado no blockchain, assim cabe à lógica do programa parsear tais detalhes.

A estrutura acima pode ser interpretada de forma mais "pura" da seguinte forma:

```
65 bytes - Assinatura do Validator
Header:
  32 bytes - Hash do bloco anterior
  32 bytes - "Randomness"
  32 bytes - Árvore Merkle do Validator
  32 bytes - Árvore Merkle do bloco
  8 bytes - Timestamp
  8 bytes - Altura do bloco
Conteúdo:
  8 bytes - número de transações do Validator
  8 bytes - número de transações do bloco
  8 bytes - offset (começo) do array de transações do validator
  8 bytes - offset (começo) do array de transações do bloco
  [
    4 bytes - tamanho da transação do validator
    N bytes - transação do validator
    ...
  ]
  [
    4 bytes - tamanho da transação do bloco
    N bytes - transação do bloco
    ...
  ]
```

Blocos transmitidos na rede devem *sempre* conter a assinatura do Validator, que faz parte do bloco mas está *fora* da estrutura do mesmo. Isso é intencional, para que o header do bloco possa ser hasheado e assinado sem a interferência da própria assinatura.

## Análise de um bloco raw

Sendo assim, um bloco "puro" (raw) teria o seguinte formato:

`5c37d504e9415c3b75afaa3ad24484382274bba31f10dcd268e554785d5b807500000181810eb6507a8b54dfbfe9f21d00000001000000aff8ad81be850c92a69c0082e18c94d586e7f844cea2f87f50152665bcbc2c279d8d7080b844a9059cbb00000000000000000000000026548521f99d0709f615aa0f766a7df60f99250b00000000000000000000000000000000000000000000002086ac351052600000830150f7a07e16328b7f3823abeb13d0cab11cdffaf967c9b2eaf3757c42606d6f2ecd8ce6a040684c94b289cdda22822e5cb374ea374d7a3ba581a9014faf35b19e5345ab92`

Onde:

TODO: eu não sei qual dos dois exemplos está correto, se um byte é contado de dois em dois ou um em um, ou se o bloco de exemplo tá correto pra começo de conversa porque as contas não batem. Deixei os dois aqui pra análise.

=== Exemplo 1 ===

* Assinatura do Validator = 5c37d504e9415c3b75afaa3ad24484382274bba31f10dcd268e554785d5b80750
* Hash do bloco anterior = 0000181810eb6507a8b54dfbfe9f21d0
* "Randomness" = 0000001000000aff8ad81be850c92a69
* Árvore Merkle do Validator = c0082e18c94d586e7f844cea2f87f501
* Árvore Merkle do bloco = 52665bcbc2c279d8d7080b844a9059cb
* Timestamp = b0000000
* Altura do bloco = 00000000
* Número de transações do Validator = 00000000
* Número de transações do bloco = 02654852
* Offset de transaçẽos do Validator = 1f99d070
* Offset de transações do bloco = 9f615aa0

f766a7df60f99250b00000000000000000000000000000000000000000000002086ac351052600000830150f7a07e16328b7f3823abeb13d0cab11cdffaf967c9b2eaf3757c42606d6f2ecd8ce6a040684c94b289cdda22822e5cb374ea374d7a3ba581a9014faf35b19e5345ab92

=== Exemplo 2 ===

* Assinatura do Validator = 5c37d504e9415c3b75afaa3ad24484382274bba31f10dcd268e554785d5b807500000181810eb6507a8b54dfbfe9f21d00000001000000aff8ad81be850c92a69c0082e18c94d586e7f844cea2f87f50152665bcbc2c279d8d7080b844a9059cbb00000000000000000000000026548521f99d0709f615aa0f766a7df60f99250b00000000000000000000000000000000000000000000002086ac351052600000830150f7a07e16328b7f3823abeb13d0cab11cdffaf967c9b2eaf3757c42606d6f2ecd8ce6a040684c94b289cdda22822e5cb374ea374d7a3ba581a9014faf35b19e5345ab92
* Hash do bloco anterior = 0082e18c94d586e7f844cea2f87f50152665bcbc2c279d8d7080b844a9059cbb
* "Randomness" = 00000000000000000000000026548521f99d0709f615aa0f766a7df60f99250b
* Árvore Merkle do Validator = 00000000000000000000000000000000000000000000002086ac351052600000
* Árvore Merkle do bloco = 830150f7a07e16328b7f3823abeb13d0cab11cdffaf967c9b2eaf3757c42606d
* Timestamp = 6f2ecd8ce6a04068
* Altura do bloco = 4c94b289cdda2282

2e5cb374ea374d7a3ba581a9014faf35b19e5345ab92

