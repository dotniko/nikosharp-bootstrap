# 08. UDA e UDM

## Definições

- UDA significa User Defined Attribute.
- UDM significa User Defined Modifier.

Ambos são metadados compile-time. Nenhum dos dois cria reflection em runtime.

## UDA

UDA usa colchetes antes da declaração:

```nikosharp
[Version(1)]
[Deprecated("Use help2 instead.")]
function oldHelp(): void
    return
;
```

Uma UDA é declarada por `attribute`:

```nikosharp
attribute Version
    value: u32
;

attribute Deprecated
    message: string
;
```

Campos de UDA devem ser tipos serializáveis em metadados: primitivos, string, enum, tipo, array fixo de valores constantes, tupla constante ou outro valor constante permitido. Argumentos de UDA são `ConstExpr`.

## UDM

UDM aparece como palavra antes da declaração:

```nikosharp
command function hello(): void
    return
;
```

UDM é declarado por `modifier`:

```nikosharp
modifier command: function
;
```

UDM não tem argumentos. Se uma anotação precisa de dados, use UDA junto:

```nikosharp
[Version(1)]
command function help(): CommandResult
    return CommandResult { }
;
```

## Ordem canônica

```text
UDA(s)
UDM(s)
built-in modifier(s)
declaration
```

Exemplo:

```nikosharp
[Path("/")]
apiRoute async function index(): number
    return 200
;
```

UDMs aparecem na ordem declarada pelo autor. Built-in modifiers como `async`, `partial` e `unsafe` aparecem depois dos UDMs quando pertencem ao cabeçalho.

## Alvos

Alvos válidos de UDM:

```text
function, method, property, type, enum, profile, routine, operator, const
```

Aplicar UDM em alvo incompatível é erro. Aplicar UDM desconhecido é erro. Duplicar o mesmo UDM no mesmo item é erro.

## Parsing

O lexer não transforma UDM em keyword dinâmica. O parser lê uma sequência de identificadores antes de uma palavra declarativa e os registra como `ModifierUse`. A semântica resolve esses nomes contra modificadores conhecidos.

Isso permite parser determinístico e evita que dependências alterem o léxico da linguagem.

## Metadados

UDAs e UDMs devem aparecer em `.nikolib` com:

- nome canônico;
- alvo;
- argumentos constantes, no caso de UDA;
- origem de declaração;
- spans de source quando debug estiver presente;
- visibilidade pública quando afetam ABI, docs ou descoberta.

## Restrições

UDA/UDM não podem:

- alterar gramática;
- executar código em runtime;
- inserir statements invisíveis;
- modificar regras de overload;
- criar símbolos sem generator declarado;
- depender da ordem física dos arquivos.
