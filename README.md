# NikoSharp Bootstrap Specification

Esta pasta define a especificação bootstrap de NikoSharp. Ela é normativa para implementações compatíveis da linguagem, da gramática textual `.ns` e dos artefatos mínimos necessários para construir, empacotar e consumir código compilado.

O objetivo é congelar a superfície da linguagem em um formato pequeno, limpo e implementável, inspirado pela produtividade de D e pela clareza de metadados de C#, mas evitando verbosidade, múltiplas formas equivalentes e semântica difícil de manter por tooling.

## Escopo

Este pacote especifica:

- arquivos `.ns`;
- léxico, gramática, declarações, expressões, tipos, statements e formatação;
- UDAs, User Defined Attributes;
- UDMs, User Defined Modifiers;
- tipos primitivos e regras de conversão;
- overload, generics, profiles, routines, async, erros, patterns, unsafe, memória e concorrência;
- `.nikolib`, artefato compilado autocontido equivalente ao papel de um `.dll`;
- `.nikoproj`, `.nikosln`, `niko.lock` e CLI `niko`;
- diagnósticos mínimos e checklist de implementação.

Este pacote não especifica catálogo de biblioteca, APIs oficiais externas, runtime interno, backend, layout interno de IR binário ou implementação de compilador.

## Decisão inicial

A especificação não ignora totalmente a plataforma, porque o bootstrap precisa saber como um pacote compilado é nomeado, assinado, referenciado e consumido. Porém, ela limita a parte de plataforma aos artefatos textuais e ao contrato de `.nikolib`. O payload intermediário dentro de `.nikolib` é obrigatório, mas opaco para esta especificação.

A decisão de `.nikolib` é autocontida: metadados e payload ficam no mesmo arquivo. Um payload textual opcional pode existir para depuração, mas o pacote compilado válido carrega o payload binário obrigatório junto dos metadados.

## Organização

```text
language/       especificação da linguagem .ns
artifacts/      formatos de arquivo e CLI
examples/       exemplos pequenos e completos
appendix/       tabelas normativas
ai/             guia de navegação para agentes
```
