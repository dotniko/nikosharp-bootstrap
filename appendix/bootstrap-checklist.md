# Checklist bootstrap

Uma implementação bootstrap compatível deve entregar, nesta ordem:

1. lexer UTF-8 com trivia preservada;
2. parser da gramática completa;
3. AST ou árvore sintática lossless;
4. resolver de nomes sem depender da ordem física dos arquivos;
5. type checker com optional, generics, profiles e overload;
6. validação de UDA e UDM;
7. formatter canônico;
8. emissão de metadados `.nikolib`;
9. payload intermediário opaco em `ir/payload.nirb`;
10. leitura de `.nikoproj`, `.nikosln` e `niko.lock`;
11. CLI `niko check`, `build`, `fmt`, `pack`, `inspect`;
12. diagnósticos mínimos definidos neste pacote.

Não adicionar novas formas sintáticas antes de concluir esses itens.
