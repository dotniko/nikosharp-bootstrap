# 05. Expressões e operadores

## Precedência

Maior para menor:

| Nível | Operadores | Associatividade |
|---:|---|---|
| 1 | primários, literais, `()` | n/a |
| 2 | `.`, `?.`, call `()`, index `[]`, safe index `?[` | esquerda |
| 3 | prefix `+`, `-`, `*` (deref), `not`, `~`, `try`, `await` | direita |
| 4 | `**` | direita |
| 5 | `as`, `as?` | esquerda |
| 6 | `*`, `/`, `%` | esquerda |
| 7 | `+`, `-` | esquerda |
| 8 | `<<`, `>>`, `>>>` | esquerda |
| 9 | `&` | esquerda |
| 10 | `^` | esquerda |
| 11 | `|` | esquerda |
| 12 | `..`, `..=` | não encadeável sem parênteses |
| 13 | `<`, `<=`, `>`, `>=`, `<=>` | não encadeável sem `and` |
| 14 | `in` | esquerda |
| 15 | `==`, `!=` | não encadeável sem `and` |
| 16 | `and` | esquerda, short-circuit |
| 17 | `or` | esquerda, short-circuit |
| 18 | `??` | direita |
| 19 | `=>` | direita |

Assignment não é expressão. Prefixo `*` (dereferência) e binário `*` (multiplicação) são o mesmo token — a aridade do `operator *` declarado (zero ou um parâmetro) desambigua, exatamente como já ocorre com `+`/`-` unário e binário.

`=>` no nível 19 só se aplica fora de `SwitchExprArm`/`SwitchStmt`: dentro de um braço de `switch`, `=>` continua sendo o separador de sintaxe de controle fixo (não overloadable naquela posição). Em qualquer outra posição de expressão, `=>` é só mais um operador binário livre — não tem relação com ponto de entrada do programa, que é sempre `function main()` (`artifacts/01-ns-files.md`).

## Operadores overloadable

Overloadable:

```text
+ - * / % **
== != < <= > >= <=>
& | ^ ~ << >> >>>
[] []= () as in =>
```

Conversão implícita — declarada com o nome do tipo alvo sozinho, sem `as` (`operator bool(): bool`, `operator string(): string`, etc.) — também é uma forma de operador declarável (`language/07-generics-profiles-overload.md`), distinta da conversão explícita `as T`.

Não overloadable:

```text
. ?. ?? = := ; :: and or not try await throw error return if repeat switch using
```

`not`, `and` e `or` são semântica de controle e não chamam overload.

`++`/`--` (incremento/decremento) não existem na linguagem nesta versão bootstrap — decisão em aberto, não uma omissão silenciosa; use `x += 1`/`x -= 1`.

`operator <=>(other: T): i32` declara comparação tripla (negativo, zero, positivo). Quando um tipo declara `<=>` mas não declara individualmente `<`, `<=`, `>` ou `>=`, o compilador deriva cada um comparando o resultado de `<=>` com zero — a mesma lógica já usada para derivar `!=` a partir de `==`.

## Chamada e argumentos nomeados

Chamadas em expressão sempre usam parênteses. Argumentos nomeados podem aparecer depois de posicionais, mas depois do primeiro nomeado todos os seguintes devem ser nomeados.

```nikosharp
resize(width: 800, height: 600)
```

## Statement call

Em statement position, uma chamada com exatamente um argumento pode omitir parênteses:

```nikosharp
print "hello"
```

Essa forma nunca é válida dentro de expressão.

## Conversão

`expr as T` realiza conversão explícita ou operador declarado. `expr as? T` tenta converter e retorna `T?`.

Conversão **implícita** (sem `as` no call site) só ocorre nos pontos que esta spec define explicitamente como contexto de coerção automática — hoje, apenas condição de `if`/`repeat ... where` aceitando um tipo com `operator bool()` (`language/03-types.md`). Declarar `operator TypeName()` para qualquer outro `TypeName` não cria coerção automática em nenhum outro lugar da linguagem; fora do contexto de condição, o operador só é alcançável via `as TypeName` explícito. Isso é deliberado (design goal de "contratos explícitos, não mágica dinâmica").

## `try` e `await`

`try await f()` é a forma canônica. `await try f()` é aceito pelo parser, mas normalizado e formatado para `try await f()`.

## Construção

```nikosharp
Point { x = 1, y = 2 }
```

Campos obrigatórios sem default precisam ser informados. Campos desconhecidos são erro.

Um literal de chave-valor sem tipo explícito antes de `{` (`{ a = 1 }`) exige contexto de tipo alvo (binding tipado, parâmetro ou campo) para ser resolvido; sem esse contexto é erro de tipo ambíguo. Esta spec não define qual tipo concreto de biblioteca resulta desse literal — isso é responsabilidade da stdlib, fora do escopo deste documento.

## Interpolação

```nikosharp
"x = {value}"
"price = {price:0.00}"
```

O `format-spec` depois de `:` é texto opaco. A expressão antes de `:` segue a gramática restrita de interpolação.
