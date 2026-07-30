# Ligações TinkerCAD — Semáforo FSM (UC00652 Aula 10)

## Convenção de pinos do 74HC74

| Pino chip | Função chip | Função na FSM |
|:---------:|-------------|---------------|
| 2  | D1  | D0_próximo (entrada dados FF que guarda Q0) |
| 3  | CLK1 | Clock (Arduino pin 13) |
| 5  | Q1  | **Q0** (saída atual) |
| 6  | !Q1 | **!Q0** |
| 9  | Q2  | **Q1** (saída atual) |
| 8  | !Q2 | **!Q1** |
| 11 | CLK2 | Clock (Arduino pin 13) |
| 12 | D2  | D1_próximo (entrada dados FF que guarda Q1) |

---

## Passos de ligação — ordem de montagem

| Passo | O quê | De | Para | Porquê |
|:-----:|-------|-----|------|--------|
| 1 | Alimentação 74HC74 | VCC (5V) | Pin 14 | Obrigatório |
| 1 | Alimentação 74HC74 | GND | Pin 7 | Obrigatório |
| 2 | !CLR e !PR (desativar reset) | VCC | Pins 1, 4, 10, 13 | Ativos a LOW — sem VCC o chip fica em reset |
| 3 | Clock | Arduino pin 13 | Pin 3 (CLK1) e Pin 11 (CLK2) | Os dois FFs partilham o mesmo clock |
| **4** | **D2 — fio direto (próximo Q1)** | **Pin 5 (Q0 atual)** | **Pin 12 (D2)** | **Próximo Q1 = Q0 atual** |
| **5** | **D1 via AND gate (próximo Q0)** | **7408 Gate 1** | **Pin 2 (D1)** | **D0_próximo = !Q1·!Q0** |
| 6 | LED VERDE | 7408 Gate 1 (mesmo fio que D1) | 220Ω → LED verde → GND | Acende quando Q1=0, Q0=0 |
| 7 | LED AMARELO | 7408 Gate 2 | 220Ω → LED amarelo → GND | Acende quando Q1=0, Q0=1 |
| 8 | LED VERMELHO | 7408 Gate 3 | 220Ω → LED vermelho → GND | Acende quando Q1=1, Q0=0 |
| 9 | Arduino lê Q0 | Pin 5 (Q0) | Arduino pin 2 | Leitura de estado |
| 9 | Arduino lê Q1 | Pin 9 (Q1) | Arduino pin 3 | Leitura de estado |
| 10 | Alimentação 7408 | VCC | 7408 pin 14 | Obrigatório |
| 10 | Alimentação 7408 | GND | 7408 pin 7 | Obrigatório |

---

## Portas AND do 7408

| Gate | Entrada A | Entrada B | Saída | Função |
|:----:|-----------|-----------|-------|--------|
| 1 | Pin 6 (!Q0) | Pin 8 (!Q1) | → Pin 2 (D1) + LED VERDE | !Q1·!Q0 = próximo Q0 |
| 2 | Pin 8 (!Q1) | Pin 5 (Q0)  | → LED AMARELO | !Q1·Q0 → acende em AMARELO |
| 3 | Pin 9 (Q1)  | Pin 6 (!Q0) | → LED VERMELHO | Q1·!Q0 → acende em VERMELHO |

---

## Verificação das transições (com ligação correta)

| Estado atual | Q1 | Q0 | D2 = Q0 | D1 = !Q1·!Q0 | Próximo Q1 | Próximo Q0 | Próximo estado |
|:------------:|:--:|:--:|:-------:|:------------:|:----------:|:----------:|:--------------:|
| VERDE        | 0  | 0  | 0       | 1            | 0          | 1          | AMARELO ✓      |
| AMARELO      | 0  | 1  | 1       | 0            | 1          | 0          | VERMELHO ✓     |
| VERMELHO     | 1  | 0  | 0       | 0            | 0          | 0          | VERDE ✓        |

> **Atenção (erro anterior):** com D1→Pin5 e Gate1→Pin12 trocados, o estado AMARELO ficava preso — D1=Q0=1 não mudava nunca.
