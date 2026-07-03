# `.ns`

`.ns` é o arquivo fonte NikoSharp.

## Regras

- UTF-8 obrigatório;
- extensão `.ns` minúscula;
- um arquivo pode conter várias declarações;
- top-level statements são opcionais e dependem do projeto;
- nenhuma ordem física entre arquivos pode alterar overload resolution;
- documentação `---` pertence ao próximo item declarativo.

## Entrada

A entrada executável padrão é uma função configurada no `.nikoproj`. Se não configurada, o nome padrão é `main`.

Assinaturas aceitas por padrão:

```nikosharp
function main(): void
function main(): i32
```

Argumentos de linha de comando não fazem parte do contrato de `main` nesta gramática: `[T; N]` exige `N` como `ConstExpr` conhecido em tempo de compilação (`language/03-types.md`), e a quantidade de argumentos só é conhecida em tempo de execução, então uma terceira assinatura `main(args: [string; N])` seria inconsistente com o próprio sistema de tipos. Acesso a argumentos de linha de comando é responsabilidade de biblioteca padrão/target (fora de escopo deste documento), não da gramática da linguagem.
