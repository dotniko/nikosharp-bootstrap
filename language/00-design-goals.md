# 00. Objetivos da linguagem

NikoSharp é uma linguagem AOT, produtiva, limpa, rápida e previsível. Ela deve favorecer um único estilo idiomático, formatter oficial e LSP simples de implementar.

## Princípios normativos

1. Uma construção deve ter uma forma canônica clara.
2. Sintaxe curta só é aceita quando melhora legibilidade e tooling.
3. Nenhuma feature pode depender de reflexão em runtime.
4. Metadados são compile-time-only por padrão.
5. O compilador deve conseguir diagnosticar erros localmente sempre que possível.
6. Recursos avançados devem baixar para contratos explícitos, não para mágica dinâmica.
7. Assignment não é expressão.
8. Blocos longos fecham com `;`; statements comuns não usam `;`.
9. `self.` é obrigatório em membros de instância.
10. O código comum não exige cerimônia manual de ownership.

## Não objetivos

- interpretação dinâmica de source como identidade da linguagem;
- runtime reflection;
- macro system geral;
- várias sintaxes equivalentes para a mesma coisa;
- binder automático arbitrário para C++;
- GC global obrigatório;
- overload mágico que o formatter ou LSP não consigam explicar.

## Influências

A linguagem usa a ideia de metadados declarativos de C#, reduzindo ruído de cabeçalhos e evitando anotações verbosas. Também herda de D a preferência por AOT, performance e extensibilidade estática, mas recusa a proliferação de formas alternativas que dificultam ferramentas.
