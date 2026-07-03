# `.nikoproj`

`.nikoproj` descreve um projeto compilável. O formato bootstrap é TOML.

## Exemplo

```toml
format = "nikoproj"
format_version = 1
name = "hello"
version = "0.1.0"
language = "nikosharp"
kind = "app"

[source]
root = "src"
include = ["**/*.ns"]
exclude = ["**/*.generated.ns"]
entry = "main"

[output]
path = "build"

[target]
default = "generic"

[dependencies]
# name = { path = "../name" }
# name = { version = "1.2.3" }
```

## Campos

| Campo | Obrigatório | Significado |
|---|---:|---|
| `format` | sim | sempre `nikoproj` |
| `format_version` | sim | versão do formato |
| `name` | sim | identidade do pacote |
| `version` | sim | versão semântica |
| `language` | sim | `nikosharp` para estes sources |
| `kind` | sim | `app`, `library`, `tool` |
| `source.root` | sim | diretório de source |
| `source.include` | sim | globs de `.ns` |
| `source.exclude` | não | globs ignorados |
| `source.entry` | não | função de entrada |
| `dependencies` | não | dependências por path, git ou registry |

## Regras

- O projeto, não o source, define dependências externas.
- A ordem dos arquivos não pode alterar semântica.
- `niko.lock` congela resolução de dependências.
- Um projeto pode produzir `.nikolib`, executável ou ambos.
