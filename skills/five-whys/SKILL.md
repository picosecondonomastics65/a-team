---
name: five-whys
description: Análise de causa raiz com a técnica dos Cinco Porquês. Usar quando há um bug persistente, falha de processo, ou problema recorrente que os fixes superficiais não resolvem. Garante que o fix ataca a raiz, não o sintoma.
---

# Five Whys — Análise de Causa Raiz

## Regra de Ouro

```
NÃO FIXES O SINTOMA. ENCONTRA A CAUSA.
```

Um fix que não ataca a raiz é um fix temporário. A causa volta.

## Protocolo

### Passo 1 — Definir o problema com precisão

Escreve uma frase curta e concreta. Não vaga.

| Errado | Correto |
|--------|---------|
| "O sistema está lento" | "A API /orders demora >3s para 90% dos requests em produção desde sexta" |
| "Os testes falham às vezes" | "test_payment_flow falha em CI uma vez por ~20 runs sem mensagem de erro clara" |

### Passo 2 — Cinco Porquês em cascata

Para cada resposta, pergunta: **"Por que é que isso acontece?"**

Não pares antes de chegar a algo que podes controlar — uma decisão, um processo, uma linha de código, uma configuração ausente.

```
Problema: Utilizadores vêem dados de outro utilizador
  Porquê 1: A cache devolve uma entrada errada
  Porquê 2: A chave de cache não inclui o user_id
  Porquê 3: O developer assumiu que o endpoint era público
  Porquê 4: Não havia test de isolamento por utilizador
  Porquê 5: O processo de code review não verifica isolamento de dados
  → Causa raiz: processo de review sem checklist de segurança
```

### Passo 3 — Validar em sentido inverso

Lê a cadeia de baixo para cima: "Porque não temos checklist de review → dados de cache sem user_id → leak de dados". Se a cadeia faz sentido, a causa raiz é válida.

### Passo 4 — Fix na causa raiz, não no sintoma

Implementa a solução no nível mais fundo que for praticável.

| Nível do fix | Exemplo | Duração |
|---|---|---|
| Sintoma | Limpar a cache manualmente | Horas |
| Causa próxima | Adicionar user_id à chave | Dias |
| Causa raiz | Adicionar checklist de isolamento ao review | Permanente |

Quando só consegues fixar um nível intermédio, documenta o nível raiz como dívida técnica com um TODO rastreável.

## Ramificações

Uma causa raiz pode ter múltiplos ramos — explora todos antes de escolher onde fixar.

```
Problema: Deploy falha em produção mas não em staging
  Ramo A: configuração diferente (env vars, secrets)
  Ramo B: dados de produção com volume/formato diferente
  Ramo C: dependências de rede só disponíveis em prod
```

Testa cada ramo independentemente. Não assumas.

## Quando Parar

Para antes do 5.º "porquê" se chegares a uma causa raiz clara e accionável. Para depois do 5.º se a cadeia não convergiu — pode haver múltiplas causas raiz independentes.

## Output Esperado

No final da análise, documenta:

```markdown
**Problema:** [frase concreta]
**Causa raiz:** [o nível mais fundo encontrado]
**Fix proposto:** [acção concreta com ficheiro/componente]
**Fix temporário (se necessário):** [o que aplicas agora enquanto o fix real não está pronto]
**Dívida técnica:** [o que ficou por resolver e porquê]
```

## Integração com A Team

- Usa esta skill **antes** de lançar o `systematic-debugging` agent — os Cinco Porquês definem o escopo da investigação
- Após encontrar a causa raiz, usa o `tdd-guide` agent para escrever um test que a teria capturado
- Documenta a análise em `DAILY.md` se a causa raiz implicar mudança de processo
