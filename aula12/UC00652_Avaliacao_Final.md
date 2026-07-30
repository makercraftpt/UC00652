# UC00652 — Avaliação Final
**Aula 12 · 27 Jul 2026 · Duração: 60 minutos**

---

**Nome:** _____________________________________________________ &nbsp;&nbsp; **Data:** __ / __ / 2026

**Consulta permitida:** formulário de referência (1 página fornecido pelo professor)
**Ferramenta:** LogiSim — entregar ficheiro `.circ` ou screenshot do circuito montado e a funcionar
**Proibido:** apontamentos, telemóvel, comunicação com colegas

---

## Enunciado — Controlador de Máquina de Lavar

Uma máquina de lavar doméstica simples tem **4 modos de operação** que se sucedem sempre na mesma ordem:

| Estado | Código | Descrição |
|--------|:------:|-----------|
| ESPERA | `00` | Máquina parada, aguarda o utilizador carregar START |
| LAVAR | `01` | Ciclo de lavagem com detergente |
| ENXAGUAR | `10` | Ciclo de enxaguamento |
| CENTRIFUGAR | `11` | Centrifugação para expulsar a água |

**Entradas:**

| Sinal | Descrição |
|-------|-----------|
| `START` | Botão de início — **só activo no estado ESPERA** |
| `TIMER` | Sinal do temporizador — indica que o tempo do ciclo actual terminou. **Activo nos estados LAVAR, ENXAGUAR e CENTRIFUGAR** |

**Regras de funcionamento:**
- Em **ESPERA**: quando o utilizador carrega `START`, a máquina avança para LAVAR
- Em **LAVAR**: quando o `TIMER` dispara, avança para ENXAGUAR
- Em **ENXAGUAR**: quando o `TIMER` dispara, avança para CENTRIFUGAR
- Em **CENTRIFUGAR**: quando o `TIMER` dispara, **volta** para ESPERA
- Se `AVANÇA = 0`, a máquina **mantém o estado actual** (self-loop)

**Saídas Moore** (dependem apenas do estado, não das entradas):

| Saída | Activa em |
|-------|-----------|
| `LED_PRONTO` | ESPERA |
| `MOTOR_LAVAR` | LAVAR e ENXAGUAR |
| `MOTOR_CENTRI` | CENTRIFUGAR |

---

## Exercício 1 — Diagrama de Estados `(20 pts)`

Desenha o diagrama de estados da FSM. Inclui:
- Os 4 estados com o respectivo código binário `(Q1 Q0)`
- As transições com a condição de `AVANÇA`
- As saídas Moore em cada estado

*(Usa o espaço abaixo — representa os estados como círculos e as transições como setas com etiqueta)*

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

---

## Exercício 2 — Tabela de Transições `(25 pts)`

Completa a tabela. As colunas `Q1` e `Q0` referem-se ao estado actual; `Q1_next` e `Q0_next` ao estado seguinte **quando `AVANÇA = 1`**.

| Estado actual | Q1 | Q0 | Condição de AVANÇA | Q1_next | Q0_next |
|:-------------:|:--:|:--:|:------------------:|:-------:|:-------:|
| ESPERA        | 0  | 0  |                    |         |         |
| LAVAR         | 0  | 1  |                    |         |         |
| ENXAGUAR      | 1  | 0  |                    |         |         |
| CENTRIFUGAR   | 1  | 1  |                    |         |         |

> **Nota:** quando `AVANÇA = 0`, o estado não muda: `Q1_next = Q1` e `Q0_next = Q0` (self-loop).

---

## Exercício 3 — Expressão de AVANÇA `(20 pts)`

Escreve a expressão booleana completa de `AVANÇA` em função de `Q1`, `Q0`, `START` e `TIMER`.

Preenche primeiro por estado:

| Estado | `AVANÇA` neste estado |
|--------|----------------------|
| ESPERA (00) | |
| LAVAR (01) | |
| ENXAGUAR (10) | |
| CENTRIFUGAR (11) | |

Expressão final de `AVANÇA` (combinação de todos os estados):

```
AVANÇA = 
```

&nbsp;

*(Dica: usa os valores de Q1 e Q0 de cada estado para identificar as linhas activas)*

---

## Exercício 4 — Equações D1 e D0 pelo Método MUX `(25 pts)`

O Método MUX dá:

```
D = AVANÇA' · Q_actual  +  AVANÇA · Q_next
```

**4.1** — Preenche a coluna `D1_next` e `D0_next` da tabela abaixo (valores de Q_next quando AVANÇA=1, da tabela do Ex. 2):

| Estado | Q1 | Q0 | D1_next (quando AVANÇA=1) | D0_next (quando AVANÇA=1) |
|:------:|:--:|:--:|--------------------------:|--------------------------:|
| ESPERA | 0 | 0 | | |
| LAVAR | 0 | 1 | | |
| ENXAGUAR | 1 | 0 | | |
| CENTRIFUGAR | 1 | 1 | | |

**4.2** — Identifica o padrão de `D1_next` (compara com XOR(Q1,Q0)):

```
D1_next  =  _______________
```

**4.3** — Identifica o padrão de `D0_next`:

```
D0_next  =  _______________
```

**4.4** — Escreve as equações finais de D1 e D0 aplicando o Método MUX:

```
D1  =  AVANÇA' · Q1  +  AVANÇA · _______________

D0  =  AVANÇA' · Q0  +  AVANÇA · _______________
```

---

## Exercício 5 — Saídas Moore `(10 pts)`

Escreve as expressões booleanas das três saídas em função de `Q1` e `Q0`:

| Saída | Activa em | Expressão booleana |
|-------|-----------|-------------------|
| `LED_PRONTO` | ESPERA (00) | |
| `MOTOR_LAVAR` | LAVAR (01) e ENXAGUAR (10) | |
| `MOTOR_CENTRI` | CENTRIFUGAR (11) | |

*(Dica: saída activa quando a combinação Q1,Q0 corresponde ao(s) estado(s) indicado(s))*

---

## ★ Desafio — LogiSim Completo `(Bonus)`

Se terminares antes do tempo, implementa o circuito completo em **LogiSim**:
- Flip-Flops D para armazenar Q1 e Q0
- Portas lógicas AND, OR, NOT para AVANÇA e as equações D
- MUX 2:1 (componente MUX do LogiSim) para D1 e D0 pelo Método MUX
- LEDs de saída para cada estado (MOTOR_LAVAR, MOTOR_CENTRI, LED_PRONTO)

Verifica que ao simular, os estados avançam correctamente com AVANÇA=1.

---

*Boa sorte! · UC00652 · CET Técnico/a Especialista em Automação, Robótica e Controlo Industrial · Escola Sicó · 2025–2026*
