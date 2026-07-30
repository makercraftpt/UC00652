# Plano — Aula 12: Revisão + Avaliação Final
**UC00652 · 27 Jul 2026 · 2 horas**

---

## Estrutura da sessão

| Hora | Bloco | Duração |
|------|-------|:-------:|
| 1ª hora | Revisão guiada | 60 min |
| 2ª hora | Avaliação Final | 60 min |

---

## Hora 1 — Revisão Guiada

### Abertura (5 min)
- Distribuir o **formulário de referência** (1 página)
- Avisar: o teste usa uma FSM nova (não semáforo nem portão)
- Relembrar o que é permitido: só o formulário + caneta/lápis

---

### Revisão: Os 4 passos do Método MUX (15 min)

Escrever no quadro e comentar cada passo:

```
PASSO 1 — Diagrama de estados
  Círculos = estados (codificados em binário)
  Setas = transições com condição de AVANÇA

PASSO 2 — Tabela de transições
  Linha por estado: Q_actual | AVANÇA | Q1_next | Q0_next | Saídas

PASSO 3 — Equações D pelo Método MUX
  D = AVANÇA' · Q_actual + AVANÇA · Q_next

  D1 — padrão universal: D1 = AVANÇA'·Q1 + AVANÇA·XOR(Q1,Q0)
  D0 — depende da FSM (calcular da tabela)

PASSO 4 — Circuito
  74HC74 (FF-D) + portas lógicas + 74HC157 (MUX opcional)
```

**Erros mais frequentes — referir explicitamente:**
- Esquecer os self-loops na tabela (quando AVANÇA=0, Q_next = Q_actual)
- Confundir D1_next com D1 final (não esquecer o AVANÇA')
- !CLR e !PR do 74HC74 sem ligação ao VCC → chip em reset
- Fio D2→pin5 trocado com Gate1→pin2 (bug da aula 10/11)

---

### Exemplo guiado — Torniquete (20 min)

Resolver em conjunto no quadro. **Não é o tema do teste.**

**Sistema:** torniquete de metro
- **BLOQUEADO (00):** ninguém passa
- **DESBLOQUEADO (01):** passa uma pessoa
- **Entradas:** MOEDA · EMPURRAR

| Estado | AVANÇA | Q_next |
|--------|--------|--------|
| BLOQUEADO (00) | MOEDA | DESBLOQUEADO (01) |
| DESBLOQUEADO (01) | EMPURRAR | BLOQUEADO (00) |

*(FSM simples de 2 estados — só para aquecer o método)*

Perguntas a colocar aos alunos:
1. "Quantos FF-D precisamos?"
2. "Qual é a expressão de AVANÇA em BLOQUEADO?"
3. "Qual é D0_next quando AVANÇA=1 em DESBLOQUEADO?"

---

### Mini-exercício individual (15 min)

Colocar no quadro (ou distribuir folha):

> **FSM de Semáforo Pedonal** — 3 estados:
> VERDE_PEAO(00) · PISCA_PEAO(01) · VERMELHO_PEAO(10)
>
> Preencher apenas a **tabela de transições** e a **expressão de AVANÇA**.

Circular pela sala e corrigir em tempo real.

---

### Erros comuns + dúvidas finais (5 min)

Lista rápida no quadro:
- ⚠️ Confirmar que a soma de D1 tem AVANÇA' (não esquecer o complement)
- ⚠️ Self-loop: quando AVANÇA=0, D = Q (estado mantém-se)
- ⚠️ AVANÇA diferente por estado — não é sempre a mesma condição
- ⚠️ Saídas Moore: dependem **só** do estado, nunca das entradas

---

## Hora 2 — Avaliação Final (60 min)

### Procedimento
1. Distribuir enunciado (ver ficheiro `UC00652_Avaliacao_Final.md`)
2. Alunos podem usar o formulário de referência (1 página)
3. Proibido: consulta de apontamentos, telemóvel, falar com colega
4. Recolher às 60 minutos exactos

### Cotações
| Exercício | Descrição | Pontos |
|:---------:|-----------|:------:|
| 1 | Diagrama de estados | 20 |
| 2 | Tabela de transições | 25 |
| 3 | Expressão de AVANÇA | 20 |
| 4 | Equações D1 e D0 (Método MUX) | 25 |
| 5 | Saídas Moore | 10 |
| **Total** | | **100** |

### Critérios de avaliação
- **Exercício 1:** 5 pts por estado bem codificado + transições corretas
- **Exercício 2:** 2 pts por linha correta (4 estados × 2 colunas next)
- **Exercício 3:** 5 pts por expressão de AVANÇA por estado (4 estados)
- **Exercício 4:** D1 correto (12 pts) + D0 correto (13 pts)
- **Exercício 5:** 3-4 pts por saída correcta (3 saídas)

---

## Formulário de referência (distribuir com o teste)

Ver ficheiro: `formulario.md`

---

*UC00652 · Escola Sicó · 2025–2026 · Belmiro Simões Luís*
