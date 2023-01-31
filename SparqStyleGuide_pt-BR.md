# Sparq Style Guide - PT-BR

Esse é um documento que especifica o estilo de código para projetos feitos no Sparq Labs.

Esse é um documento vivo, já que é somente um guia de referência, não um livro de regras. Enganos podem ocorrer e podem surgir "edge cases".

No momento esse guia somente engloba código em **C++**. Outras linguagens, se consideradas, serão incluídas aqui.

Alguns items podem sugerir configurações específicas para editores de texto (e.g. (Neo)Vim) como referência para outros desenvolvedores.

# C++

Este guia de estilo de código é baseado em uma mistura do estilo pessoal do Supra, o estilo do [Bitcoin](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#coding-style-c) e o estilo do [Google](https://google.github.io/styleguide/cppguide.html) para C++.

* [Tamanho e indentação de linha](#tamanho-e-indentação-de-linha)
* [Estilo dos colchetes, one-liners e operadores ternários](#estilo-dos-colchetes-one-liners-e-operadores-ternários)
* [Formatos de nomes](#formatos-de-nomes)
* [Variáveis e funções](#variáveis-e-funções)
* [Palavra auto](#palavra-auto)
* [Const correctness](#const-correctness)
* [Comentários](#comentários)
* [Organização de arquivos](#organização-de-arquivos)
* [Organização de código](#organização-de-código)
* [Includes](#includes)
* [Definições de Structs e Classes](#definições-de-structs-e-classes)
* [Miscelâneas](#miscelâneas)

## Tamanho e indentação de linha

**Tamanho**: 80 caracteres como limite primário, 100 caracteres como limite absoluto. A métrica é [ergonomia de leitura](https://softwareengineering.stackexchange.com/a/222998).

**Indentação**: Soft tabs (espaços), 2 espaços por nível.

Para usuários do (Neo)Vim, pode-se usar os seguintes configs:

* `set textwidth=80` (linha visual para o limite da linha)
* `set colorcolumn=+1` (deslocamento para a linha visual - não necessário porém útil)
* `set expandtab` (expande hard tabs para espaços)
* `set shiftwidth=2` (número de espaços para cada indentação feita com `<<` ou `>>`)
* `set tabstop=2` (número de espaços inseridos ao pressionar Tab)
* `set softtabstop=2` (número de espaços que o cursor se move ao apertar Tab - [aqui tem uma explicação melhor](https://vi.stackexchange.com/a/28017) mas manter o mesmo número do tabstop já tá bom)

## Estilo dos colchetes, one-liners e operadores ternários

[**1TBS (OTBS)**](https://en.wikipedia.org/wiki/Indentation_style#Variant:_1TBS_(OTBS)), com a exceção de que estruturas de controle (`if/else if/else`, `while/do while`, etc.) com apenas um comando podem tirar os colchetes e virar one-liners ou operadores ternários. Isso é encorajado (porém não obrigatório) inclusive para definições de funções, desde que:

* A linha ainda seja legível e entendível dentro do limite de tamanho
* A função seja simples o suficiente pra caber dentro do limite de tamanho (e.g. funções `inline`)
* A estrutura `if { ... } else { ... }` só tem um comando dentro de cada bloco (no caso de operadores ternários)

Todos os exemplos a seguir estão certos:

```
std::string isNeg(x) {
  if (x < 0) {
    return "negative";
  } else {
    return "positive";
  }
}

std::string isNeg(x) {
  if (x < 0) return "negative"; else return "positive";
}

std::string isNeg(x) { return (x < 0) ? "negative" : "positive"; }
```

## Formatos de nomes

*Variáveis e funções* devem usar **lowerCamelCase** (e.g. `int numApples; std::string getBananas()`).

*Callbacks* devem começar com `on_` e ter o mesmo nome das funções que as chamam (e.g. `on_read(); on_write()`).

*Classes, structs, enums e namespaces* devem usar **UpperCamelCase** first letter (e.g. `class MyClass; struct MyStruct; enum MyEnum; namespace MyNamespace`).

*Globais*, se houverem, devem estar em **caixa alta** e usar **underlines** (e.g. `GLOBAL_TIMER`)

*Aliases* feitos com `using` devem seguir o nome original (e.g. `using json = nlohmann::json; using PrivKey = Hash`) *ou* seus "vizinhos" em questão de conformidade (e.g. `using uint256_t = boost::multiprecision::number<...>` - `uint256_t` fica assim pois outros tipos similares como `uint64_t` que vêm do `std` também são usados).

## Variáveis e funções

**Crie nomes curtos e direto ao ponto onde possível**. Vai depender muito da ocasião, mas no geral use seu discernimento. Contadores dentro de loops, por exemplo, podem ser minúsculos (e.g. `for (int i = 0; i < 10; i++)`), enquanto loops "for each" podem ser um pouco mais verbosos (e.g. `for (std::string item : items)`). O que importa é ser claro e conciso.

**Mantenha o `&` e `*` grudados no tipo, não no nome**. e.g. `int* a; std::string& b`, não `int *a; std::string &b`.

## Palavra auto

**NÃO use "auto" como um tipo a não ser que seja realmente necessário** (e.g. a linha ficou muito grande).

Se tiver que fazer isso, faça mas *por favor* deixe um comentário logo acima para saber qual é o tipo verdadeiro.

## Const correctness

TODO: isso é uma dor na bunda pra analisar mas existem *muitos* `const`s em todo lugar no código, provavelmente a maioria deles é desnecessário mas eu não tenho conhecimento suficiente pra dizer

## Comentários

**Escreva código auto-documentável, mas COMENTE quando necessário, só não exagere**. Use seu discernimento. Comentários devem *complementar* o código, não transformá-lo numa [sopa de letrinhas](https://i.ytimg.com/vi/wLZ01zcwbr0/hqdefault.jpg).

Não esqueça de **seguir o limite de tamanho da linha e a indentação**. Ambos os comentários de linha única e multi-linha podem ser usados, mas use-os com sabedoria.

**Não tenha medo de se expressar**. Ninguém vai te banir por falar palavrão, só não exagere. Às vezes programação é um pé no saco e é *preciso* soltar umas barbaridades junto com outras coisas mais pesadas. Faz parte da catarse.

TODO: eu não sei se vamos integrar o Doxygen, mas se formos, melhor seguir o [estilo de comentários](https://www.doxygen.nl/manual/docblocks.html) dele.

## Organização de arquivos

**O código deve ser separado em headers e sources (`.h/.cpp`)**. Um header pode ter vários sources se preciso, para fins de melhor organizar o código.

**As sources NÃO devem ter includes que não sejam os de seus respectivos headers**. Os headers devem concentrar todos os includes necessários para compilar os sources.

**Sempre use include guards em todos os headers**, baseado no nome do arquivo, da seguinte forma:

```
#ifndef TEST_H
#define TEST_H

// code here...

#endif // TEST_H
```

## Organização de código

Os headers devem ser programados nessa ordem, de cima pra baixo, dentro dos include guards:

* Includes
* Aliases feitos com `using`
* Globals (se houverem)
* Enums
* Namespaces
* Structs
* Classes

**A ordem é recursiva**, ou seja, um namespace seguiria a mesma ordem se tivesse aliases, enums e classes declarados dentro dele.

## Includes

Os includes nos headers devem ser organizados em "blocos", onde cada bloco está em **ordem alfabética**. Para usuários do (Neo)Vim, pode-se organizar uma seleção com `SHIFT+V` e o comando `:sort`.

Os blocos são divididos por espaços e devem estar nessa ordem, de cima pra baixo:

* Includes do **C++ STD** (e.g. `#include <string>`)
* Includes **externos** (e.g. Boost, OpenSSL, Ethash, etc. - pode-se separar esses com espaços também),
* Includes de **arquivos locais** (e.g. `#include "../include.h"`)

**A exceção é caso alguns arquivos precisem ser incluídos em uma outra ordem específica**. Por exemplo, por algum motivo um certo header tem que ser incluído antes de todos os outros, senão não compila. Se isso acontecer, por favor deixe um comentário do lado pra que ninguém escorregue no quiabo.

## Definições de Structs e Classes

Definições dentro de structs e classes devem ser feitos começando pelos membros **privados**, depois **protegidos** e por fim **públicos**.

Pra cada um desses escopos, segue-se a seguinte ordem de cima pra baixo:

* Variáveis/atributos
* Construtor(es)
* Getters e setters (na mesma ordem que variáveis/atributos)
* Funções/métodos normais
* Funções/métodos "avançados" (e.g. operators, overrrides, etc.)

**Agrupe definições similares**. e.g. `uint256ToBytes()` até `uint8ToBytes()`.

## Miscelâneas

**Mantenha itens dentro de parênteses separados, sem espaços extras dos lados**. Faça `(isso, eisso)`, não `( isso,eisso )`.

**Tente não deixar espaços no fim das linhas**. Para usuários do (Neo)Vim pode-se setar o seguinte comando: `:command ClearTrailing %s/\s\+$//e`, e usá-lo durante a edição: `:ClearTrailing` (isso é uma mão na roda, confia).

**`i++` é preferível ao invés de `++i`**, a não ser que a lógica obrigue a usar o outro por algum motivo.

**Use `nullptr` para ponteiros, `NULL` para o resto**.
