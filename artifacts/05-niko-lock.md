# `niko.lock`

`niko.lock` congela dependências resolvidas.

## Exemplo

```toml
format = "niko.lock"
format_version = 1

[[package]]
name = "example"
version = "1.0.0"
source = "registry"
hash = "sha256-..."

[[package]]
name = "local-tool"
version = "0.1.0"
source = "path:../local-tool"
hash = "sha256-..."
```

## Regras

- `niko restore` cria ou atualiza o lock.
- `niko build --locked` falha se o lock estiver ausente ou desatualizado.
- O lock deve ser determinístico.
- Entradas são ordenadas por nome, versão e source.
- Hash cobre conteúdo normalizado do pacote ou commit resolvido.
