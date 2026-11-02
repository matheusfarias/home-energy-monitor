# Monitor de Energia Residencial

Protótipo que mede a corrente de um aparelho, calcula potência, energia e custo estimado, e mostra esses valores em tempo real. O projeto foi desenvolvido na disciplina **Microcontroladores e Aplicações** (julho de 2023), na placa didática **BIG8051**, que traz um **C8051F040** com conversor A/D e memória já disponíveis na placa.

**Autores:** Fernando Martins, Marcos Antonio Salami e Matheus Farias.

## Objetivo

Medir a corrente de dispositivos variados e, a partir dela, obter a potência, o gasto energético e o custo monetário estimado. Os valores medidos (custo estimado, kWh e demais grandezas) podem ser enviados para um computador ou um celular.

## Visão geral

O sensor de corrente fica no circuito do aparelho monitorado. A tensão proporcional à corrente entra no conversor A/D do microcontrolador, que converte a leitura em ampères e calcula as demais grandezas. O resultado aparece no LCD gráfico e fica gravado em memória não volátil. Pelo módulo serial, um computador (Tera Term) ou um celular (Bluetooth) envia comandos e recebe as leituras armazenadas.

```
Aparelho  →  ACS712  →  A/D (C8051F040)  →  cálculos
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           GLCD 128×64    EEPROM (I²C)    UART / RS-232
           (tempo real)   (histórico)     PC e celular
```

## Hardware

| Componente | Função no projeto |
| --- | --- |
| C8051F040 (placa BIG8051) | Processa as leituras, faz os cálculos e controla os periféricos |
| Sensor de corrente ACS712 (5 A) | Mede a corrente pelo efeito Hall. Faixa de −5 A a 5 A e sensibilidade de 185 mV/A |
| Conversor A/D | Aproximação sucessiva, 12 bits, erro de ±4 LSB, conversão em 16 clocks e referência interna de 2,43 V |
| LCD gráfico 128×64 (WDG0151-TMI-V) | Exibe corrente, potência, energia e custo estimado |
| EEPROM I²C de 128 bytes (24AA01 / 24LC01B) | Guarda o histórico das leituras, inclusive com o sistema desligado |
| UART / RS-232 | Liga o monitor ao computador e ao celular |

## Funcionamento

1. A interface serial recebe comandos pelo teclado do computador ou do celular.
2. O LCD gráfico mostra os valores lidos pelo sensor.
3. A saída de texto usa `printf_fast_f` tanto na UART quanto no GLCD. Uma flag escolhe qual implementação de `putchar` está ativa, para o caractere ir ao dispositivo certo.
4. As leituras são gravadas na EEPROM pelo protocolo I²C, inclusive valores em ponto flutuante. Quando a memória enche, as posições mais antigas são sobrescritas.
5. A tensão do sensor é convertida em corrente pelo A/D. Com a versão de 5 A, cada 185 mV na saída correspondem a 1 A.
6. Corrente, potência, energia e custo são mostrados no GLCD e gravados na EEPROM. Os comandos recebidos pela UART listam o que está armazenado.

## Cálculos

- **Potência:** \(W = V \times I\), com \(V\) igual à tensão nominal da rede, tratada como conhecida.
- **Energia:** média da potência ao longo do tempo, acumulada em um laço de repetição e expressa em kWh.
- **Custo:** energia calculada multiplicada por uma tarifa de referência por kWh.

## Testes

A calibração do sensor foi feita com uma fonte de corrente contínua de bancada. Um gerador de funções simulou o comportamento de aparelhos eletrônicos.

## Código

O firmware está em `src/main.c` e foi escrito para o C8051F040. O arquivo traz a fonte de caracteres e o acesso ao LCD gráfico 128×64, além de leitura e escrita de uma memória externa pela SPI, com um teste que percorre os endereços e confere os dados gravados. O programa depende de `config.c` e `def_pinos.h`, que acompanham o ambiente da placa BIG8051.

## Referências

1. Microchip. *1K I²C Serial EEPROM, 24AA01/24LC01B*. Julho de 2003.
2. Allegro. *Fully Integrated, Hall Effect-Based Linear Current Sensor ACS712*. 2007.
3. Winstar Display. *Graphical LCD Display, WDG0151-TMI-V*. Novembro de 2011.
4. NXP Semiconductors. *High-speed CAN transceiver, TJA1043*, Rev. 3. Abril de 2013.
