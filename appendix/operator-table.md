# Tabela de operadores

| Operador | Categoria | Overload | Observação |
|---|---|---:|---|
| `+ - * / % **` | aritmético | sim | `**` associa à direita |
| unary `+ - *` | prefix | sim | `*` unário é dereferência de tipo definido pelo usuário; aridade (0 vs 1 parâmetro) desambigua de `+`/`-`/`*` binários |
| `== != < <= > >=` | comparação | sim | `!=` pode baixar para `not ==` |
| `<=>` | comparação tripla | sim | retorna `i32` (negativo/zero/positivo); se declarado sem `< <= > >=` individuais, o compilador deriva cada um a partir dele |
| `& | ^ ~ << >> >>>` | bitwise | sim | |
| `[] []=` | index | sim | multi-key por `a[i, j]`, sempre posicional (sem argumento nomeado) |
| `()` | call | sim | callable object |
| `as` | conversão explícita | sim | `expr as T` |
| `as?` | conversão explícita optional | sim | mesmo operador declarado de `as`, retorna `T?` em vez de lançar |
| `TypeName` (sem `as`) | conversão implícita | sim | `operator bool(): bool` etc.; só é chamado automaticamente nos pontos que a spec define como contexto de coerção (hoje: condição de `if`/`repeat where`) |
| `in` | membership | sim | |
| `=>` | operador livre | sim | `operator =>(input: T): R`; sem relação com ponto de entrada do programa (sempre `function main()`); dentro de `SwitchExprArm` continua sendo sintaxe de controle fixa, não overload |
| `.. ..=` | range | não | forma sintática — já é `BinaryOperator`, então `1..10` é `Expr` comum |
| `and or` | lógico | não | short-circuit |
| `??` | nil coalescing | não | short-circuit |
| `. ?. ?[` | acesso | não | |
| `=` | assignment | não | statement |
| `:=` | sinônimo de `=` em `local` | não | statement; sem diferença semântica, formatter normaliza para `=` |

Fora de escopo nesta versão: `++`/`--` (incremento/decremento) — decisão deliberadamente em aberto, não implementados.
