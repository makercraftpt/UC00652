# Revisões TP11 — FSM Semáforo com Chips TTL
**UC00652 · Aula 11 · Breadboard Física**

---

## 1. Definição da Máquina de Estados

### 1.1 Estados

O semáforo tem **3 estados** codificados em 2 bits (Q1 Q0):

| Estado | Q1 | Q0 | LED activo |
|:------:|:--:|:--:|-----------|
| VERDE | 0 | 0 | Verde |
| AMARELO | 0 | 1 | Amarelo |
| VERMELHO | 1 | 0 | Vermelho |
| *(inválido)* | 1 | 1 | — |

> **Porquê 2 bits?** Com 1 bit só representamos 2 estados. Com 2 bits temos 2² = 4 combinações → chega para 3 estados. O estado `11` nunca é usado (don't care — X).

### 1.2 Entradas e saídas

| Tipo | Sinal | Descrição |
|------|-------|-----------|
| **Clock** | CLK | Gerado pelo Arduino — só dispara quando é hora de avançar |
| **Saída** | LED_VERDE | Activo quando Q1Q0 = 00 |
| **Saída** | LED_AMARELO | Activo quando Q1Q0 = 01 |
| **Saída** | LED_VERMELHO | Activo quando Q1Q0 = 10 |

> Não há sinal `AVANÇA` separado — o Arduino só gera o pulso de clock quando o tempo do estado acabou. Isto é uma escolha de implementação com consequências importantes (ver secção 5).

### 1.3 Sequência de estados

```
         CLK              CLK              CLK
  VERDE ──────▶ AMARELO ──────▶ VERMELHO ──────▶ VERDE
  (00)           (01)             (10)             (00)
```

A sequência é sempre a mesma: 00 → 01 → 10 → 00 → …

---

## 2. Tabela de Transições

Como o Arduino só dispara o clock quando é hora de avançar, **cada pulso de clock avança sempre o estado**. Não há self-loops na tabela.

| Estado actual | Q1 | Q0 | Estado seguinte | Q1_next | Q0_next |
|:-------------:|:--:|:--:|:---------------:|:-------:|:-------:|
| VERDE | 0 | 0 | AMARELO | 0 | 1 |
| AMARELO | 0 | 1 | VERMELHO | 1 | 0 |
| VERMELHO | 1 | 0 | VERDE | 0 | 0 |
| *(inválido)* | 1 | 1 | — | X | X |

### Tabela completa de D

D1 e D0 são os valores que os Flip-Flops recebem. Na subida do CLK, Q1←D1 e Q0←D0.

| Q1 | Q0 | D1 | D0 |
|:--:|:--:|:--:|:--:|
| 0  | 0  | 0  | 1  |
| 0  | 1  | 1  | 0  |
| 1  | 0  | 0  | 0  |
| 1  | 1  | X  | X  |

---

## 3. Expressões Booleanas de D1 e D0

### 3.1 Mapa de Karnaugh — D1

```
         Q0
      0     1
    ┌─────┬─────┐
Q1 0│  0  │  1  │
    ├─────┼─────┤
   1│  0  │  X  │
    └─────┴─────┘
```

Grupo possível: célula (Q1=0,Q0=1) = 1 e célula (Q1=1,Q0=1) = X

Se incluirmos o **don't care** X como 1:

```
Grupo de 2 colunas Q0=1: D1 = Q0
```

**D1 = Q0** *(simplificado com don't care)*

Sem usar o don't care: D1 = !Q1 · Q0 (mais conservador, mas requer porta AND)

### 3.2 Mapa de Karnaugh — D0

```
         Q0
      0     1
    ┌─────┬─────┐
Q1 0│  1  │  0  │
    ├─────┼─────┤
   1│  0  │  X  │
    └─────┴─────┘
```

Apenas a célula (Q1=0, Q0=0) = 1.  
O don't care (1,1)=X não ajuda a criar um grupo maior com esta célula sozinha.

**D0 = !Q1 · !Q0**

### 3.3 Resumo das expressões

```
D1 = Q0              (fio directo de Q0 para D1)
D0 = !Q1 · !Q0       (porta AND com as negações de Q1 e Q0)
```

---

## 4. Implementação no Circuito

### 4.1 Pinos do 74HC74 (convenção UC00652)

| Sinal FSM | Pino 74HC74 | Descrição |
|:---------:|:-----------:|-----------|
| Q0 | 5 | FF1 saída directa |
| !Q0 | 6 | FF1 saída negada |
| Q1 | 9 | FF2 saída directa |
| !Q1 | 8 | FF2 saída negada |
| D0 (entrada D1) | 2 | FF1 entrada |
| D1 (entrada D2) | 12 | FF2 entrada |
| CLK | 3 e 11 | Clock (ligados em conjunto) |
| !CLR e !PR | 1,4,10,13 | → VCC (inactivos) |

> **Convenção importante:** o 74HC74 chama "FF1" e "FF2" aos seus dois flip-flops. No nosso esquema:
> - **FF1 do chip** armazena **Q0 da FSM** (o bit menos significativo)
> - **FF2 do chip** armazena **Q1 da FSM** (o bit mais significativo)

### 4.2 Ligações para D1 e D0

**D1 = Q0 → fio directo:**
```
Pin 5 (Q0) ──────────────────▶ Pin 12 (D2 do chip = D1 da FSM)
```

**D0 = !Q1 · !Q0 → porta AND (7408):**
```
Pin 8 (!Q1) ──┐
              ├──▶ 7408 Gate 1 ──▶ Pin 2 (D1 do chip = D0 da FSM)
Pin 6 (!Q0) ──┘
```

### 4.3 Saídas (LEDs)

As saídas Moore dependem só do estado:

| LED | Expressão | Ligação |
|-----|-----------|---------|
| LED_VERDE | !Q1 · !Q0 | = mesma saída de Gate 1 (ou separar) |
| LED_AMARELO | !Q1 · Q0 | 7408 Gate 2: Pin 8(!Q1) e Pin 5(Q0) |
| LED_VERMELHO | Q1 · !Q0 | 7408 Gate 3: Pin 9(Q1) e Pin 6(!Q0) |

> LED_VERDE = D0_next — podem partilhar a mesma porta AND, mas é boa prática usar gate separada para evitar carga eléctrica extra.

---

## 5. Porque é que não usámos o Método MUX?

### O que é o Método MUX?

O Método MUX é usado quando existe um sinal `AVANÇA` **separado do clock**:

```
D = AVANÇA' · Q_actual  +  AVANÇA · Q_next
```

- Quando `AVANÇA = 0` → D = Q_actual → estado mantém-se (self-loop)
- Quando `AVANÇA = 1` → D = Q_next → estado avança

O nome vem do facto de um MUX 2:1 implementar exactamente esta equação:
```
SEL = AVANÇA
Entrada A = Q_actual
Entrada B = Q_next
Saída = D
```

### Por que não foi necessário aqui?

No nosso circuito de Aula 10/11, **o Arduino faz as vezes do sinal AVANÇA**:

```
Arduino lê Q1 (pin3) e Q0 (pin2)
    ↓
Calcula o tempo do estado actual
    ↓
Espera esse tempo (delay)
    ↓
Gera um pulso LOW→HIGH→LOW no pin 13 (CLK)
```

O clock **só dispara quando é hora de avançar**. Não existe a situação "clock dispara mas estado não muda". Portanto:

| Situação | Método MUX | Nosso circuito |
|----------|:----------:|:--------------:|
| Estado deve manter-se | AVANÇA=0, CLK pode disparar | CLK **não dispara** |
| Estado deve avançar | AVANÇA=1, CLK dispara | CLK **dispara** |

Como o CLK só vem quando AVANÇA seria 1, as equações simplificam-se:

```
D = AVANÇA' · Q  +  AVANÇA · Q_next
  = 0 · Q  +  1 · Q_next       (porque AVANÇA é sempre 1 quando CLK chega)
  = Q_next
```

**Conclusão: D = Q_next directamente.** Sem MUX, sem sinal AVANÇA.

---

## 6. Como seria com o Método MUX?

Se em vez do Arduino usarmos um **temporizador em hardware** (ex.: 74HC4020 + 555), existe um sinal `AVANÇA` separado do clock. O clock corre sempre a frequência constante e `AVANÇA` decide se o estado muda.

### 6.1 Tabela de transições com AVANÇA

| Q1 | Q0 | AVANÇA | Q1_next | Q0_next |
|:--:|:--:|:------:|:-------:|:-------:|
| 0  | 0  | 0 | 0 | 0 | *(self-loop)*
| 0  | 0  | 1 | 0 | 1 |
| 0  | 1  | 0 | 0 | 1 | *(self-loop)*
| 0  | 1  | 1 | 1 | 0 |
| 1  | 0  | 0 | 1 | 0 | *(self-loop)*
| 1  | 0  | 1 | 0 | 0 |
| 1  | 1  | — | X | X |

### 6.2 Equações pelo Método MUX

**D1:**
```
D1 = AVANÇA' · Q1  +  AVANÇA · Q1_next

Q1_next por estado (quando AVANÇA=1):
  VERDE   (00) → Q1_next = 0
  AMARELO (01) → Q1_next = 1  ← é Q0 actual!
  VERMELHO(10) → Q1_next = 0

⟹ Q1_next = !Q1 · Q0  (ou Q0 com don't care em 11)

D1 = AVANÇA' · Q1  +  AVANÇA · Q0
```

**D0:**
```
D0 = AVANÇA' · Q0  +  AVANÇA · Q0_next

Q0_next por estado (quando AVANÇA=1):
  VERDE   (00) → Q0_next = 1  ← é !Q1 · !Q0
  AMARELO (01) → Q0_next = 0
  VERMELHO(10) → Q0_next = 0

⟹ Q0_next = !Q1 · !Q0

D0 = AVANÇA' · Q0  +  AVANÇA · !Q1 · !Q0
```

### 6.3 Implementação com 74HC157

```
Para D1:
  74HC157 MUX:  SEL = AVANÇA
                Entrada A = Q1  (estado actual — self-loop)
                Entrada B = Q0  (próximo Q1 — avançar)
                Saída → Pin 12 (D2 do chip)

Para D0:
  74HC157 MUX:  SEL = AVANÇA
                Entrada A = Q0              (self-loop)
                Entrada B = !Q1 · !Q0       (precisa de AND gate)
                Saída → Pin 2  (D1 do chip)
```

### 6.4 Onde vem AVANÇA?

Com 74HC4020 (contador) e 555 (clock):

```
555 (4 Hz) ──▶ 74HC4020 ──▶ Q3 (tap rápido, ≈1s) ──┐
                          └▶ Q4 (tap lento, ≈4s) ──┤
                                                     ▼
                                               74HC157 (SEL=Q0)
                                                     │
                                                  AVANÇA
                                               ┌────┴────┐
                                               ▼         ▼
                                         CLK (74HC74)  CLR (74HC4020)
```

O AVANÇA conectado ao CLR do 4020 cria um **auto-reset**: assim que o timer dispara, o contador recomeça para o próximo estado.

---

## 7. Comparação dos dois métodos

| | Sem MUX (Arduino) | Com MUX (hardware puro) |
|---|---|---|
| **Sinal AVANÇA** | Implícito no CLK | Explícito, separado do CLK |
| **CLK** | Arduino gera quando é hora | 555 corre sempre (frequência fixa) |
| **Self-loops** | Não existem no circuito | Tratados pelas equações D |
| **Equação D** | D = Q_next directamente | D = AVANÇA'·Q + AVANÇA·Q_next |
| **Chips usados** | 74HC74 + 7408 + Arduino | 74HC74 + 74HC157 + 74HC4020 + 555 + 7408 |
| **Vantagem** | Mais simples, timing flexível | 100% hardware, sem microcontrolador |
| **Limitação** | Precisa de Arduino (software) | Mais chips, timing fixo pelo 555 |

---

## 8. Erros frequentes a evitar

| Erro | Consequência | Solução |
|------|-------------|---------|
| Trocar D1↔D2 no 74HC74 | AMARELO fica bloqueado (state 01 nunca muda) | Pin12(D2)→Pin5(Q0); Gate1→Pin2(D1) |
| !CLR ou !PR sem ligar ao VCC | Chip fica em reset permanente | Ligar pinos 1,4,10,13 ao VCC (5V) |
| Esquecer resistências nos LEDs | LEDs queimam | 220Ω–470Ω em série com cada LED |
| GND não partilhado entre chips | Sinais flutuantes, comportamento errático | GND Arduino = GND 74HC74 = GND 7408 |
| D1 = Q0 interpretado como D_chip1 = Q0 | Confusão entre numeração do chip e da FSM | FF1 do chip = Q0 da FSM; FF2 do chip = Q1 |

---

*UC00652 · Escola Sicó · Aula 11 · 2025–2026*
