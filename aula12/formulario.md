# Formulário de Referência — UC00652
**Avaliação Final · 1 página · Consulta permitida**

---

## Método MUX — FSM com 2 Flip-Flops D

### Passo 1 — Estrutura
- **n estados** → **k Flip-Flops D**, onde 2^k ≥ n
- 2 estados → 1 FF &nbsp;|&nbsp; 4 estados → 2 FF &nbsp;|&nbsp; 8 estados → 3 FF

### Passo 2 — Sinal AVANÇA
- AVANÇA = condição que faz avançar o estado
- Quando **AVANÇA = 0** → estado mantém-se (self-loop): `Q_next = Q_actual`
- Quando **AVANÇA = 1** → estado avança para o próximo

### Passo 3 — Equação D (Método MUX)

```
D  =  AVANÇA' · Q_actual  +  AVANÇA · Q_next
```

> **Notação:** `AVANÇA'` = `!(AVANÇA)` — o complemento (NOT) de AVANÇA.
>
> **O que é AVANÇA?** É o sinal que representa **todas as condições** (botões, sensores, timer, etc.) que fazem a máquina de estados avançar para o estado seguinte.
> - `AVANÇA = 0` → condições não satisfeitas → estado mantém-se (self-loop: D = Q_actual)
> - `AVANÇA = 1` → condições satisfeitas → estado avança (D = Q_next)

### Passo 4 — Saídas Moore
- Dependem **apenas do estado** (Q1, Q0)
- **Nunca** dependem das entradas

---

## Padrões frequentes de Q_next

| Q_next | Padrão | Expressão |
|--------|--------|-----------|
| XOR(Q1,Q0) | `01→10→11→00→01` | Q1⊕Q0 |
| !Q0 | Q0 alterna sempre | !Q0 |
| Q1 | Q1 não muda | Q1 |
| !Q1·!Q0 | Só activo em 00 | !Q1 · !Q0 |
| Q1·Q0 | Só activo em 11 | Q1 · Q0 |

---

## Portas lógicas usadas

| Função | Expressão | Chip |
|--------|-----------|------|
| AND 2 entradas | A · B | 7408 |
| OR 2 entradas | A + B | 7432 |
| NOT | !A | 7404 |
| NAND | !(A·B) | 7400 |
| XOR | A⊕B | 7486 |

---

## Chips de apoio

### 74HC74 — Dual D Flip-Flop

```
         ┌──────────┐
  !CLR1 ─┤1       14├─ VCC
     D1 ─┤2       13├─ !CLR2
    CLK1─┤3       12├─ D2
   !PR1 ─┤4       11├─ CLK2
     Q1 ─┤5       10├─ !PR2
    !Q1 ─┤6        9├─ Q2
    GND ─┤7        8├─ !Q2
         └──────────┘
```

**Convenção UC00652:**
- **FF1** = Q0 da FSM → pin 5 = Q0 · pin 6 = !Q0
- **FF2** = Q1 da FSM → pin 9 = Q1 · pin 8 = !Q1
- !CLR e !PR → ligar a **VCC** (HIGH = inactivo)
- Dispara na **subida** do clock ↑

### 74HC157 — Quad MUX 2:1

```
Método MUX:   SEL = AVANÇA
              Entrada A = Q_actual
              Entrada B = Q_next
              Saída = D
```

| SEL | Saída |
|:---:|-------|
| 0   | Entrada A |
| 1   | Entrada B |

---

## Álgebra Booleana — Identidades úteis

| Identidade | Expressão |
|------------|-----------|
| AND com 1 | A · 1 = A |
| OR com 0 | A + 0 = A |
| Complemento | A · !A = 0 &nbsp;·&nbsp; A + !A = 1 |
| Absorção | A + A·B = A |
| De Morgan | !(A·B) = !A + !B |
| De Morgan | !(A+B) = !A · !B |
| XOR | A⊕B = A·!B + !A·B |

---

*UC00652 · Escola Sicó · 2025–2026*
