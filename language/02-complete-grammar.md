# 02. Gramática completa (revisão consolidada)

Esta é a revisão consolidada da gramática canônica de NikoSharp para parser bootstrap. Ela substitui integralmente a versão anterior de `language/02-complete-grammar.md`.

Proveniência: a base é a spec bootstrap atual (a que define UDA/UDM, `partial`, patterns ricos, `[T; N]`, ponteiros, `unsafe`, `local`/labels, sem import/módulo). Um conjunto arcaico de rascunhos (mais antigo, com `module`/`use`/`include`, `[entry]`/endpoint binding, ECS, stdlib extensa) foi consultado **apenas** para resolver lacunas pontuais de sintaxe que a versão atual deixava em aberto — nenhuma sintaxe, keyword ou mecanismo exclusivo do rascunho arcaico (imports, `module`, `[entry]`, tipos de stdlib como `List`/`Map`) foi trazido de volta. `=>` **é** incorporado, mas só como mais um operador binário sobrecarregável comum — nunca esteve preso a ponto de entrada de programa nesta spec, ver seção 6.3. Onde algo do arcaico foi aproveitado, isso está marcado explicitamente no Apêndice B.

Esta gramática cobre **somente a linguagem central** (léxico, sintaxe, tipos de superfície). Não descreve biblioteca padrão, runtime, backend ou toolchain — o suficiente para: (1) um bootstrap em Python que faça lexer + parser + AST + checagens semânticas mínimas, e (2) depois disso, cada módulo (lexer, parser, checker, formatter, etc.) ser reescrito como projeto Rust separado consumindo a mesma gramática como contrato.

`NL` representa uma ou mais quebras de linha ou trivia equivalente preservada pela árvore sintática. `;` fecha blocos, não statements comuns.

## 0. Notação

EBNF prático: `|` alternativa, `?` opcional, `*` zero ou mais, `+` uma ou mais, `{}` agrupamento repetível, `[]` grupo opcional, `""` terminal literal. Regras de precedência/associatividade de expressões **não** são codificadas como aninhamento de produções (isso explodiria a gramática em ~18 camadas); `BinaryExpr` é propositalmente plana e um parser bootstrap deve resolvê-la por precedence climbing / Pratt parsing usando a tabela do Apêndice A. Essa é uma escolha de implementação deliberada, não uma lacuna.

## 1. Léxico

### 1.1 Identificadores

```ebnf
identifier-start = "_" | unicode-letter ;
identifier-part  = "_" | unicode-letter | unicode-decimal-digit ;
identifier       = identifier-start, { identifier-part } ;
```

Identificadores são normalizados para NFC antes de comparação semântica; duas grafias que normalizam para o mesmo valor são o mesmo identificador. Implementações devem diagnosticar identificadores públicos com caracteres visualmente confundíveis quando houver risco de colisão.

Convenções (não alteram parsing; formatter/linter podem avisar):

- tipos e UDAs usam PascalCase;
- valores, funções, properties e UDMs usam lowerCamelCase;
- `_` inicial é reservado por convenção para itens internos ou descartes locais.

### 1.2 Keywords

Hard:

```text
and as async await attribute break continue const else enum error false function if in local method modifier nil not operator or partial profile property repeat return routine switch throw throws true try type unsafe using where
```

Contextuais (identificadores fora da posição gramatical específica):

```text
get set self static
```

`self` só é ligado automaticamente dentro de `method`, `property` accessor e `operator` de instância. `static` marca parâmetro/argumento de tipo conhecido em tempo de compilação — não existe "static method"; um membro associado ao tipo e não ligado a `self` já é apenas uma `function` declarada dentro do corpo de `type`.

Palavras de tipo reservadas em contexto de tipo:

```text
void never bool number string i8 i16 i32 i64 isize u8 u16 u32 u64 usize
```

### 1.3 Comentários

```ebnf
line-comment = "--", { char-except-newline } ;
doc-comment  = "---", { char-except-newline } ;
```

`---` anexa documentação ao próximo item declarativo. Não existe delimitador de bloco de comentário. Comentários não alteram gramática.

### 1.4 Literais numéricos

```ebnf
digit          = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
hex-digit      = digit | "a".."f" | "A".."F" ;
bin-digit      = "0" | "1" ;
dec-int        = digit, { digit | "_" } ;
hex-int        = "0x", hex-digit, { hex-digit | "_" } ;
bin-int        = "0b", bin-digit, { bin-digit | "_" } ;
exponent       = ("e" | "E"), ["+" | "-"], dec-int ;
number-literal = (dec-int, [".", dec-int], [exponent]) | hex-int | bin-int ;
```

`_` não pode aparecer no começo, fim, imediatamente após prefixo, imediatamente antes/depois de `.`, nem repetido adjacente. Literais inteiros sem tipo são tipados por contexto; literais reais sem contexto são `number`. Literal fora do range do tipo alvo é erro.

### 1.5 Strings

```ebnf
string-literal = '"', { string-part | interpolation }, '"' ;
interpolation  = "{", interpolation-expression, [ ":", format-spec ], "}" ;
```

Escapes válidos:

| Escape | Valor |
|---|---|
| `\\` | barra invertida |
| `\"` | aspas |
| `\0` | NUL |
| `\n` | line feed |
| `\r` | carriage return |
| `\t` | tab |
| `\b` | backspace |
| `\f` | form feed |
| `\u{H...}` | escalar Unicode de 1 a 6 hex digits |
| `{{` | `{` literal em string interpolada |
| `}}` | `}` literal em string interpolada |

`format-spec` é texto opaco para o formatter; a semântica pertence ao consumidor de metadados, não ao lexer. `interpolation-expression` aceita apenas: identificador, literal, `self`, acesso de membro (`.`, `?.`), indexação (`[]`, `?[`), chamada com parênteses, operadores unários e binários — sem assignment, `return`, `throw`, `error`, `using` e sem formas de bloco. O lexer rastreia profundidade de chave para tokenizar corretamente, então uma string aninhada dentro de `{...}` não fecha a string externa por engano.

### 1.6 Tokens especiais

```text
( ) [ ] { } , . ?. ?[ : :: ; => ->
+ - * / % **
& | ^ ~ << >> >>>
= := += -= *= /= %= **= &= |= ^= <<= >>= >>>=
== != < <= > >=
.. ..= ?? ?
```

**Correção em relação à revisão anterior:** o token de indexação segura é `?[` (dois caracteres — abre, não fecha), não `?[]`. `x?[5]` deve ser lexado como `identifier("x") ?[ number-literal(5) ]`; o token fechado `?[]` era incompatível com a própria produção que o usava (ver Apêndice B).

O lexer deve escolher o maior token possível.

## 2. Arquivo e metadados

```ebnf
File            = Shebang?, FileItem* ;
Shebang         = "#!", { char-except-newline }, NL ;
FileItem        = Declaration | TopLevelStatement ;
```

**Correção:** a revisão anterior tinha `FileItem = Metadata*, Declaration | TopLevelStatement`, com `Metadata*` duplicado — `Declaration` já começa com `DeclarationHead = Metadata*, UdmUse*`. O `Metadata*` extra em `FileItem` foi removido; metadados só entram por `DeclarationHead`.

Top-level statements são permitidos somente quando o projeto declara entrada por arquivo (`.nikoproj`); fora desse modo, são erro semântico.

```ebnf
Metadata        = Uda | DocComment ;
Uda             = "[", TypeName, UdaArguments?, "]" ;
UdaArguments    = "(", [ ArgumentList ], ")" ;
UdmUse          = identifier ;
DeclarationHead = Metadata*, UdmUse* ;
```

O parser aceita identificadores antes de uma palavra declarativa como possíveis UDMs. A validação semântica confirma se cada UDM é conhecido e permitido no alvo; o lexer não transforma UDM em keyword dinâmica.

## 3. Declarações de topo

```ebnf
Declaration     = DeclarationHead, DeclarationCore ;
DeclarationCore = AttributeDecl
                | ModifierDecl
                | TypeDecl
                | EnumDecl
                | ProfileDecl
                | ExtensionDecl
                | FunctionDecl
                | RoutineDecl
                | ConstDecl ;

AttributeDecl   = "attribute", TypeName, TypeParameters?, NL,
                  AttributeFieldDecl*, ";" ;
AttributeFieldDecl = identifier, ":", TypeExpr, [ "=", ConstExpr ], NL ;

ModifierDecl    = "modifier", identifier, [ ":", ModifierTargetList ], NL?, ";" ;
ModifierTargetList = ModifierTarget, { ",", ModifierTarget } ;
ModifierTarget  = "function" | "method" | "property" | "type" | "enum" | "profile" | "routine" | "operator" | "const" ;

ConstDecl       = "const", BindingPattern, [ ":", TypeExpr ], "=", ConstExpr, NL ;
```

UDM declarations não carregam argumentos; quando dados são necessários, use UDA junto.

## 4. Parâmetros de tipo (genéricos)

**Adição — não existia como produção formal na revisão anterior**, apesar de `TypeParameters` ser referenciado por `TypeDecl`, `FunctionDecl`, `MethodDecl`, `RoutineDecl`, `AttributeDecl` e `ProfileDecl`.

```ebnf
TypeParameters  = "<", TypeParamItem, { ",", TypeParamItem }, [ "," ], ">" ;
TypeParamItem   = TypeParam | StaticParam ;
TypeParam       = [ "in" | "out" ], TypeName, [ ":", TypeExpr, { "+", TypeExpr } ] ;
StaticParam     = "static", BindingName, ":", TypeExpr ;
```

- `TypeParam` é um parâmetro de tipo comum; o bound inline (`<T: Comparable>`) é equivalente a declarar o mesmo bound via `where` na assinatura — os dois formam a mesma restrição, só muda onde é escrita.
- `in`/`out` só são válidos em `TypeParam` de `profile` (variância); em qualquer outro alvo de `TypeParameters` é erro semântico (`language/03-types.md`, `language/07-generics-profiles-overload.md`).
- `StaticParam` declara um **parâmetro de tipo estático**: um valor conhecido em tempo de compilação (equivalente a `comptime` do Zig ou const generics de Rust), não um tipo. `TypeExpr` aqui é o tipo do valor constante (ex.: `<static N: i32>`), não uma lista de bounds — `+` não é válido depois do `:` de um `StaticParam`.
- `StaticParam` também pode aparecer fora de `TypeParameters`, como modificador de `Parameter` em `ParameterList` (seção 6.4) — as duas posições (`<static N: i32>(...)` e `(static count: i32)`) são formas irmãs para o mesmo conceito: uma declara o parâmetro estático na lista de tipo, a outra na lista de parâmetros de valor de uma `routine`/`function`/`method` genérica.

## 5. Tipos, membros e extensões

```ebnf
TypeDecl        = [ "partial" ], "type", TypeName, TypeParameters?, TypeBodyOrAlias ;
TypeBodyOrAlias = "=", TypeExpr, NL
                | NL, TypeMember*, ";" ;

TypeMember      = DeclarationHead, ( FieldDecl | ConstructorDecl | MethodDecl | PropertyDecl | OperatorDecl | TypeDecl | EnumDecl ) ;
FieldDecl       = [ "$" ], BindingName, ":", TypeExpr, [ "=", Expr ], NL ;
ConstructorDecl = "function", "new", ParameterList, Block ;

EnumDecl        = "enum", TypeName, [ ":", IntegerType ], NL, EnumVariant+, ";" ;
EnumVariant     = TypeName, [ "=", ConstExpr ], NL ;

ProfileDecl     = "profile", TypeName, TypeParameters?, WhereClause?, NL, ProfileMember*, ";" ;
ProfileMember   = DeclarationHead, ( MethodSignature | PropertySignature | OperatorSignature ) ;

ExtensionDecl   = TypeExpr, "::", NL, ExtensionMember*, ";" ;
ExtensionMember = DeclarationHead, ( MethodDecl | PropertyDecl | OperatorDecl | FunctionDecl | ConstDecl ) ;
```

`$` antes de `BindingName` em `FieldDecl` marca o campo como imutável após a construção (mesma convenção de `Parameter` e `BindingPattern` — `$` sempre precede o nome, nunca o tipo).

`partial` permite dividir o mesmo tipo em vários arquivos do mesmo projeto, desde que todos os fragmentos tenham metadados compatíveis. `::` anexa membros de extensão a um tipo existente em tempo de compilação — não é patch dinâmico; extensão não pode adicionar campo, mudar layout, mudar visibilidade ou mutar tabelas de tipo em runtime.

`function new` só aparece dentro de `type` e **não aceita `throws` nem `async`** nesta gramática bootstrap (`ConstructorDecl` não tem `ReturnAndThrows` nem prefixo `async`) — construção que pode falhar deve usar uma função-fábrica declarada ao lado do tipo (ex.: `function create(...): T throws E`), não um construtor lançante. Isso é uma restrição intencional, não uma omissão: mantém `Type { ... }` e `Type.new(...)` sempre infalíveis na forma canônica.

`IntegerType` (usado no discriminante explícito de `enum`):

```ebnf
IntegerType     = "i8" | "i16" | "i32" | "i64" | "isize" | "u8" | "u16" | "u32" | "u64" | "usize" ;
```

## 6. Funções, methods, properties, operators, routines

### 6.1 Assinaturas

```ebnf
FunctionDecl    = [ "async" ], "function", FunctionName, TypeParameters?, ParameterList,
                  ReturnAndThrows?, WhereClause?, Block ;
MethodDecl      = [ "async" ], "method", FunctionName, TypeParameters?, ParameterList,
                  ReturnAndThrows?, WhereClause?, Block ;
MethodSignature = [ "async" ], "method", FunctionName, TypeParameters?, ParameterList,
                  ReturnAndThrows?, WhereClause?, NL ;
```

`FunctionName` é `identifier`. Methods nunca declaram parâmetro `self`; `self` é ligado automaticamente e seu uso para acessar membros do receiver é obrigatório (`self.membro`).

### 6.2 Properties

```ebnf
PropertyDecl     = "property", BindingName, ":", TypeExpr, PropertyBody ;
PropertyBody     = Block | NL, PropertyAccessor+, ";" ;
PropertyAccessor = "get", ReturnAndThrows?, Block
                 | "set", SetterParameter?, ReturnAndThrows?, Block ;
SetterParameter  = "(", BindingName, ":", TypeExpr, ")" ;
PropertySignature = "property", BindingName, ":", TypeExpr, NL ;
```

**Correção/adição em relação à revisão anterior:** `PropertyAccessor` não tinha nenhuma forma de o corpo de `set` acessar o valor atribuído. Regra:

- se `SetterParameter` for omitido, `set` recebe implicitamente um parâmetro chamado `value`, cujo tipo é o `TypeExpr` declarado na `property`;
- se `SetterParameter` estiver presente, ele nomeia esse mesmo parâmetro explicitamente — o tipo declarado deve ser idêntico ao tipo da `property`;
- `PropertyBody = Block` (sem `get`/`set` explícitos) declara apenas um getter — a property resultante é somente leitura. Para uma property gravável, use a forma com `PropertyAccessor+` incluindo `set`.

`property` não pode ser `async`; use `method` para trabalho assíncrono. `PropertyAccessor` pode declarar `throws`.

### 6.3 Operators

```ebnf
OperatorDecl      = "operator", OperatorName, ParameterList, ReturnAndThrows?, Block ;
OperatorSignature = "operator", OperatorName, ParameterList, ReturnAndThrows?, NL ;

OperatorName      = OverloadableBinaryOp
                   | OverloadableUnaryOnlyOp
                   | "[", "]"
                   | "[", "]", "="
                   | "(", ")"
                   | "as", TypeExpr
                   | "in"
                   | "=>"
                   | TypeName ;

OverloadableBinaryOp    = "+" | "-" | "*" | "/" | "%" | "**"
                        | "==" | "!=" | "<" | "<=" | ">" | ">=" | "<=>"
                        | "&" | "|" | "^" | "<<" | ">>" | ">>>" ;
OverloadableUnaryOnlyOp = "~" ;
```

**Adição — `OperatorName` não existia como produção na revisão anterior.** Regras de desambiguação:

- `+`, `-` e `*` servem tanto para operador binário quanto unário (`*` unário é dereferência/acesso transparente de um tipo "ponteiro inteligente" definido pelo usuário, não a dereferência builtin de `*T`); a aridade de `ParameterList` decide: zero parâmetros = unário (`operator -(): T`, `operator *(): T`), um parâmetro = binário (`operator +(other: T): T`).
- `[]` (index get) tem um parâmetro (o índice) e retorna o tipo do elemento; `[]=` (index set) tem dois parâmetros (índice, valor) e retorna `void`. Indexação multi-chave usa múltiplos parâmetros na mesma `ParameterList` (ex.: `operator [](i: i32, j: i32): T?`).
- `()` (call) tem a `ParameterList` que a chamada aceitará.
- `as TypeExpr` declara conversão **explícita** para `TypeExpr`; o mesmo `operator as` atende tanto `expr as T` (lança/erro em falha) quanto `expr as? T` (retorna `T?`) — não existe um `operator as?` separado.
- `TypeName` sozinho (sem `as`) declara conversão **implícita** para esse tipo (`operator bool(): bool`, `operator string(): string`). É uma família de operador diferente de `as TypeExpr`: só é chamada automaticamente pelo compilador nos pontos que a spec define como contexto de coerção (hoje, somente a condição de `if`/`repeat where` aceitando `operator bool()` — `language/03-types.md`); em qualquer outro lugar só é alcançável via `as TypeName` explícito.
- `in` declara `operator in(value: T): bool` para o tipo receptor à direita de `in` (`value in receiver`).
- `<=>` declara `operator <=>(other: T): i32` (comparação tripla: negativo, zero, positivo). Se um tipo declarar `<=>` sem declarar individualmente `<`, `<=`, `>` ou `>=`, o compilador deriva cada um comparando o resultado de `<=>` com zero — mesma lógica de `!=` derivando de `==`.
- `=>` é um operador binário livre comum (`operator =>(input: T): R`) — **não** tem relação com ponto de entrada do programa (isso é sempre `function main()`, configurável só via `.nikoproj`/`source.entry`; ver Apêndice C). Dentro de um braço de `SwitchExprArm`, `=>` continua sendo sintaxe de controle fixa, não uma chamada de operador — as duas posições não colidem porque `SwitchExprArm` não passa por `BinaryTail` naquele ponto.
- `!=` não é declarável separadamente: quando ausente, deriva de `not (a == b)`.
- `++`/`--` (incremento/decremento) não existem nesta gramática — decisão em aberto, deliberadamente não resolvida nesta revisão.

### 6.4 Routines

```ebnf
RoutineDecl     = [ "async" ], "routine", FunctionName, TypeParameters?, ParameterList,
                  ReturnAndThrows?, WhereClause?, RoutineBlock ;
RoutineLiteral  = [ "async" ], ParameterList, ReturnAndThrows?, RoutineBlock ;
RoutineBlock    = Block ;
```

### 6.5 Parâmetros e cláusulas comuns

```ebnf
ReturnAndThrows = [ ":", TypeExpr ], [ "throws", TypeExpr ] ;
ParameterList   = "(", [ Parameter, { ",", Parameter }, [ "," ] ], ")" ;
Parameter       = [ "static" ], [ "$" ], BindingName, ":", TypeExpr, [ "=", ConstExpr ]
                | "...", BindingName, ":", TypeExpr ;
WhereClause     = "where", WhereBound, { ",", WhereBound } ;
WhereBound      = TypeName, ":", TypeExpr, { "+", TypeExpr } ;
```

Parâmetro variádico (`...`) deve ser o último e não combina com `static` nem `$`. `static` em `Parameter` marca um argumento que deve ser conhecido em tempo de compilação (ver seção 4); é válido em `RoutineDecl`/`RoutineLiteral` e em `FunctionDecl`/`MethodDecl` genéricas. A ordem dos bounds em `WhereClause` não é semântica; bounds duplicados são diagnosticados.

## 7. Statements

```ebnf
Block           = NL, Statement*, ";" ;
```

**Adição — `Block` não existia como produção na revisão anterior**, apesar de ser usado por praticamente toda declaração com corpo (`IfStmt`, `RepeatStmt`, `MethodDecl`, `PropertyAccessor`, `OperatorDecl`, `ConstructorDecl`, `UnsafeStmt`, `RoutineBlock`, etc.).

```ebnf
Statement       = LocalDecl
                | Assignment
                | IfStmt
                | RepeatStmt
                | SwitchStmt
                | ReturnStmt
                | BreakStmt
                | ContinueStmt
                | LabelStmt
                | UsingStmt
                | ThrowStmt
                | ErrorStmt
                | UnsafeStmt
                | StatementCall
                | ExprStatement ;

LocalDecl       = "local", BindingPattern, [ ":", TypeExpr ], ("=" | ":="), Expr, NL
                | "local", BindingPattern, ":", TypeExpr, NL ;
```

`:=` em `LocalDecl` é sintaticamente aceito como sinônimo de `=` (ambos inferem tipo quando `TypeExpr` está ausente); não há diferença semântica entre `local x = expr` e `local x := expr`. O formatter deve normalizar `:=` para `=` na forma canônica, do mesmo modo que já normaliza `await try` para `try await` — não há dois comportamentos distintos escondidos atrás dessas duas grafias.

```ebnf
Assignment      = Assignable, AssignmentOperator, Expr, NL ;
Assignable      = PostfixExpr
                | "(", Assignable, { ",", Assignable }, [ "," ], ")" ;
AssignmentOperator = "=" | "+=" | "-=" | "*=" | "/=" | "%=" | "**=" | "&=" | "|=" | "^=" | "<<=" | ">>=" | ">>>=" ;
```

**Adição:** `Assignable` não existia como produção; a forma tupla cobre o destructuring assignment (`(a, b) = (b, a)`). Um `AssignmentOperator` composto (`+=`, `-=`, ...) só é válido quando `Assignable` é `PostfixExpr` — não se aplica a tupla. Compound assignment lê o alvo uma única vez (`collection[expensiveIndex()] += 1` avalia `expensiveIndex()` uma vez, não duas).

```ebnf
IfStmt          = "if", Expr, Block, [ "else", ( IfStmt | Block ) ] ;
RepeatStmt      = [ LabelPrefix ], "repeat", RepeatSource, [ "where", Expr ], [ "as", BindingPattern ], Block ;
RepeatSource    = Expr ;
```

**Correção:** a revisão anterior tinha `RepeatSource = Expr | RangeExpr`, com `RangeExpr` nunca definida em lugar nenhum. Como `..`/`..=` já são `BinaryOperator` (nível 12 de precedência — seção Apêndice A), um range como `1..10` já é um `Expr` comum; a alternativa `RangeExpr` era redundante e foi removida.

Nota de escopo do `where`/`as`: apesar de `where Expr` aparecer **antes** de `as BindingPattern` na gramática (e nos exemplos), o item vinculado por `as` já está disponível dentro da expressão de `where` — `where` é avaliado depois de obter o item e antes do corpo, então `repeat users where user.active as user` é válido mesmo com essa ordem textual. Implementações devem resolver o nome de `as` antes de checar `where`, não na ordem em que os tokens aparecem.

```ebnf
SwitchStmt      = "switch", Expr, NL, SwitchArm*, ";" ;
SwitchArm       = PatternList, [ "where", Expr ], Block
                | "else", Block ;
PatternList     = Pattern, { ",", Pattern } ;
ReturnStmt      = "return", [ Expr ], NL ;
BreakStmt       = "break", [ LabelName ], NL ;
ContinueStmt    = "continue", [ LabelName ], NL ;
LabelStmt       = LabelName, ":", NL ;
LabelPrefix     = LabelName, ":" ;
UsingStmt       = "using", BindingPattern, "=", Expr, NL ;
ThrowStmt       = "throw", Expr, NL ;
ErrorStmt       = "error", Expr, NL ;
UnsafeStmt      = "unsafe", Block ;
StatementCall   = CallTarget, CallArgument, NL ;
CallTarget      = identifier, { ".", identifier } ;
CallArgument    = Expr ;
ExprStatement   = Expr, NL ;
```

**Adição:** `CallTarget`, `CallArgument` e `LabelName` (= `identifier`) não existiam como produções. Call sem parênteses só existe em statement position e aceita exatamente um argumento sintático (`print "hello"`); nunca é válida dentro de expressão (`local x = max 1 2` é erro de sintaxe).

## 8. Patterns

```ebnf
Pattern         = "_"
                | "nil"
                | Literal
                | BindingPattern
                | TypePattern
                | TuplePattern
                | ArrayPattern
                | ObjectPattern
                | RangePattern
                | OrPattern ;
BindingPattern  = [ "$" ], BindingName
                | "(", BindingPattern, { ",", BindingPattern }, [ "," ], ")" ;
TypePattern     = TypeExpr, BindingPattern? ;
TuplePattern    = "(", Pattern, ",", Pattern, { ",", Pattern }, [ "," ], ")" ;
ArrayPattern    = "[", [ Pattern, { ",", Pattern }, [ "," ] ], "]" ;
ObjectPattern   = TypeExpr?, "{", [ FieldPattern, { ",", FieldPattern }, [ "," ] ], "}" ;
FieldPattern    = BindingName, [ "=", Pattern ] ;
RangePattern    = Expr, (".." | "..="), Expr ;
OrPattern       = Pattern, "or", Pattern, { "or", Pattern } ;
```

Patterns podem aparecer em `switch` (statement e expression), destructuring `local` e destructuring assignment. Bindings introduzidos por patterns existem somente no braço em que foram declarados; em `OrPattern`, todos os lados devem introduzir o mesmo conjunto de nomes com tipos compatíveis.

## 9. Expressões

```ebnf
Expr            = BinaryExpr ;
BinaryExpr      = PrefixExpr, { BinaryTail } ;
BinaryTail      = BinaryOperator, PrefixExpr
                | AsOperator, TypeExpr ;
AsOperator      = "as", [ "?" ] ;
BinaryOperator  = "**"
                | "*" | "/" | "%"
                | "+" | "-"
                | "<<" | ">>" | ">>>"
                | "&" | "^" | "|"
                | ".." | "..="
                | "<" | "<=" | ">" | ">=" | "<=>"
                | "in"
                | "==" | "!="
                | "and" | "or"
                | "??"
                | "=>" ;
PrefixExpr      = PrefixOperator*, PostfixExpr ;
PrefixOperator  = "+" | "-" | "*" | "not" | "~" | "try" | "await" ;
PostfixExpr     = PrimaryExpr, { PostfixPart } ;
```

**Adição:** `BinaryOperator`, `PrefixOperator` e `AsOperator` não existiam como produções (eram só prosa/tabela de precedência). `AsOperator` recebe `TypeExpr` à direita, não `PrefixExpr` — é por isso que fica separado de `BinaryTail`/`BinaryOperator` em vez de ser só mais um item da lista. Prefixo `*` desambigua de `*` binário pela mesma regra de aridade usada em `+`/`-` (não é uma ambiguidade nova, é a mesma já aceita pela gramática para os outros dois). A precedência real (19 níveis, associatividades e não-encadeamento de comparação/igualdade sem `and`) é dada pela tabela do Apêndice A; esta produção plana só define o vocabulário de tokens.

```ebnf
PrimaryExpr     = Literal
                | identifier
                | "self"
                | "nil"
                | "true"
                | "false"
                | "(", Expr, ")"
                | TupleExpr
                | SequenceLiteral
                | KeyedLiteral
                | ConstructionExpr
                | SwitchExpr
                | RoutineLiteral ;
Literal         = number-literal | string-literal ;
```

**Adição:** `Literal` não existia como produção própria (era usada mas nunca definida); `nil`/`true`/`false` continuam como alternativas próprias de `PrimaryExpr`/`Pattern`, não fazem parte de `Literal`.

```ebnf
PostfixPart     = ".", identifier
                | "?.", identifier
                | "(", [ ArgumentList ], ")"
                | "[", IndexArgumentList, "]"
                | "?[", IndexArgumentList, "]" ;
ArgumentList    = Argument, { ",", Argument }, [ "," ] ;
Argument        = [ identifier, ":" ], Expr ;
IndexArgumentList = Expr, { ",", Expr }, [ "," ] ;
```

**Correções:**
- indexação segura agora usa o token `?[` (fechado corretamente por `]` depois do(s) índice(s)) em vez do `?[]` já fechado da revisão anterior (ver seção 1.6).
- índice (`[...]` e `?[...]`) agora usa `IndexArgumentList` (lista posicional de `Expr`), não `ArgumentList` — a forma anterior permitia sintaticamente argumento nomeado dentro de colchete de índice (`arr[i: 5]`), o que não é intencional. Indexação multi-chave continua válida (`matrix[i, j]`), só sem nomes.
- chamada (`(...)`) continua usando `ArgumentList` com suporte a argumento nomeado; argumentos nomeados podem vir depois de posicionais, mas após o primeiro nomeado todos os seguintes devem ser nomeados.

```ebnf
TupleExpr       = "(", Expr, ",", Expr, { ",", Expr }, [ "," ], ")" ;
SequenceLiteral = "[", [ Expr, { ",", Expr }, [ "," ] ], "]" ;
KeyedLiteral    = "{", [ KeyedEntry, { ",", KeyedEntry }, [ "," ] ], "}" ;
KeyedEntry      = identifier, "=", Expr
                | StringLiteral, "=", Expr
                | Expr, ":", Expr ;
ConstructionExpr = TypeExpr, KeyedLiteral ;
SwitchExpr      = "switch", Expr, NL, SwitchExprArm*, ";" ;
SwitchExprArm   = PatternList, [ "where", Expr ], "=>", Expr, NL
                | "else", "=>", Expr, NL ;
```

`Type { field = value }` constrói um valor de `Type`; campos obrigatórios sem default precisam ser informados, campos desconhecidos são erro. Um `KeyedLiteral` **sem** `TypeExpr` à esquerda (isto é, sem passar por `ConstructionExpr`) exige contexto de tipo alvo (binding tipado, parâmetro, campo) para ser resolvido; sem contexto de tipo é erro semântico de tipo ambíguo — esta gramática não define qual tipo concreto de biblioteca um `KeyedLiteral` "solto" produz (isso é responsabilidade de stdlib, fora de escopo deste documento), só fixa a regra de quando ele é aceitável.

## 10. Tipos

```ebnf
TypeExpr        = FunctionType ;
FunctionType    = PrimaryType, [ "->", TypeExpr ] ;
PrimaryType     = TypeName
                | QualifiedTypeName
                | PrimitiveType
                | OptionalType
                | PointerType
                | FixedArrayType
                | TupleType
                | RoutineType
                | "(", TypeExpr, ")" ;
QualifiedTypeName = identifier, { ".", identifier } ;
PrimitiveType   = "void" | "never" | "bool" | "number" | "string" |
                  "i8" | "i16" | "i32" | "i64" | "isize" |
                  "u8" | "u16" | "u32" | "u64" | "usize" ;
OptionalType    = PrimaryType, "?" ;
PointerType     = "*", [ "const" ], PrimaryType ;
FixedArrayType  = "[", TypeExpr, ";", ConstExpr, "]" ;
TupleType       = "(", TypeExpr, ",", TypeExpr, { ",", TypeExpr }, [ "," ], ")" ;
RoutineType     = [ "async" ], "routine", ParameterTypeList, ReturnAndThrows? ;
ParameterTypeList = "(", [ TypeExpr, { ",", TypeExpr }, [ "," ] ], ")" ;
TypeName        = identifier ;
BindingName     = identifier ;
```

**Adição:** `TypeName` e `BindingName` não existiam como produções (eram usadas em dezenas de lugares — `Uda`, `AttributeDecl`, `TypeDecl`, `EnumDecl`/`EnumVariant`, `ProfileDecl`, `FieldDecl`, `PropertyDecl`, `TypeParamItem` etc. — mas nunca declaradas). Ambas são `identifier`; a distinção PascalCase/lowerCamelCase é convenção de estilo (seção 1.1), não regra de parsing — o parser não rejeita um `TypeName` grafado em minúsculas.

`void` e `never` são palavras de tipo reservadas mesmo não sendo keywords léxicas hard em todos os contextos. Note que `PointerType`/`unsafe` (seção 7) e conversões numéricas ficam fora do escopo desta gramática de sintaxe — ver `language/03-types.md`.

## 11. ConstExpr

```ebnf
ConstExpr       = Expr ;
```

Uma `ConstExpr` é parseada como expressão comum e depois restrita semanticamente a literais, operações puras constantes, construção constante, acesso a constantes e parâmetros `static`.

---

## Apêndice A — Precedência de expressões (referência normativa para `BinaryExpr`/`PrefixExpr`)

Maior para menor:

| Nível | Operadores | Associatividade |
|---:|---|---|
| 1 | primários, literais, `()` | n/a |
| 2 | `.`, `?.`, call `()`, index `[]`, safe index `?[` | esquerda |
| 3 | prefix `+`, `-`, `*` (deref), `not`, `~`, `try`, `await` | direita |
| 4 | `**` | direita |
| 5 | `as`, `as?` | esquerda |
| 6 | `*`, `/`, `%` | esquerda |
| 7 | `+`, `-` | esquerda |
| 8 | `<<`, `>>`, `>>>` | esquerda |
| 9 | `&` | esquerda |
| 10 | `^` | esquerda |
| 11 | `|` | esquerda |
| 12 | `..`, `..=` | não encadeável sem parênteses |
| 13 | `<`, `<=`, `>`, `>=`, `<=>` | não encadeável sem `and` |
| 14 | `in` | esquerda |
| 15 | `==`, `!=` | não encadeável sem `and` |
| 16 | `and` | esquerda, short-circuit |
| 17 | `or` | esquerda, short-circuit |
| 18 | `??` | direita |
| 19 | `=>` (fora de `SwitchExprArm`) | direita |

Assignment não é expressão. `try await f()` é a forma canônica; `await try f()` é aceito pelo parser mas normalizado pelo formatter para `try await f()`.

Operadores sobrecarregáveis: `+ - * / % ** == != < <= > >= <=> & | ^ ~ << >> >>> [] []= () as in => TypeName` (a última forma é conversão implícita, seção 6.3).
Não sobrecarregáveis: `. ?. ?? = := ; :: and or not try await throw error return if repeat switch using`.
Deliberadamente fora desta versão: `++`/`--`.

## Apêndice B — Correções aplicadas nesta revisão

| # | Item | Fonte da correção |
|---|---|---|
| 1 | `Block` sem produção formal | inferido da própria estrutura da gramática atual + confirmado pelo rascunho arcaico (`grammar.md`: `Block ::= Newline Statement* ';'`) |
| 2 | `TypeParameters`, `TypeName`, `BindingName`, `FunctionName`, `LabelName`, `CallTarget`, `CallArgument`, `Assignable`, `IntegerType`, `Literal`, `BinaryOperator`, `PrefixOperator`, `AsOperator` sem produção formal | fechado a partir do próprio uso consistente na gramática atual; `TypeParam`/`StaticParam` com `in`/`out`/`static` inline foi confirmado pelo arcaico (`generics.md`, `routines.md#static-parameters`: `routine repeatStatic<static N: number>(...)`) |
| 3 | Token `?[]` já fechado, incompatível com a própria produção de indexação segura | corrigido para `?[` por análise interna (o arcaico usava outra sintaxe de segurança e não continha este bug) |
| 4 | `Metadata*` duplicado em `FileItem`/`DeclarationHead` | corrigido por análise interna |
| 5 | Setter de `property` sem forma de acessar o valor atribuído | resolvido com base no rascunho arcaico (`properties.md`: "A setter has one parameter named `value` unless explicitly named by grammar extension") |
| 6 | `where`/`as` em `repeat`: ordem textual sugere escopo invertido | clarificado com base no rascunho arcaico (`repeat.md`: "despite the grammar listing `where Expr` before `as Identifier` textually, the bound item is available inside the filter expression") |
| 7 | `RepeatSource = Expr \| RangeExpr` com `RangeExpr` nunca definida | resolvido por análise interna, removendo a alternativa redundante já que range é `BinaryOperator` |
| 8 | `OperatorName` sem produção (como declarar `operator []=`, `operator as T`, `operator in`?) | resolvido com base no rascunho arcaico (`operators.md`, seção "Declaration Syntax") |
| 9 | Índice `[...]`/`?[...]` reaproveitando `ArgumentList` (permitindo argumento nomeado em índice) | corrigido por análise interna, com nova produção `IndexArgumentList`; a nota "multi-key by `a[i, j]`" do arcaico (`operators.md`) confirma que índice é sempre posicional |
| 10 | `:=` em `local` sem diferença semântica de `=` | confirmado como ambiguidade também presente no arcaico (mesma duplicação existia lá); resolvido como sinônimo formal + normalização pelo formatter, não removido, para não quebrar fontes já escritas com `:=` |
| 11 | `KeyedLiteral` solto sem regra de quando é aceitável | regra portada do arcaico (`grammar.md`: "Ambiguous `{}` without type context is an error"), sem importar o tipo concreto (`Map`) que o arcaico atribuía, por ser stdlib |
| 12 | `=>` tinha sido classificado (nesta mesma revisão, versão anterior) como mecanismo de "endpoint binding" excluído — caracterização errada. **Correção:** `=>` é só mais um operador binário livre e sobrecarregável (`operator =>(input: T): R`), sem qualquer relação com ponto de entrada de programa | apontado pelo usuário; confirmado revendo o arcaico (`operators.md`: "`=>` is documented as a general overloadable operator... not syntax hardcoded to `std.start`" — o próprio arcaico já separava as duas coisas, e a versão anterior desta spec misturou-as por engano |
| 13 | Conjunto de operadores sobrecarregáveis pequeno demais | adicionados `<=>` (comparação tripla, com derivação automática de `<`/`<=`/`>`/`>=`), `*` unário (dereferência de tipo definido pelo usuário) e conversão implícita por `TypeName` sozinho (`operator bool()`) — os dois últimos vistos no arcaico (`operators.md`: `operator bool(): bool`); `<=>` é proposta nova, sem equivalente direto em nenhum dos dois zips, justificada pela necessidade (já registrada em `rascunho-organizar.md`) de consistência entre comparação e igualdade |
| 14 | `FieldDecl` colocava `$` entre `:` e `TypeExpr` (`radius: $number`), inconsistente com `Parameter`/`BindingPattern` (`$nome: T`) | corrigido por análise interna para `["$"], BindingName, ":", TypeExpr` — mesma posição nos três lugares |

## Apêndice C — Itens do rascunho arcaico deliberadamente não incorporados

Estes mecanismos existiam no rascunho arcaico e **não** foram trazidos, por serem de uma geração de design anterior e incompatível com as decisões já tomadas na spec atual:

- `module` / `use` / `include` / `global` / `exclude` — a spec atual resolve dependências inteiramente via `.nikoproj`/`niko.lock`, sem import em nível de source (`ai/navigation-guide.md`: "não restaurar source-level imports").
- `[entry]` e a ideia de amarrar `=>` a um ponto de entrada de programa — a spec atual usa `function main()` simples, configurável via `source.entry` em `.nikoproj`. Isso é diferente de `=>` em si: `=>` **foi** incorporado como operador sobrecarregável comum (seção 6.3), só sem o mecanismo de endpoint que o arcaico amarrava a ele.
- Patterns simples (`SwitchPattern = Expr | 'nil' | Binding`) — a spec atual já tem pattern matching estrutural mais rico (tupla, array, objeto, range, or-pattern) e isso foi mantido como está, sem regressão.
- Parâmetros sem default/variádico/nomeado no `ParamList` — a spec atual já suporta os três e isso foi mantido.
- Qualquer tipo ou API de biblioteca padrão (`List<T>`, `Map<K,V>`, `Iterable<T>`, `Disposable`, etc.) — fora de escopo por definição do próprio README da spec atual.
