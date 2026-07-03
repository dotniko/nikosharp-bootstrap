# CLI `niko`

A CLI `niko` é a interface oficial de build, inspeção e empacotamento.

## Comandos

```text
niko --version
niko help [command]
niko new <template> <name>
niko check [path] [--locked]
niko build [path] [--release] [--target <target>] [--locked]
niko run [path] [-- args...]
niko test [path] [--filter <pattern>]
niko fmt [path] [--check]
niko doc [path] [--output <dir>]
niko restore [path] [--locked]
niko pack [path] [--output <dir>]
niko inspect <file.nikolib> [--json]
niko metadata <file.nikolib>
niko clean [path]
niko publish <file.nikolib> [--dry-run]
```

## `check`

Executa parsing, type checking, análise semântica e validação de artefatos sem produzir binário final.

## `build`

Produz outputs definidos pelo projeto. Para libraries, produz `.nikolib`. Para apps, pode produzir executável e `.nikolib` intermediário.

## `run`

Compila se necessário e executa a entrada configurada.

## `fmt`

Formata `.ns`, `.nikoproj`, `.nikosln` e `niko.lock` quando aplicável. `--check` falha se alteração seria necessária.

## `inspect`

Lista manifesto, metadados públicos, UDAs, UDMs, dependências e hashes de `.nikolib`. Não precisa entender o payload opaco para mostrar metadados.

## Exit codes

| Código | Significado |
|---:|---|
| 0 | sucesso |
| 1 | erro de uso da CLI |
| 2 | erro de parsing/sintaxe |
| 3 | erro semântico/type checking |
| 4 | erro de dependência/lock |
| 5 | erro de build/backend |
| 6 | teste falhou |
| 7 | artefato inválido |
