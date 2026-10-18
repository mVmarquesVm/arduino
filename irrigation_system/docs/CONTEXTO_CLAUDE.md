# Contexto para o Claude do VS Code: Projeto Sistema de Irrigação (FIAP)

> Leia este arquivo inteiro antes de começar. Ele resume o que já foi feito numa sessão anterior e o que falta.

## Quem eu sou e como quero aprender

- Sou o Matheus (Engenharia de Software, FIAP). Faço o trabalho em dupla com o Jean R. Matias.
- **Quero aprender C++ e entender o projeto, não só entregar.** Aprendo pelo "feeling" e com **analogias**. Explicações longas e maçantes não funcionam pra mim.
- Vá em etapas pequenas, explique o **porquê**, e me deixe mexer no código quando fizer sentido. O `docs/GUIA_DO_CODIGO.md` já tem o mapa e as analogias; use como base.

## O trabalho (resumo do PDF do Prof. Guilherme Salati)

Protótipo IoT com **Arduino** (sensores, atuador, interface) + **ESP32** (internet via **MQTT + FIWARE**), protocolo próprio Arduino↔ESP32, **dashboard**, repositório no template do professor (`Arduino/`, `ESP32/`, `Dashboard/`, `README.md`), entregas por **Pull Request**.

Requisitos de software: `millis()` (todo `delay` justificado), **watchdog**, validação de sensores como erro, detecção de perda de comunicação mostrada ao usuário, **debounce**, funções/structs/ponteiros.

**Checkpoint 3: 27/10/2026.** Apresentação de até 15 min (explicação, demo ao vivo ou em vídeo, dashboard) + PR final com README completo e código do dashboard. Os checkpoints 1 e 2 já foram entregues.

## O que já está pronto (branch `feature/checkpoint3`)

| Parte | Estado | Como foi verificado |
|---|---|---|
| `Arduino/` (`.ino` + `src/` modular) | pronto | Compilado pelo **arduino-cli** (pipeline oficial da IDE) para o UNO com `--warnings all`: 0 avisos do nosso código, 41% da flash, 22% da RAM |
| Lógica de irrigação + protocolo | pronto | 32 testes no PC (`ferramentas/testes_host.cpp`), com AddressSanitizer |
| `ESP32/` (simulador do professor + `esp32.cpp`) | pronto | Compilado e rodado no Linux com Mosquitto real + `arduino_fake.py` |
| Cenários de falha | ok | cabo cortado (os dois lados detectam), queda do broker com reconexão, comando com o Arduino offline, comandos seguidos, valor inválido, falha de DHT e de solo |
| `FIWARE/provisionar.py` | pronto | Testado contra um **FIWARE falso** que imita as APIs (não contra o Docker real) |
| `Dashboard/` (Python puro + HTML + Chart.js local) | pronto | Testado no navegador: clicar em "Regar" chegou ao Arduino e a confirmação voltou; claro, escuro e celular |
| `README.md` | pronto | Segue o formato exigido pelo PDF |

### Decisões tomadas

- **Sem display LCD** (decisão do Matheus). A interface é feita com 2 botões (REGAR, MODO) e 2 LEDs (verde e vermelho, com padrões de pisca). Há um gancho `displayAtualizar()` em `src/interface/interface.cpp` caso entre um display.
- O simulador de ESP32 abre a **USB do Arduino**, então o protocolo usa a `Serial` (D0/D1). Os logs do Arduino saem pela mesma serial com o prefixo `#`, e o ESP32 mostra esses logs no terminal.
- O protocolo é texto `$TIPO,campos*CS\n` com checksum XOR. Os arquivos `protocolo.h/.cpp` são **idênticos** em `Arduino/src/protocolo/` e `ESP32/src/protocolo/`. Se mudar um, copie para o outro.
- FIWARE: apikey `TEF`, device `irrig001`, entidade `urn:ngsi-ld:Irrigacao:001`, comandos `regar` e `definirModo`.
- Foram corrigidos 5 bugs no simulador do professor, todos marcados com `[ALTERADO]` e listados em `ESP32/README.md`. O principal: o `subscribe()` ignorava o tópico.

## ⚠️ Pontos de atenção

1. **Requisito 2 do PDF: "Possuir um display com interface gráfica para o usuário".** Sem display, isso pode custar nota. Vale **confirmar com o professor** se os LEDs bastam, ou colocar um display barato (OLED SSD1306 I2C ou LCD 16x2 I2C) no gancho de `interface.cpp`. O código já está preparado.
2. **Nunca foi testado:** o hardware real (DHT11, servo, sensor de solo, botões, LEDs), o **PlatformIO** (`Arduino/platformio.ini`), o **FIWARE real em Docker** e o fluxo do **WSL2 + usbipd** no Windows.
3. **Registro de comandos no Orion:** algumas versões do IoT Agent criam esse registro sozinhas. O `provisionar.py` só cria se não existir. Se um comando chegar duplicado ao ESP32, apague o registro extra.
4. O teste de "fio solto" do sensor de solo usa o pull-up interno. Funciona com sensores de saída analógica comum, mas **calibre `SOLO_DESCONECTADO` no hardware real** (ver `config.h`).

## Próximos passos sugeridos

1. Me ensinar a **ler o código** na ordem do `docs/GUIA_DO_CODIGO.md`, um módulo por vez.
2. Configurar o ambiente no meu PC: WSL2, `libpaho-mqtt*`, cmake, Docker, e IDE do Arduino ou PlatformIO.
3. **Rodar sem hardware:** FIWARE (Docker) → `provisionar.py` → `arduino_fake.py` → `ESP32Mock` → `dashboard.py`.
4. Montar o hardware, gravar o Arduino e trocar o fake pelo real.
5. Decidir a questão do display (ponto 1).
6. Fazer commit, push e abrir o **Pull Request** (me ensine o fluxo pelo VS Code).
7. Roteiro da apresentação de 15 min. Plano B: `dashboard.py --demo` + `arduino_fake.py`, ou um vídeo gravado antes.
