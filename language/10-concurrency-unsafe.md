# 10. Concorrência e unsafe

## Contratos de transferência

A linguagem define dois contratos semânticos registrados em metadados:

- `Transfer`: valor pode cruzar fronteira de execução concorrente;
- `Share`: valor pode ser acessado por mais de uma execução concorrente ao mesmo tempo.

Os nomes concretos expostos por bibliotecas não são especificados aqui. O contrato de metadados é obrigatório.

## Regras

Safe code não compartilha mutabilidade entre threads sem contrato explícito. Capturas de `async` que podem migrar para worker exigem `Transfer`. Acesso simultâneo exige `Share` ou wrapper sincronizado.

Tipos imutáveis e primitivos são candidatos naturais a `Transfer` e `Share`. Ponteiros crus nunca satisfazem automaticamente esses contratos.

## Modelo de memória

Operações atômicas e sincronização estabelecem happens-before. Leituras/escritas não atômicas concorrentes no mesmo local mutável são comportamento indefinido quando não protegidas por contrato seguro.

## Unsafe

`unsafe` cria ilha explícita:

```nikosharp
unsafe
    local p = getPointer()
;
```

Dentro de unsafe, o programador é responsável por:

- validade de ponteiros;
- alinhamento;
- ausência de uso após free;
- ausência de aliasing mutável inválido;
- preservação de invariantes de tipos;
- chamada correta de FFI.

Unsafe não desliga type checking comum. Ele apenas permite operações que safe code proíbe.
