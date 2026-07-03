# Guia para agentes

Leia nesta ordem:

1. `README.md`
2. `language/00-design-goals.md`
3. `language/01-lexical-structure.md`
4. `language/02-complete-grammar.md`
5. `language/08-uda-udm.md`
6. `artifacts/02-nikolib.md`
7. `artifacts/03-nikoproj.md`
8. `artifacts/06-cli-niko.md`
9. `appendix/bootstrap-checklist.md`

Regras de trabalho:

- não adicionar nova sintaxe;
- não restaurar source-level imports;
- não documentar catálogo de biblioteca;
- não transformar UDM em keyword dinâmica;
- não tratar UDA/UDM como runtime reflection;
- não separar `.nikolib` em metadados e payload externos obrigatórios;
- manter parser determinístico e formatter canônico.
