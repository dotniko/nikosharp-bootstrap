# 11. Formatação canônica

O formatter é parte da linguagem. A gramática foi desenhada para ter uma saída canônica estável.

## Regras principais

- indentação de 4 espaços;
- sem tabs;
- uma declaração top-level separada por uma linha em branco;
- blocos fecham com `;` sozinho na indentação do cabeçalho;
- statements comuns não terminam com `;`;
- UDA uma por linha;
- UDMs na mesma linha do item;
- `async` aparece depois de UDMs;
- chamadas em expressão sempre usam parênteses;
- statement call sem parênteses é preservado somente com um argumento simples;
- trailing comma permitido em listas multi-linha;
- `try await` é forma canônica.

## Cabeçalhos

```nikosharp
[Version(1)]
command async function run(name: string): void throws RunError
    return
;
```

## Objetos

Uma construção pequena pode ficar em uma linha:

```nikosharp
Point { x = 1, y = 2 }
```

Construção multi-linha usa trailing comma:

```nikosharp
Point {
    x = 1,
    y = 2,
}
```

## Switch

```nikosharp
switch status
    Open
        return 1
    Closed
        return 0
    else
        return -1
;
```
