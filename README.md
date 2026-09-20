# Projeto ULA - Sistemas Digitais

Repositório oficial do projeto da Unidade Lógica e Aritmética (ULA) desenvolvido para a disciplina de Sistemas Digitais. O projeto foi implementado inteiramente por meio de diagramas de blocos e portas lógicas básicas (como AND, OR, XOR, NOT), sem o uso de linguagens de descrição de hardware ou programação (VHDL/Verilog).

## Sobre o Projeto

O sistema é composto por uma ULA de 5 bits acoplada a um decodificador binário para acionamento de dois displays de sete segmentos, além de sinalizações dedicadas via LEDs:

### Entradas
- **A (5 bits):** Operando A (binário com sinal em complemento de dois, sendo 1 bit dedicado ao sinal).
- **B (5 bits):** Operando B (binário com sinal em complemento de dois).
- **S (3 bits):** Vetor seletor de operações.

### Saídas
- **F (5 bits):** Resultado numérico da operação.
- **Overflow (1 bit - LED):** Indica estouro aritmético.
- **Status (1 bit - LED):** Sinal lógico booleano de acordo com a função executada.
- **Sinal (1 bit - LED):** Indicador de sinal (aceso para resultados negativos e apagado para positivos).
- **Displays de 7 Segmentos:** Dois displays controlados por um decodificador dedicado para exibição do valor final.

## Tabela de Operações da ULA

As operações são selecionadas através do vetor **S (`S₂ S₁ S₀`)**:

| S₂ | S₁ | S₀ | Operação / Função |
| :---: | :---: | :---: | :--- |
| **0** | **0** | **0** | $F = A + B$ (Soma aritmética com complemento a 2) |
| **0** | **0** | **1** | $F = A - B$ (Subtração aritmética) |
| **0** | **1** | **0** | $F = \text{MIN}(A, B)$ (Menor entre A e B) |
| **0** | **1** | **1** | $F = 0$; $\text{Status} = (A \le B)$ |
| **1** | **0** | **0** | $F = 0$; $\text{Status} = 1$ se $A$ for par, e $0$ caso contrário |
| **1** | **0** | **1** | $F = \sqrt{A}$ (Raiz quadrada inteira, restrita para $A \ge 0$) |
| **1** | **1** | **0** | $F = \text{MAX}(0, A - B)$ |
| **1** | **1** | **1** | $F = (A \times 3) + 2$ |

## Estrutura e Organização dos Blocos
- **Ferramenta:** Intel Quartus Prime (utilizado para captura esquemática com arquivos `.bdf` e `.bsf`)
- **Módulos Principais:** 
  - Somador/Subtrator completo
  - Multiplexadores customizados (como `mux2to1`, `mux4to1`)
  - Blocos lógicos e aritméticos dedicados (como comparação de mínimo/máximo `min_a,b`)
  - Decodificador de 7 segmentos para a exibição das saídas.
- **Arquitetura:** Todos os blocos intermediários foram projetados de forma modular utilizando estritamente portas lógicas fundamentais.
