# Instruções de uso das funções do Utils

Este é um "Rosetta Stone" das funções do Utils e como usá-las para facilitar a vida de quem estiver programando no Sparq.

* [Debugando por meio de print nos logs](#debugando-por-meio-de-print-nos-logs)
* [Conversão de tipos](#conversao-de-tipos)
* [A classe StringContainer](#a-classe-stringcontainer)
* [Manipulação de endereços](#manipulacao-de-enderecos)

## Debugando por meio de print nos logs

Devido ao comportamento da Subnet, depois que a mesma é iniciada (`subnet->start()`), printar detalhes no terminal via `std::cout` não funciona como deveria. Para debugar o programa com a Subnet rodando, é preciso usar as funções `Utils::logToFile()` e `Utils::LogPrint()`.

A função `Utils::logToFile()` loga para o arquivo `log.txt` do Node e pede apenas o string que deve ser logado. Já a função `Utils::LogPrint()` loga para o arquivo `debug.txt`, e pede além do string que deve ser logado, o nome da função (pode se usar o macro `__func__` aqui) e o prefixo do módulo aonde está sendo chamada, que corresponde a um item do namespace Log (e.g. "Log::Subnet = 'Subnet::'"). TODO: confirmar isso

Exemplo de uso:

```
Utils::logToFile("Logging");
// log.txt terá uma linha escrita "Logging"

void testFunc() {
  Utils::LogPrint(Log::db, __func__, "Debugging");
  // debug.txt terá uma linha escrita "DBService::testFunc: Debugging"
}
```

## Conversão de tipos

Em vários pontos do Sparq, bytes puros (raw bytes) são usados ao invés de bytes em formato hexadecimal (hex string), porém ambos são guardados no tipo `std::string`, o que pode ser confuso pra quem decidir printar um "string" e ver um monte de lixo ao invés de um "Hello World".

Caso isso aconteça, jogue a variável que contém os dados na função `Utils::bytesToHex()` caso seja bytes, ou na função `Utils::hexToBytes()` caso seja hex. Exemplo:

```
std::string data = "0x1234567890abcdef";
std::string bytes = Utils::hexToBytes(data);
std::string hex = Utils::bytesToHex(bytes);
std::cout << data << std::endl;
std::cout << bytes << std::endl;
std::cout << hex << std::endl;
```

Para converter inteiros em bytes e vice-versa, deve-se usar as funções `Utils::uintXToBytes()` e `Utils::bytesToUintX()`, respectivamente. "X" é o tamanho do inteiro em bits - pode ser 8, 16, 32, 64, 160 ou 256.

Exemplo de uso:

```
uint64_t timestampOri = 1234567890;
std::string timestampBytes = Utils::uint64ToBytes(timestampOri);
std::string timestampHex = Utils::bytesToHex(timestampBytes);
uint64_t timestampNew = Utils::bytesToUint64(timestampHex);
std::cout << timestampOri << std::endl;
std::cout << timestampBytes << std::endl;
std::cout << timestampHex << std::endl;
std::cout << timestampNew << std::endl;
```

TODO: essas são funções menos comuns, lembrar de puxar a orelha do Ita pra ver se são relevantes de documentar aqui ou se sequer estão sendo usadas

Alternativamente, existem outras funções de apoio:

* `Utils::uintToHex()` funciona com qualquer uint, sem precisar saber o tamanho - e.g. `Utils::uintToHex(32)`.
* `Utils::hexToUint()` funciona da mesma forma, porém somente retorna inteiros de 256 bits.
* Pode-se usar o struct `HexTo` em conjunto com `boost::lexical_cast` para converter um string hex para outros tipos:

```
std::string hex = "0x37285422";
uint256_t bigInt = boost::lexical_cast<HexTo<uint256_t>>(hex);
```

## A classe StringContainer

A classe StringContainer abstrai um string de tamanho fixo (por exemplo, `StringContainer<10>` é um string com **exatamente** 10 caracteres).

Vários tipos que se repetem no programa na realidade são apelidos (aliases) para, ou herdam de um StringContainer de um determinado tamanho.

Caso fique confuso o que é cada coisa, cheque a referência aqui:

* **PrivKey** é um alias para `StringContainer<32>`
* **UncompressedPubkey** é um alias para `StringContainer<65>`
* **CompressedPubkey** é um alias para `StringContainer<33>`
* **Hash** é uma classe que herda `StringContainer<32>`
* **Signature** é uma classe que herda `StringContainer<65>`

Para pegar o conteúdo de qualquer StringContainer, use a função `data()`:

```
StringContainer<10> str = "HelloWorld";
std::cout << str.data() << std::endl;
```

## Manipulação de endereços

Algumas funções do Utils existem para facilitar a manipulação de endereços:

* **toLowercaseAddress()** converte o endereço para um formato todo em caixa baixa (e.g. "0xabcdef...")
* **toUppercaseAddress()** converte o endereço para um formato todo em caixa alta (e.g. "0XABCDEF...")
* **toChecksumAddress()** converte o endereço para um formato de caixa mista, também conhecido como "checksum", de acordo com a especificação do [EIP-55](https://eips.ethereum.org/EIPS/eip-55) (e.g. "0xaBcdEF...")
* **isAddress()** checa se um string é um endereço (tamanho, formato, e também o checksum se estiver nesse formato)
* **checkAddressChecksum()** checa se o checksum do endereço está correto

Exemplo de uso:

```
std::string addOri = "0x1a2b3c4d5e6f7e8d9c0b1a2b3c4d5e6f7e8d9c0b";
std::string addUpp = Utils::toUppercaseAddress(addOri);
std::string addLow = Utils::toLowercaseAddress(addUpp);
std::string addChk = Utils::toChecksumAddress(addOri);
std::string << addOri << std::endl;
std::string << addUpp << std::endl;
std::string << addLow << std::endl;
std::string << addChk << std::endl;
std::string << Utils::isAddress(addChk) << std::endl;
std::string << Utils::checkAddressChecksum(addChk) << std::endl;
```

TODO: não existe bem um "padrão", mas o ideal é que tudo fosse sempre guardado no programa em **caixa baixa**, convertendo para outros tipos on-the-fly somente quando necessário. Ver isso com o Ita

