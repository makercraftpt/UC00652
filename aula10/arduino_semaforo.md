# Arduino — Semáforo FSM (UC00652 Aula 10)

O Arduino substitui o NE555 + 74HC4020 + 74HC157.  
Lê o estado atual (Q1, Q0) dos FF-D, espera o tempo certo, e gera um pulso de clock.

## Ligações

| Pino Arduino | Liga a |
|---|---|
| 13 (CLK) | CLK do 74HC74 (pino 3 e/ou 11) |
| 2 (Q0 in) | Q0 do 74HC74 (pino 5) |
| 3 (Q1 in) | Q1 do 74HC74 (pino 9) |
| GND | GND comum do circuito |

## Sketch

```cpp
// UC00652 — Semáforo FSM
// Arduino como temporizador + gerador de clock
// Estados: VERDE(Q1=0,Q0=0) · AMARELO(Q1=0,Q0=1) · VERMELHO(Q1=1,Q0=0)

const int CLK = 13;  // saída → CLK do 74HC74
const int IN_Q0 = 2; // entrada ← Q0 do 74HC74
const int IN_Q1 = 3; // entrada ← Q1 do 74HC74

void setup() {
  pinMode(CLK, OUTPUT);
  pinMode(IN_Q0, INPUT);
  pinMode(IN_Q1, INPUT);
  digitalWrite(CLK, LOW);
}

void loop() {
  // 1. Lê o estado atual dos flip-flops
  int q1 = digitalRead(IN_Q1);
  int q0 = digitalRead(IN_Q0);

  // 2. Decide o tempo de espera conforme o estado
  int tempo = 3000;                        // default

  if (q1 == 0 && q0 == 0) tempo = 3000;   // VERDE    → 3 s
  if (q1 == 0 && q0 == 1) tempo = 800;    // AMARELO  → 0,8 s
  if (q1 == 1 && q0 == 0) tempo = 3000;   // VERMELHO → 3 s

  // 3. Aguarda o tempo do estado atual
  delay(tempo);

  // 4. Gera pulso de clock → FF-D avança para próximo estado
  digitalWrite(CLK, HIGH);
  delay(50);
  digitalWrite(CLK, LOW);
}
```

## O que acontece passo a passo

```
loop() começa
  │
  ├─ lê Q1, Q0  →  sabe em que estado está
  │
  ├─ delay(tempo)  →  espera o tempo certo para esse estado
  │
  └─ pulso CLK HIGH→LOW  →  74HC74 captura D1,D0 → novo estado
        │
        └─ D1 = Q0  (fio direto)
           D0 = !Q1 · !Q0  (porta AND com pinos Q̄)
```

## Tempos configuráveis

| Estado | Q1 | Q0 | `tempo` (ms) | Alterar para… |
|--------|----|----|--------------|--------------|
| VERDE  | 0  | 0  | 3000         | qualquer valor |
| AMARELO| 0  | 1  | 800          | qualquer valor |
| VERMELHO| 1 | 0  | 3000         | qualquer valor |

> **Nota:** o estado Q1=1, Q0=1 não existe no semáforo (estado inválido).  
> O circuito nunca chega lá se iniciado corretamente.
