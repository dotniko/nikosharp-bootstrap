# 12. Diagnósticos obrigatórios

Implementações compatíveis devem emitir diagnósticos determinísticos para pelo menos estes casos:

## Léxico

- caractere inválido;
- escape de string inválido;
- interpolação não fechada;
- literal numérico malformado;
- identificador público confusável.

## Sintaxe

- bloco sem `;`;
- `;` depois de statement comum;
- UDM depois da palavra declarativa;
- chamada sem parênteses fora de statement position;
- `else` sem `if` correspondente;
- pattern inválido;
- type expression incompleta.

## Semântica

- nome não resolvido;
- UDA desconhecida;
- UDM desconhecido;
- UDM em alvo incompatível;
- overload ambíguo;
- conversão implícita proibida;
- uso de `T?` como `T` sem narrowing;
- `try` em contexto sem `throws`;
- `throw` em contexto sem `throws`;
- `await` fora de `async`;
- acesso a membro sem `self.` dentro de method/property/operator;
- unsafe operation fora de `unsafe`;
- switch não exaustivo quando exigido;
- or-pattern com bindings incompatíveis.

## Quick fixes mínimos

- inserir `;` de fechamento;
- remover `;` de statement comum;
- mover UDM para antes da palavra declarativa;
- adicionar parênteses em chamada;
- trocar acesso por `self.nome`;
- adicionar `try` ou `throws`;
- adicionar `else` em switch expression;
- substituir conversão implícita por `as`.
