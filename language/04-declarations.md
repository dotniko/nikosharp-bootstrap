# 04. Declarações

## Ordem canônica

A ordem canônica de cabeçalho é:

```text
doc-comments
UDAs
UDMs
built-in modifiers
declaration keyword
name
signature
body
```

Exemplo:

```nikosharp
--- Ajuda visível ao usuário.
[Version(1)]
[Deprecated("Use help2 instead.")]
command function help(): CommandResult
    return CommandResult { }
;
```

Não existe forma alternativa como `function command help`. O formatter deve corrigir ou diagnosticar ordem não canônica.

## Functions

```nikosharp
function distance(a: Point, b: Point): number
    return ((a.x - b.x) ** 2 + (a.y - b.y) ** 2) ** 0.5
;
```

Parâmetros podem ter default e podem ser chamados por nome:

```nikosharp
function connect(host: string, port: u16 = 80): bool
    return true
;

local ok = connect(host: "localhost", port: 8080)
```

Parâmetro variádico deve ser o último:

```nikosharp
function sum(...values: number): number
    local total = 0
    repeat values as value
        total += value
    ;
    return total
;
```

## Methods e self

Methods aparecem dentro de `type` ou extensão. `self.` é obrigatório para acessar membros do receiver.

```nikosharp
type Counter
    value: i32

    method inc(): void
        self.value += 1
    ;
;
```

Acesso implícito a membro sem `self.` é erro.

## Properties

```nikosharp
type User
    firstName: string
    lastName: string

    property fullName: string
        return self.firstName + " " + self.lastName
    ;
;
```

A forma acima (`property nome: T` seguido direto de um `Block`) declara só getter — é somente leitura. Para uma property gravável, use `get`/`set` explícitos:

```nikosharp
type Circle
    $radius: number

    property radius: number
        get
            return self.$radius
        ;
        set
            self.$radius = value
        ;
    ;
;
```

`set` recebe implicitamente um parâmetro chamado `value` do mesmo tipo declarado na `property` — não é preciso declará-lo. Para renomear, use `set(novoNome: number)` no lugar de `set`. Property não pode ser `async`; use method.

## Operators

```nikosharp
type Vec2
    x: number
    y: number

    operator +(other: Vec2): Vec2
        return Vec2 { x = self.x + other.x, y = self.y + other.y }
    ;
;
```

Operators são resolvidos em compile time. Operador ambíguo é erro.

`operator <=>` declara comparação tripla; se `<`, `<=`, `>`, `>=` não forem declarados individualmente, o compilador os deriva a partir dele:

```nikosharp
type Version
    major: i32
    minor: i32

    operator <=>(other: Version): i32
        if self.major != other.major
            return self.major - other.major
        ;
        return self.minor - other.minor
    ;
;
```

## Extensões

`::` anexa members estáticos de extensão a um tipo existente em compile time. Não é patch dinâmico.

```nikosharp
string ::
    method isBlank(): bool
        return self.length == 0
    ;
;
```

## Tipos aninhados e partial

Tipos podem conter outros tipos. `partial type` permite dividir um tipo em vários arquivos do mesmo projeto. Todos os fragmentos precisam concordar em nome, parâmetros genéricos e metadados incompatíveis são erro.

## Const

`const` declara valor resolvido em compile time.

```nikosharp
const MaxUsers: i32 = 100
```

`const` não é variável runtime imutável; para isso use `$` em bindings.
