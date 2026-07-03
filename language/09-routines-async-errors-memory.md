# 09. Routines, async, erros e memória

## Routine

`routine` representa código estático, tipado e especializável. Ela é apropriada para callbacks de compile time, geração controlada e passagem de corpo executável sem interpretação dinâmica.

```nikosharp
routine twice(action: routine(): void)
    action()
    action()
;
```

Routine literal:

```nikosharp
local r = ()
    work()
;
```

Captura de `self` dentro de routine literal aninhada é permitida somente quando o lifetime da routine não escapa do receiver ou quando o receiver é capturado por valor seguro.

## Async

`async function` retorna valor awaitable definido pelo artefato/target. A gramática não fixa tipo de runtime.

```nikosharp
async function load(): string throws LoadError
    return try await fetchText()
;
```

`await` só é válido dentro de contexto `async`.

## Erros recuperáveis

```nikosharp
function read(path: string): string throws ReadError
    local text = try readText(path)
    return text
;
```

`throw expr` origina erro recuperável dentro de contexto `throws`. `try expr` propaga erro de chamada que já pode falhar. Cada função declara no máximo um tipo de erro; agregue categorias em tipo próprio.

## Erro fatal

`error expr` aborta o caminho atual e tem tipo `never`. Não requer `throws`.

## Memória

A linguagem não exige GC global. A semântica exige:

- `$` torna binding imutável;
- `using` garante cleanup determinístico;
- `clone()` explícito é necessário para cópia profunda quando o tipo exige;
- ponteiros são unsafe;
- valores que não escapam devem ser candidatos a stack/inline;
- valores que escapam podem usar estratégia de runtime sem alterar semântica.

## Using

```nikosharp
using file = open(path)
write(file)
```

O cleanup ocorre no fim do escopo léxico. `using` não é expressão.
