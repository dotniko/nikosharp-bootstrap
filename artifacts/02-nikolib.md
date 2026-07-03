# `.nikolib`

`.nikolib` é o artefato compilado autocontido de Niko. Ele cumpre papel equivalente ao de um `.dll`: carrega metadados, identidade, dependências, declarações públicas e payload intermediário compilado.

## Decisão

O `.nikolib` deve conter o payload intermediário no mesmo arquivo. Isso evita bibliotecas quebradas por arquivo lateral ausente e permite cache, assinatura, restore e inspeção determinística. Para depuração, um payload textual opcional pode ser incluído junto, mas o payload binário é obrigatório.

## Container

Bootstrap usa ZIP determinístico:

- entradas ordenadas lexicograficamente;
- timestamps normalizados para `1980-01-01T00:00:00Z`;
- nomes com `/`;
- compressão `deflate` ou `store`;
- nenhum path absoluto;
- nenhum `..`.

Estrutura:

```text
nikolib.toml
metadata/declarations.json
metadata/attributes.json
metadata/modifiers.json
metadata/dependencies.json
metadata/public.txt
ir/payload.nirb
ir/payload.nir          # opcional, debug
symbols/debug.json      # opcional
docs/docs.json          # opcional
signatures/content.sha256
```

## `nikolib.toml`

```toml
format = "nikolib"
format_version = 1
name = "example"
version = "0.1.0"
language = "nikosharp"
language_version = "0.1"
abi_version = "niko-abi-1"
ir_version = "niko-ir-1"
target = "generic"
kind = "library"

[content]
metadata = "metadata/declarations.json"
attributes = "metadata/attributes.json"
modifiers = "metadata/modifiers.json"
dependencies = "metadata/dependencies.json"
public = "metadata/public.txt"
payload = "ir/payload.nirb"
```

## Declarações

`metadata/declarations.json` contém tabela estável:

```json
{
  "schema": "niko.declarations.v1",
  "items": [
    {
      "id": "fn:main",
      "kind": "function",
      "name": "main",
      "visibility": "public",
      "signature": {
        "parameters": [],
        "return": "i32",
        "throws": null,
        "async": false
      },
      "attributes": [],
      "modifiers": []
    }
  ]
}
```

IDs são estáveis dentro do artefato, mas não precisam ser paths de source.

## Attributes

```json
{
  "schema": "niko.attributes.v1",
  "declared": [
    {
      "name": "Version",
      "fields": [ { "name": "value", "type": "u32", "default": null } ]
    }
  ],
  "applied": [
    {
      "target": "fn:help",
      "name": "Version",
      "arguments": [1]
    }
  ]
}
```

## Modifiers

```json
{
  "schema": "niko.modifiers.v1",
  "declared": [
    { "name": "command", "targets": ["function"] }
  ],
  "applied": [
    { "target": "fn:help", "name": "command" }
  ]
}
```

## Dependências

```json
{
  "schema": "niko.dependencies.v1",
  "items": [
    {
      "name": "other",
      "version": "1.2.3",
      "hash": "sha256-...",
      "public": true
    }
  ]
}
```

## Public surface

`metadata/public.txt` é representação textual estável para revisão humana e snapshots. Exemplo:

```text
function main(): i32
attribute Version(value: u32)
modifier command: function
command function help(): CommandResult
```

## Payload

`ir/payload.nirb` é obrigatório e opaco para esta especificação. Ele deve corresponder aos metadados públicos. O validador de artefato deve rejeitar divergência entre metadados e payload.

## Hash e assinatura

`signatures/content.sha256` lista hash SHA-256 de cada entrada, exceto ele mesmo. Assinatura criptográfica externa pode ser adicionada depois sem alterar a gramática da linguagem.
