# Projeto ULA - Sistemas Digitais

Repositório oficial do projeto da **Unidade Lógica e Aritmética (ULA)** de 5 bits desenvolvido para a disciplina de Sistemas Digitais.

O projeto foi implementado **inteiramente por captura esquemática** (diagramas de blocos e portas lógicas básicas como AND, OR, XOR e NOT), **sem o uso de linguagens de descrição de hardware ou de programação** (VHDL/Verilog) nos circuitos.

## Sumário

1. [Visão geral](#visão-geral)
2. [Entradas e saídas](#entradas-e-saídas)
3. [Tabela de operações](#tabela-de-operações)
4. [Sinalização por LEDs](#sinalização-por-leds)
5. [Arquitetura](#arquitetura)
6. [Descrição dos blocos](#descrição-dos-blocos)
7. [Estado do desenvolvimento e testes](#estado-do-desenvolvimento-e-testes)
8. [Como compilar e simular](#como-compilar-e-simular)
9. [Convenções e dicas do projeto](#convenções-e-dicas-do-projeto)
10. [Equipe](#equipe)

## Visão geral

O sistema é composto por uma ULA de 5 bits acoplada a um decodificador binário para acionamento de **dois displays de sete segmentos**, além de sinalizações dedicadas por **LEDs** (overflow, status e sinal).

- **Ferramenta de projeto:** Intel Quartus Prime Lite 21.1 (captura esquemática com arquivos `.bdf` e símbolos `.bsf`).
- **Simulação:** Questa Intel FPGA Starter Edition 2021.2 (simulação funcional em nível de portas, a partir do Quartus).
- **Representação numérica:** números inteiros de 5 bits em **complemento de dois**, no intervalo de **−16 a +15**.

## Entradas e saídas

### Entradas

| Sinal | Largura | Descrição |
|---|---|---|
| `A[4..0]` | 5 bits | Operando A (complemento de dois, 1 bit de sinal) |
| `B[4..0]` | 5 bits | Operando B (complemento de dois, 1 bit de sinal) |
| `S[2..0]` | 3 bits | Vetor seletor da operação |

### Saídas

| Sinal | Largura | Descrição |
|---|---|---|
| `F[4..0]` | 5 bits | Resultado numérico da operação |
| `Overflow` | 1 bit (LED) | Indica estouro aritmético |
| `Status` | 1 bit (LED) | Sinal booleano, conforme a função executada |
| `Neg` ("Sinal") | 1 bit (LED) | Aceso para resultado negativo, apagado para positivo |
| Displays de 7 segmentos | 2 displays | Valor final de `F`, acionado por um decodificador dedicado |

## Tabela de operações

As operações são selecionadas pelo vetor `S` (S2 S1 S0):

| S2 | S1 | S0 | Operação |
|:-:|:-:|:-:|---|
| 0 | 0 | 0 | `F = A + B` (soma em complemento de dois) |
| 0 | 0 | 1 | `F = A − B` (subtração em complemento de dois) |
| 0 | 1 | 0 | `F = MIN(A, B)` (menor entre A e B) |
| 0 | 1 | 1 | `F = 0`; `Status = (A ≤ B)` |
| 1 | 0 | 0 | `F = 0`; `Status = 1` se A for par, `0` caso contrário |
| 1 | 0 | 1 | `F = ⌊√A⌋` (raiz quadrada inteira, definida apenas para A ≥ 0) |
| 1 | 1 | 0 | `F = MAX(0, A − B)` |
| 1 | 1 | 1 | `F = (A × 3) + 2` |

## Sinalização por LEDs

| LED | Quando acende |
|---|---|
| **Overflow** | Quando o resultado **não cabe em 5 bits** (fora de −16..15), nas operações que podem estourar: `A + B`, `A − B`, `MAX(0, A − B)` (somente quando `A − B > 15`) e `(A × 3) + 2` (somente quando A está fora do intervalo −6..4). |
| **Status** | `S = 011`: acende quando `A ≤ B`. `S = 100`: acende quando A é par. Nas demais operações fica apagado. |
| **Sinal (Neg)** | Quando o resultado **verdadeiro** é negativo. Na soma e na subtração usa o bit de sinal corrigido por overflow; no `MIN` usa o sinal do menor valor; em `(A × 3) + 2` usa o sinal de A; nas demais operações fica apagado. |

## Arquitetura

A ULA calcula **todas as operações em paralelo**, cada uma em seu bloco, e **multiplexadores** escolhem qual resultado chega à saída, de acordo com `S`. Para **cada saída** (cada bit de `F` e cada LED) a seleção é feita por três multiplexadores:

```
            S1 S0                       S2
  grupo S2=0 ──► mux4t01 ─┐
  (000,001,010,011)       ├──► mux2to1 ──► saída
  grupo S2=1 ──► mux4t01 ─┘
  (100,101,110,111)
```

| Entrada do mux | `mux4t01` (S2 = 0) | `mux4t01` (S2 = 1) |
|---|---|---|
| D0 | A + B | 0 (S = 100) |
| D1 | A − B | √A |
| D2 | MIN(A, B) | MAX(0, A − B) |
| D3 | 0 (S = 011) | (A × 3) + 2 |

No `mux2to1`, `S2 = 1` escolhe o grupo de cima (S2 = 1) e `S2 = 0` escolhe o grupo de baixo. As saídas `Overflow`, `Status` e `Neg` usam a mesma estrutura de seleção, cada uma com os sinais correspondentes de cada operação. As ligações entre blocos são feitas por **nomes de fios** (fios com o mesmo nome se conectam), o que evita fios longos no esquemático.

## Descrição dos blocos

Todos os blocos são arquivos `.bdf` na raiz do projeto, construídos apenas com portas lógicas fundamentais e com os blocos listados abaixo.

| Arquivo | Função |
|---|---|
| `ula.bdf` | Esquemático principal: instancia os blocos de operação e os multiplexadores de seleção. |
| `full_adder.bdf` | Somador completo de 1 bit (XOR, AND e OR). |
| `somador_completo.bdf` | Somador de 5 bits com 5 `full_adder` em cascata (carry em cadeia). Entradas `A`, `B`, `Cin`; saídas `Sum` e `Cout`. |
| `subtrator_completo.bdf` | Subtrator `A − B = A + (NOT B) + 1`, usando o `somador_completo`. Gera `S`, o sinal bruto `Neg` e o `Overflow` (`(A4 XOR B4) AND (A4 XOR S4)`). |
| `min_ab.bdf` | `MIN(A, B)`: usa o subtrator e escolhe `A` ou `B`, bit a bit, por cinco `mux2to1`, com seletor `Neg XOR Overflow` (sinal verdadeiro de `A − B`). |
| `max_zero_ab.bdf` | `MAX(0, A − B)`: zera o resultado do subtrator quando o sinal verdadeiro é negativo. |
| `le_status.bdf` | `A ≤ B`: `Status = (S == 0) OR (Neg XOR Overflow)`, a partir da saída do subtrator. |
| `status_par.bdf` | Paridade de A: `Status = NOT A[0]`; `F` fixo em 0. |
| `raiz_a.bdf` | Raiz quadrada inteira de A (A ≥ 0), por lógica combinacional: `F[1] = A3 + A2` e `F[0] = A3'·A2'·(A1 + A0) + A3·(A2 + A1 + A0)`. `F[4..2]` são 0. |
| `ax3_2.bdf` | `(A × 3) + 2` com somadores completos. O overflow acende quando A está fora de −6..4. |
| `mux2to1.bdf` | Multiplexador de 2 entradas de 1 bit. |
| `mux4t01.bdf` | Multiplexador de 4 entradas de 1 bit (`Y = D[S1 S0]`). |
| Decodificador de 7 segmentos | Converte `F` para os dois displays (ver [estado do desenvolvimento](#estado-do-desenvolvimento-e-testes)). |

## Estado do desenvolvimento e testes

Cada bloco foi testado isoladamente na simulação funcional (Questa), aplicando entradas com `force` e comparando o resultado com o valor esperado.

| Bloco | Situação |
|---|---|
| `subtrator_completo` | Testado (casos normais e de overflow). |
| `max_zero_ab` | Testado (inclui overflow positivo). |
| `min_ab` | Testado (inclui casos de overflow na subtração). |
| `le_status` | Testado (inclui casos de overflow). |
| `status_par` | Testado nos 32 valores de A. |
| `raiz_a` | Testado para A de 0 a 15. |
| `ax3_2` | Testado nos 32 valores de A (valor de F e overflow). |
| `full_adder`, `somador_completo` | Em uso pelos blocos acima; teste isolado do somador a ser repetido após a última correção. |
| `mux2to1`, `mux4t01` | Em uso nos blocos e na ULA. |
| `ula` | Montada; compilação e teste das 8 operações em andamento. |
| Decodificador de 7 segmentos | Em desenvolvimento. |

## Como compilar e simular

### Pré-requisitos

- Intel Quartus Prime Lite 21.1 com a **Questa Intel FPGA Starter Edition** instalada.
- Licença gratuita da Questa (tipo **Fixed**), apontada pelas variáveis de ambiente `MGLS_LICENSE_FILE` (e, por garantia, `LM_LICENSE_FILE` e `SALT_LICENSE_FILE`) com o caminho completo do arquivo `.dat`. Feche e reabra o Quartus depois de criar as variáveis.
- Em *Assignments > Settings > EDA Tool Settings > Simulation*: *Tool name* = `Questa Intel FPGA`, *Format* = `Verilog HDL`. Em *Tools > Options > EDA Tool Options*, aponte a pasta `questa_fse\win64`.

### Passo a passo

1. Abra o projeto (`PROJETO_ULA.qpf`).
2. Defina o bloco a testar como **topo**: botão direito no arquivo `.bdf` > *Set as Top-Level Entity*.
3. Compile com **Ctrl+L** (compilação completa; é ela que gera o netlist de simulação).
4. Abra **Tools > Run Simulation Tool > Gate Level Simulation**.
5. No Transcript do Questa, digite `vdir` para ver o **nome do módulo** (o Quartus costuma gravá-lo com a primeira letra maiúscula, por exemplo `Ula`).
6. Carregue o circuito:

```tcl
vsim -voptargs=+acc -L cyclonev_ver -L altera_ver -L altera_lnsim_ver -L lpm_ver -L sgate_ver -L altera_mf_ver work.Ula
```

7. Aplique entradas e confira as saídas. Exemplo (`S = 001`, `A = 15`, `B = −1`; esperado `F = 10000` com overflow):

```tcl
force -freeze S 2#001 0; force -freeze A 2#01111 0; force -freeze B 2#11111 0; run 10 ns; echo "F=[examine -radix binary F] Ovf=[examine -radix unsigned Overflow] St=[examine -radix unsigned Status] Neg=[examine -radix unsigned Neg]"
```

### Observações

- **Feche o Questa** antes de rodar outra simulação (senão o Quartus dá erro de permissão em `msim_transcript`).
- Se o Questa disser `Failed to find design unit`, o netlist é de outro bloco: confira o topo do projeto, apague `simulation/modelsim/PROJETO_ULA.vo` e compile de novo.
- Ao trocar de bloco, repita *Set as Top-Level* + **Ctrl+L** + *Gate Level Simulation*.


## Convenções e dicas do projeto

- **Nomes de arquivos e blocos:** use apenas letras, números e `_`. Evite vírgula, hífen e `+` (causam erros na compilação e na simulação).
- **Barramentos** (`A[4..0]`): linha **grossa** (bus), com o nome do barramento. **Fios de 1 bit** (`A[0]`, `g1`): linha **fina**, com nome.
- **Fios com o mesmo nome se conectam**, mesmo sem fio desenhado entre eles.
- Um fio só liga uma porta se a **ponta encostar exatamente** na porta. Entradas soltas são assumidas como `1`, o que gera resultados errados sem aviso óbvio.
- Depois de mudar os pinos de um bloco, atualize o símbolo (*Create Symbol Files for Current File*) e, nos esquemáticos que o usam, *Update Symbol or Block*.
- Mantenha **um único arquivo** por multiplexador (evite `mux4t01` e `mux4to1` duplicados).
- Os números com sinal são de 5 bits: a faixa válida é **−16 a +15**.

## Equipe

- Ana Carolina Cavalcante de Jesus
- Maria Luísa Silva Nunes de Souza
