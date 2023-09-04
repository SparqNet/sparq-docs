# Sparq Style Guide - EN-US

This is a document that specifies the coding style for projects made in Sparq Labs.

This is a living document, as it is merely a reference guide, not a rule book. Mistakes can happen and edge cases may be found.

At the moment this guide is only for **C++** code. Other languages, if considered, will be included here.

Some items may suggest specific configs for text editors (e.g. (Neo)Vim) as reference for other developers.

# C++

This coding style is based on a mix of Supra's personal style guide, [Bitcoin's](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#coding-style-c) style guide and [Google's](https://google.github.io/styleguide/cppguide.html) style guide for C++.

* [Line width and indentation](#line-width-and-indentation)
* [Brace style, one-liners and ternary operators](#brace-style-one-liners-and-ternary-operators)
* [Name casing](#name-casing)
* [Variables and functions](#variables-and-functions)
* [Auto keyword](#auto-keyword)
* [Const correctness](#const-correctness)
* [Comments](#comments)
* [File organization](#file-organization)
* [Code organization](#code-organization)
* [Includes](#includes)
* [Struct and Class definitions](#struct-and-class-definitions)
* [Miscellaneous](#miscellaneous)

## Line width and indentation

**Width**: 80 characters as a soft limit, 100 characters as a hard limit. The metric is [reading ergonomics](https://softwareengineering.stackexchange.com/a/222998).

**Indentation**: Soft tabs (spaces), 2 spaces per level.

For (Neo)Vim users, you can use the following configs:

* `set textwidth=80` (visual aid for the width limit)
* `set colorcolumn=+1` (offset for visual aid - not required but handy)
* `set expandtab` (expands hard tabs to spaces)
* `set shiftwidth=2` (number of spaces for each indent made with `<<` or `>>`)
* `set tabstop=2` (number of spaces inserted when pressing Tab)
* `set softtabstop=2` (number of spaces the cursor will move after pressing Tab - [here's a better explanation](https://vi.stackexchange.com/a/28017) but keeping it the same as tabstop is fine)

## Brace style, one-liners and ternary operators

[**1TBS (OTBS)**](https://en.wikipedia.org/wiki/Indentation_style#Variant:_1TBS_(OTBS)), with the exception that control statements (`if/else if/else`, `while/do while`, etc.) with a single statement are allowed to take off the braces and become one-liners or ternary operators. Those are encouraged (not forced) even for function definitions, as long as:

* The line is still readable and understandable under the line width limit
* The function is simple enough to fit the line width limit (e.g. `inline` functions)
* You have a single `if { ... } else { ... }` statement (in case of ternary operators)

All the following examples are OK:

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

## Name casing

*Variables and functions* should use **lowerCamelCase** (e.g. `int numApples; std::string getBananas()`).

*Callbacks* should start with `on_` and have the same name as their caller functions (e.g. `on_read(); on_write()`).

*Classes, structs, enums and namespaces* should use **UpperCamelCase** first letter (e.g. `class MyClass; struct MyStruct; enum MyEnum; namespace MyNamespace`).

*Globals*, if there should be any, should be **all caps** and use **underlines** (e.g. `GLOBAL_TIMER`)

*Aliases* made with `using` should follow along with the original name (e.g. `using json = nlohmann::json; using PrivKey = Hash`) *or* its "peers" for conformity (e.g. `using uint256_t = boost::multiprecision::number<...>` - `uint256_t` is kept that way because other similar types like `uint64_t` that come from `std` are also being used).

## Variables and functions

**Keep names short and to the point where possible**. It'll heavily depend on the occasion, but overall just use your discernment. Counters inside loops, for example, can be pretty short (e.g. `for (int i = 0; i < 10; i++)`), while "for each" loops can be a bit more verbose (e.g. `for (std::string item : items)`). What matters is to be clear and concise.

**Keep `&` and `*` glued to the type, not the name**. e.g. `int* a; std::string& b`, not `int *a; std::string &b`.

**Class member variables should always contain a suffixed `_`**. e.g. `std::string name_; uint256_t value_;`. The reason is to avoid hard-to-detect cases like variable shadowing (SanitizerAddress).

## Auto keyword

**DO NOT use "auto" as a type unless you have no other way out** (e.g. the line gets too big).

If you need to, do it but *please* leave a comment above it so the real type is known.

## Comments

**Write self-documenting code, but DO comment it when required, just don't overdo it**. Use your discernment. Comments should *complement* code, not turn it into a [word soup](https://i.ytimg.com/vi/wLZ01zcwbr0/hqdefault.jpg).

Don't forget to **follow line width and indentation**. Both single-line and multi-line comments are welcome, but use them wisely.

**Don't be afraid to express yourself**. No one will ban you for saying bad words, jsut don't overdo it. Sometimes coding is a PITA and we just *have* to emphasize the A with some other flavors of swearing on top. It helps with catharsis.

When commenting documentation for Doxygen, use its [commenting style](https://www.doxygen.nl/manual/docblocks.html).

## File organization

**Code should be split into headers and sources (`.h/.cpp`)**. One header can have several sources if required, for ease of code organizing.

**Sources should NOT have includes other than their respective headers**. Headers should concentrate all required includes for sources to compile.

**Always use include guards for every header**, based on the file's name, like this:

```
#ifndef TEST_H
#define TEST_H

// code here...

#endif // TEST_H
```

## Code organization

Headers should be coded in this order, top-down, inside the include guards:

* Includes
* Aliases made with `using`
* Globals (if any)
* Enums
* Namespaces
* Structs
* Classes

**The order is recursive**, e.g. a namespace would follow the same order if it had aliases, enums and classes declared inside it.

## Includes

Includes in headers should be organized in "blocks", where each block is in **alphabetical order**. For (Neo)Vim users, you can sort a selection with `SHIFT+V` and the command `:sort`.

Blocks are divided by spaces and should be in this order, top-down:

* **C++ STD** includes (e.g. `#include <string>`)
* **External** includes (e.g. Boost, OpenSSL, Ethash, etc. - you can further separate those with spaces as well),
* **Local file** includes (e.g. `#include "../include.h"`)

**The exception to this is if some files have to be included in some other specific order**. For example, somehow a header has to be included before all the others, otherwise it won't compile. If that happens, please leave a comment beside it so no one messes that up.

## Struct and Class definitions

Definitions inside structs and classes should be done starting from **private**, then **protected**, then **public**.

For each one of those scopes, this order should be followed top-down:

* Variables/attributes
* Constructor(s)
* Getters and setters (in the same order as variables/attributes)
* Normal functions/methods
* "Advanced" functions/methods (e.g. operators, overrides, etc.)

**Keep similar definitions together**. e.g. `uint256ToBytes()` all the way down to `uint8ToBytes()`.

## Miscellaneous

**Keep items inside parenthesis separated, with no extra spaces on the sides**. Do `(this, andthis)`, not `( this,andthis )`.

**Try to not leave trailing whitespaces**. For (Neo)Vim users you can set this custom command: `:command ClearTrailing %s/\s\+$//e`, and run it while editing: `:ClearTrailing` (this is a lifesaver, trust me).

**`i++` is preferred over `++i`**, unless the logic requires it for some reason.

**Use `nullptr` for pointers, `NULL` for everything else**.
