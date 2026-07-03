# 07. Generics, profiles e overload

## Generics

```nikosharp
function identity<T>(value: T): T
    return value
;
```

Bounds múltiplos usam `where`:

```nikosharp
function max<T>(a: T, b: T): T where T: Comparable + Equatable
    if a > b
        return a
    ;
    return b
;
```

A ordem dos bounds não é semântica. Bounds duplicados são diagnosticados.

## Static parameters

`static` em parâmetro generic/routine exige valor conhecido em compile time.

```nikosharp
routine unroll(static count: i32)
    repeat count as i
        step(i)
    ;
;
```

Especialização recursiva precisa respeitar orçamento configurado pelo compilador. Estouro de orçamento é erro determinístico.

## Profiles

Profile é contrato estrutural:

```nikosharp
profile Comparable<T>
    operator <(other: T): bool
;
```

Um tipo satisfaz um profile quando possui todos os membros exigidos com assinatura compatível. Satisfação é calculada em compile time e registrada em metadados de artefato.

## Prioridade entre profiles

Quando mais de um profile satisfaz uma chamada, a escolha segue:

1. membro concreto do tipo receiver;
2. extensão mais específica ao tipo concreto;
3. profile explicitamente anotado no tipo contextual;
4. profile com menor conjunto de conversões necessárias;
5. erro de ambiguidade.

## Overload resolution

Para uma chamada, o compilador monta candidatos visíveis com mesmo nome e aridade compatível. Depois aplica:

1. descartar candidatos com tipo de receiver incompatível;
2. descartar candidatos com argumentos nomeados inexistentes;
3. descartar candidatos que exigem conversão proibida;
4. classificar cada argumento: exact, optional-lift, numeric-widening, profile-upcast, explicit-conversion-not-allowed;
5. preferir candidato com mais matches exact;
6. preferir candidato que não usa default;
7. preferir candidato não variádico;
8. preferir membro concreto a extensão;
9. preferir declaração mais local;
10. se ainda houver empate, erro.

Nenhuma escolha pode depender de ordem textual entre arquivos.

## Variância

`in` e `out` só podem ser usados em parâmetros de profile. Tipos concretos genéricos são invariantes salvo regra explícita futura de artefato.
