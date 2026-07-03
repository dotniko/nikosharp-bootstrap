# `.nikosln`

`.nikosln` agrupa projetos. Formato bootstrap: TOML.

```toml
format = "nikosln"
format_version = 1
name = "workspace"

[[projects]]
path = "app/app.nikoproj"

[[projects]]
path = "lib/lib.nikoproj"
```

## Regras

- Paths são relativos ao arquivo `.nikosln`.
- Ciclos entre projetos são erro.
- O comando `niko build` no arquivo de solução constrói em ordem topológica.
- Configurações do projeto vencem defaults da solução.
