# 03. Tipos

## Tipos primitivos

| Tipo | Significado |
|---|---|
| `void` | ausência de valor retornado |
| `never` | expressão que não retorna |
| `bool` | `true` ou `false` |
| `number` | ponto flutuante binário de 64 bits |
| `string` | texto Unicode imutável |
| `i8`, `i16`, `i32`, `i64`, `isize` | inteiros assinados |
| `u8`, `u16`, `u32`, `u64`, `usize` | inteiros sem sinal |
| `T?` | valor ausente ou `T` |
| `*T`, `*const T` | ponteiros unsafe |
| `[T; N]` | array de tamanho fixo conhecido em compile time |

`nil` só habita `T?`. Não existe conversão implícita de `T?` para `T`.

## Tipos compostos

```nikosharp
type Point
    x: number
    y: number
;

type UserId = u64
```

Alias de tipo não cria tipo nominal distinto; usa a mesma representação. Para tipo nominal, declare `type` com corpo.

Enums são fechados:

```nikosharp
enum Status
    Open
    Closed
    Archived
;
```

O discriminante padrão é o menor unsigned integer capaz de representar todas as variantes, salvo representação explicitamente declarada por metadado.

## Optional e narrowing

`if value` é válido quando `value` é `bool`, `T?`, ou um tipo que declare `operator bool(): bool` (conversão implícita para `bool`, ver `language/07-generics-profiles-overload.md`). Quando é `T?`, o ramo verdadeiro enxerga `value` como `T` até mutação ou escape incerto. Quando é um tipo com `operator bool()`, o compilador chama esse operador uma vez e usa o resultado — não há narrowing adicional nesse caso, só no caso `T?`.

```nikosharp
if user
    return user.name
;
```

`?.`, `?[` e `??` operam apenas sobre optional.

## Fixed arrays

`[T; N]` é stack-friendly e tem tamanho fixo. `N` precisa ser `ConstExpr` de tipo inteiro não negativo. O tipo `[i32; 4]` é diferente de `[i32; 5]`.

## Tuplas

Tuplas têm aridade fixa. `(T)` não é tupla; `(T, U)` é.

## Ponteiros e unsafe

Ponteiros só podem ser criados, dereferenciados ou convertidos dentro de `unsafe` ou por item marcado com UDM/metadata equivalente que autorize unsafe. Ponteiros não possuem ownership automático.

## Conversões numéricas

Conversões implícitas são permitidas apenas quando preservam valor para todos os casos possíveis.

| De | Para implícito |
|---|---|
| `i8` | `i16`, `i32`, `i64`, `number` |
| `i16` | `i32`, `i64`, `number` |
| `i32` | `i64`, `number` |
| `i64` | `number` somente com diagnóstico de perda potencial se exigido por perfil estrito |
| `u8` | `u16`, `u32`, `u64`, `number` |
| `u16` | `u32`, `u64`, `number` |
| `u32` | `u64`, `number` |
| `u64` | `number` somente com diagnóstico de perda potencial se exigido por perfil estrito |
| `usize` | `u64` quando target garante largura menor ou igual; caso contrário explícito |
| `isize` | `i64` quando target garante largura menor ou igual; caso contrário explícito |

Não há conversão implícita entre signed e unsigned exceto literal inteiro positivo que caiba no tipo alvo. Narrowing exige `as`.

## Conversões de profile

Um tipo concreto converte implicitamente para profile satisfeito por ele quando todos os membros exigidos estão presentes e tipos são compatíveis. Downcast usa `as` ou `as?`.

## Variância

Tipos genéricos definidos pelo usuário são invariantes por padrão. Variância precisa ser declarada:

```nikosharp
profile Producer<out T>
    method next(): T?
;

profile Consumer<in T>
    method accept(value: T)
;
```

`out` só permite `T` em posições de saída. `in` só permite `T` em posições de entrada. Tipos mutáveis comuns devem ser invariantes.
