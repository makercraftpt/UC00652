# Anexo — 74HC4020: Contador Binário como Temporizador de FSM

**UC00652 · Complemento à Aula 11**

---

## O que é o 74HC4020?

O **74HC4020** é um contador binário ripple de **14 andares**. Cada andar divide a frequência do clock por 2.

```
CLK → [÷2] → Q1 → [÷2] → Q2 → [÷2] → Q3 → … → Q14
```

Se o clock tiver frequência **f**, cada saída Q_n tem frequência **f / 2ⁿ**, ou seja, período de **2ⁿ / f** segundos.

---

## Pinagem (DIP-16)

```
        ┌────┬────┐
   Q12 ─┤ 1  │ 16 ├─ Q1   (não acessível em alguns packages)
   Q13 ─┤ 2  │ 15 ├─ GND
   Q14 ─┤ 3  │ 14 ├─ Q3
   CLK ─┤ 4  │ 13 ├─ Q4
   CLR ─┤ 5  │ 12 ├─ Q5
   VCC ─┤ 6  │ 11 ├─ Q6
   Q10 ─┤ 7  │ 10 ├─ Q7
    Q8 ─┤ 8  │  9 ├─ Q9
        └────────┘
```

| Pino | Função |
|:----:|--------|
| CLK | Entrada de clock — dispara na descida ↓ |
| CLR | Reset assíncrono — activo a **HIGH** · coloca todos os Q a 0 |
| VCC | 5V |
| GND | 0V |
| Q3…Q14 | Saídas dos andares (Q1 e Q2 **não estão disponíveis** no DIP-16) |

> ⚠️ O 74HC4020 dispara na **descida** do clock (falling edge). Se o 555 ou o Arduino gerarem um pulso LOW→HIGH→LOW, o contador avança na descida.

---

## Como funciona — tabela de taps

Com clock a **4 Hz** (período = 0,25 s):

| Saída | Divisão | Período | Frequência |
|:-----:|:-------:|:-------:|:----------:|
| Q3    | ÷ 8     | 2,0 s   | 0,5 Hz     |
| Q4    | ÷ 16    | 4,0 s   | 0,25 Hz    |
| Q5    | ÷ 32    | 8,0 s   | 0,125 Hz   |
| Q6    | ÷ 64    | 16 s    | —          |

Com clock a **8 Hz** (período = 0,125 s):

| Saída | Período | Uso típico |
|:-----:|:-------:|-----------|
| Q3    | 1,0 s   | ≈ AMARELO |
| Q4    | 2,0 s   | — |
| Q5    | 4,0 s   | ≈ VERDE / VERMELHO |

> **Regra prática:** escolher o clock de modo a que Q3 dê o tempo mais curto pretendido e Q_(3+n) dê o tempo mais longo.

---

## Aplicação ao Semáforo FSM

### Tempos pretendidos
- VERDE → **3 s** (longo)
- AMARELO → **1 s** (curto)
- VERMELHO → **3 s** (longo)

Rácio longo/curto = 3 → aproximar para **4 = 2²** (diferença de 2 andares).

### Configuração
| Parâmetro | Valor |
|-----------|-------|
| Clock (555) | **4 Hz** (T = 0,25 s) |
| Tap AMARELO | Q2* (= 0,25 × 4 = 1 s) |
| Tap VERDE/VERMELHO | Q4 (= 0,25 × 16 = 4 s) |

> *Q2 não acessível no DIP-16 — usar Q3 (2 s) e ajustar o 555 para 2 Hz, ou usar Q4 para VERDE (4 s) e Q3 para AMARELO (1 s) com clock 2 Hz. Alternativa: usar um 74HC4040 que disponibiliza Q1–Q12.

### Com clock a 2 Hz:

| Tap | Período | Estado |
|:---:|:-------:|--------|
| Q3  | 4 s     | → VERDE / VERMELHO (≈ ok) |
| Q2  | 2 s     | → (não acessível) |
| Q4  | 8 s     | (demasiado longo) |

**Compromisso prático:** clock 2 Hz → Q3 = 4 s para VERDE/VERMELHO · substituir AMARELO por Q1 do Arduino (não disponível no 4020) **ou** usar o 74HC4040.

---

## Selecção por estado — 74HC157

O **74HC157** é um MUX quad 2:1. Usado aqui para selecionar o tap certo conforme o estado actual da FSM.

```
Estado     Q1  Q0    Tap desejado   SEL
─────────────────────────────────────────
VERDE       0   0    Q4 (lento)     Q0 = 0
AMARELO     0   1    Q3 (rápido)    Q0 = 1
VERMELHO    1   0    Q4 (lento)     Q0 = 0
```

Ligação ao 74HC157:
```
SEL (74HC157)  →  Q0 da FSM (pino 5 do 74HC74)
Entrada A      →  Q4 do 74HC4020  (tap lento)
Entrada B      →  Q3 do 74HC4020  (tap rápido)
Saída          →  AVANÇA
```

Equação: **AVANÇA = Q0'·Q4 + Q0·Q3**

---

## Reset automático do contador

Quando o timer dispara (AVANÇA = 1), o contador deve recomeçar do zero para a contagem do próximo estado.

```
AVANÇA ──┬──▶ CLK do 74HC74  (avança o estado da FSM)
          └──▶ CLR do 74HC4020  (reset do contador → AVANÇA volta a 0)
```

Este **auto-reset** cria um pulso muito curto:
1. Contador atinge Q_n = 1 → AVANÇA = 1
2. CLR = 1 → contador reset → Q_n = 0 → AVANÇA = 0
3. Contador começa a contar de novo para o novo estado

> ⚠️ O pulso é muito curto (≈ 1 propagation delay). Para garantir que o 74HC74 captura a transição, adicionar um **AND** entre AVANÇA e o clock do 555:
> ```
> AVANÇA_seguro = AVANÇA_raw AND Clock
> ```
> Assim o pulso só acontece quando o clock está activo, garantindo uma duração mínima de meio período.

---

## Diagrama completo

```
  555 (astable)
     │ f = 2 Hz
     ├──────────────────────────────▶ CLK do 74HC74 (flip-flops)
     │
     ▼
  74HC4020 (contador)
     ├── Q3  ──▶ 74HC157 entrada B  (rápido)
     ├── Q4  ──▶ 74HC157 entrada A  (lento)
     └── CLR ◀── AVANÇA (reset automático)

  74HC74 (FSM)
     ├── Q0 (pin 5)  ──▶ 74HC157 SEL
     └── Q1 (pin 9)  ──▶ (lógica de saída LEDs)

  74HC157
     └── Saída ──▶ AVANÇA ──┬──▶ CLK da FSM (ou SEL do MUX D)
                              └──▶ CLR do 74HC4020
```

---

## Comparação com o Arduino (Aula 10/11)

| | Arduino (Aulas 10-11) | 74HC4020 + 74HC157 (hardware puro) |
|---|---|---|
| Clock | Arduino gera pulso após delay() | 555 corre sempre a f constante |
| Timer | Software (delay por estado) | Hardware (tap do contador) |
| AVANÇA | = CLK (sempre avança) | Sinal separado — só ativa quando o timer esgota |
| Precisão | ±1 ms (millis) | Depende da precisão do 555 |
| Complexidade | Baixa (só código) | Média (chips extra) |
| Realismo industrial | Simulado | Próximo do circuito real |

---

*UC00652 · Escola Sicó · Formador: Belmiro Simões Luís · 2025–2026*
