# 01. Estrutura léxica

Um arquivo NikoSharp usa extensão `.ns` e texto UTF-8. Implementações devem aceitar LF e CRLF. O lexer produz uma sequência de tokens preservando trivia para formatter e LSP.

## Espaços e comentários

Whitespace separa tokens, mas indentação não tem semântica.

```text
line-comment = "--", { char-except-newline }
doc-comment  = "---", { char-except-newline }
```

`---` anexa documentação ao próximo item declarativo. Comentários não alteram gramática.

## Identificadores

```ebnf
identifier-start = "_" | unicode-letter ;
identifier-part  = "_" | unicode-letter | unicode-decimal-digit ;
identifier       = identifier-start, { identifier-part } ;
```

Identificadores são normalizados para NFC antes de comparação semântica. Duas grafias que normalizam para o mesmo valor são o mesmo identificador. Implementações devem diagnosticar identificadores públicos com caracteres visualmente confundíveis quando houver risco de colisão.

Convenções:

- tipos e UDAs usam PascalCase;
- valores, funções, properties e UDMs usam lowerCamelCase;
- `_` inicial é permitido, mas reservado por convenção para itens internos ou descartes locais.

A convenção não muda parsing, mas o formatter/linter pode emitir avisos.

## Keywords hard

```text
and as async await attribute break continue const else enum error false function if in local method modifier nil not operator or partial profile property repeat return routine switch throw throws true try type unsafe using where
```

## Keywords contextuais

```text
get set self static
```

Keywords contextuais são identificadores fora da posição gramatical específica. `self` só é ligado automaticamente dentro de method, property accessor e operator de instância.

## Literais numéricos

```ebnf
digit          = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
hex-digit      = digit | "a".."f" | "A".."F" ;
bin-digit      = "0" | "1" ;
dec-int        = digit, { digit | "_" } ;
hex-int        = "0x", hex-digit, { hex-digit | "_" } ;
bin-int        = "0b", bin-digit, { bin-digit | "_" } ;
exponent       = ("e" | "E"), ["+" | "-"], dec-int ;
number-literal = (dec-int, [".", dec-int], [exponent]) | hex-int | bin-int ;
```

`_` não pode aparecer no começo, fim, imediatamente após prefixo, imediatamente antes/depois de `.`, nem repetido de forma adjacente. Literais inteiros sem tipo são tipados por contexto. Literais reais sem contexto são `number`.

## Strings

Strings usam aspas duplas.

```ebnf
string-literal = '"', { string-part | interpolation }, '"' ;
interpolation  = "{", interpolation-expression, [ ":", format-spec ], "}" ;
```

Escapes válidos:

| Escape | Valor |
|---|---|
| `\\` | barra invertida |
| `\"` | aspas |
| `\0` | NUL |
| `\n` | line feed |
| `\r` | carriage return |
| `\t` | tab |
| `\b` | backspace |
| `\f` | form feed |
| `\u{H...}` | escalar Unicode de 1 a 6 hex digits |
| `{{` | `{` literal em string interpolada |
| `}}` | `}` literal em string interpolada |

Uma interpolação aceita expressão sem assignment, sem `return`, sem `throw`, sem `error`, sem `using` e sem blocos. `format-spec` é texto opaco para o formatter; a semântica de formatação pertence ao consumidor de metadados, não ao lexer.

## Tokens especiais

```text
( ) [ ] { } , . ?. ?[ : :: ; => ->
+ - * / % **
& | ^ ~ << >> >>>
= := += -= *= /= %= **= &= |= ^= <<= >>= >>>=
== != < <= > >= <=>
.. ..= ?? ?
```

O lexer deve escolher o maior token possível.
