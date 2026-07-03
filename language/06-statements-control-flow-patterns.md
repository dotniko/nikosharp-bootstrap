# 06. Statements, controle de fluxo e patterns

## Blocos

Blocos são abertos pela declaração ou statement e fechados por `;`.

```nikosharp
if ready
    run()
;
```

Statements comuns não terminam com `;`.

## If

Condição aceita `bool`, `T?` ou tipo que declare `operator bool(): bool` (conversão implícita, chamada automaticamente só neste contexto — `language/03-types.md`, `language/07-generics-profiles-overload.md`).

```nikosharp
if user
    return user.name
else
    return "anonymous"
;
```

## Repeat

`repeat` cobre contagem, iteração e filtro:

```nikosharp
repeat 10 as i
    tick(i)
;

repeat users where user.active as user
    send(user)
;
```

`where` é avaliado depois de obter o item e antes do corpo. Apesar de `where` aparecer antes de `as` no texto (`repeat users where user.active as user`), o nome introduzido por `as` já está disponível dentro da expressão de `where` — a ordem de escrita não é a ordem de escopo.

## Labels, break e continue

```nikosharp
outer: repeat rows as row
    repeat row.cells as cell
        if cell.done
            break outer
        ;
    ;
;
```

Label só pode ser alvo de `break`/`continue` em escopo envolvente.

## Switch statement

```nikosharp
switch value
    0
        return "zero"
    1, 2
        return "small"
    else
        return "other"
;
```

Enums sem `else` exigem exaustividade. `else` deve ser último.

## Switch expression

```nikosharp
local text = switch status
    Open => "open"
    Closed => "closed"
    else => "other"
;
```

Todos os braços devem produzir tipo comum.

## Patterns

Patterns suportados:

```text
_
nil
literal
binding
type binding
tuple
array
object/field
range
or-pattern
```

Exemplos:

```nikosharp
switch value
    Point { x = 0, y = y }
        return y
    (a, b)
        return a + b
    1..=10
        return 1
    nil
        return 0
;
```

Bindings introduzidos por patterns existem somente no braço em que foram declarados. Em or-pattern, todos os lados precisam introduzir o mesmo conjunto de nomes com tipos compatíveis.

## Destructuring local e assignment

```nikosharp
local (x, y) = point.toTuple()
(a, b) = (b, a)
```

Destructuring assignment exige que todos os alvos sejam assignable e cada alvo seja avaliado uma vez.

`local x := expr` é aceito como sinônimo de `local x = expr` (os dois inferem tipo do mesmo jeito quando não há anotação); não há diferença semântica entre as duas grafias. O formatter normaliza `:=` para `=` na forma canônica. Um `AssignmentOperator` composto (`+=`, `-=`, ...) só vale sobre um único alvo (`PostfixExpr`), nunca sobre uma tupla de destructuring.
