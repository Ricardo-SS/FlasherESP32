# Web Flasher ESP32/ESP8266

Interface web para gravação de firmware em ESP32/ESP8266 via WebSerial, usando pacote `.zip` com os binários.

## Status do Projeto

Este é um projeto em desenvolvimento.

- Novas melhorias e ajustes de compatibilidade podem ser adicionados.
- Comportamentos de interface, logs e validações de ZIP podem evoluir entre versões.

## Visão Geral

Este projeto permite:

- Conectar ao ESP via USB no navegador.
- Detectar chip e exibir informações técnicas no log.
- Ler firmware empacotado em `.zip`.
- Gravar firmware com opção de apagar flash antes.
- Acompanhar progresso da gravação.
- Usar terminal serial com histórico de comandos (`↑` / `↓`).

## Requisito Importante de Execução

Não abra o `flasher.html` com duplo clique (`file://...`).

Você precisa rodar a página em:

- `http://localhost` (recomendado para desenvolvimento local), ou
- uma página publicada em `https://...`.

### Por que isso é obrigatório?

- A API **WebSerial** exige contexto seguro (`localhost`/`https`).
- O app usa módulos JavaScript (`type="module"`), que dependem de carregamento via servidor.
- Em `file://`, o navegador bloqueia recursos importantes (imports, permissões e acesso serial).

## Como Rodar Localmente

No diretório do projeto, suba um servidor HTTP local.

### Opção 1: VS Code Live Server

1. Abra a pasta no VS Code.
2. Clique com o botão direito em `flasher.html`.
3. Selecione `Open with Live Server`.
4. Acesse a URL local gerada (ex.: `http://127.0.0.1:5500/flasher.html`).

### Opção 2: Python

```bash
python -m http.server 5500
```

Depois acesse:

`http://127.0.0.1:5500/flasher.html`

## Estrutura de Arquivos Relevantes

- `flasher.html`: interface principal.
- `jszip.min.js`: biblioteca local para leitura de ZIP.
- `libs/esptool-js/`: biblioteca local do esptool-js (modo offline/local).
- `esp32.zip`: exemplo de pacote de firmware (se aplicável ao seu fluxo).

## Dependências (Local e CDN)

O projeto está configurado para rodar com bibliotecas locais (offline):

- `esptool-js`: `./libs/esptool-js/bundle.js`
- `JSZip`: `./jszip.min.js`

No `flasher.html`, existem linhas de CDN comentadas para fallback rápido.  
Se quiser usar CDN, descomente e comente a linha local correspondente.

## Funcionalidades da Página

### 1. Conexão e detecção

- O botão `Conectar` abre o seletor de porta serial.
- Após conectar, o log mostra:
  - chip detectado,
  - VID/PID USB,
  - baud do flasher,
  - descrição/recursos,
  - cristal,
  - MAC,
  - tamanho de flash detectado.

### 2. Seleção de firmware ZIP

- Botão `Escolher ZIP` para selecionar o pacote.
- O conteúdo do ZIP é validado e exibido no log.

### 3. Gravação

- Toggle `Apagar antes` para executar erase completo antes da gravação.
- Botão `Gravar` inicia o processo.
- Barra de progresso atualizada durante escrita.

### 4. Abas de visualização

- `Log de gravação`: eventos técnicos do processo de conexão e flash.
- `Terminal serial`: monitor serial com envio de comandos.

### 5. Terminal serial

- Envio por `Enter` ou botão `Enviar`.
- Histórico de comandos com setas `↑` e `↓`.
- Botão `Limpar` para limpar a saída do terminal.

## Formato de ZIP Suportado

### ESP32

O parser aceita dois padrões:

1. Padrão legado (nomes fixos):
- `esp32/bootloader.bin`
- `esp32/partitions.bin`
- `esp32/firmware.bin`
- opcional: `esp32/boot_app0.bin`
- alternativa: `esp32/*.merged.bin`

2. Padrão Arduino IDE (nomes por sufixo, em qualquer pasta):
- `*.bootloader.bin`
- `*.partitions.bin`
- `*.bin` (firmware principal)
- opcional: `*.boot_app0.bin`
- alternativa prioritária: `*.merged.bin`

Quando existir `*.merged.bin`, ele tem prioridade.

### ESP8266

- Legado: `esp8266/firmware.bin`
- Flexível: qualquer `*.bin` compatível (sem sufixos de bootloader/partitions/merged)

### Regras de Segurança do Parser

- Se houver múltiplos candidatos para o mesmo papel (ex.: dois bootloaders), o processo é interrompido com erro de ambiguidade.
- Isso evita gravação de arquivo incorreto.

## Fluxo de Uso Recomendado

1. Feche Arduino IDE e outros apps que possam ocupar a porta serial.
2. Conecte o ESP via USB (cabo de dados).
3. Abra o `flasher.html` via `localhost`/`https`.
4. Clique em `Conectar` e selecione a porta.
5. Clique em `Escolher ZIP` e selecione o firmware.
6. (Opcional) marque `Apagar antes`.
7. Clique em `Gravar` e aguarde a conclusão.
8. Use a aba `Terminal serial` para validações pós-flash.

## Solução de Problemas

### A porta serial não aparece

- Troque o cabo USB (precisa ser cabo de dados).
- Troque a porta USB (evite hub/adaptador).
- Feche apps que usam serial (Arduino IDE, monitores seriais, etc.).
- Instale driver CH340/CP210x, se necessário.

### Erro ao abrir apenas o HTML no Explorer

- Esperado: `file://` não é modo suportado para este app.
- Rode via servidor local (`http://localhost`) ou publicação HTTPS.

### Falha de conexão/gravação

- Reduza baud se necessário.
- Teste com `Apagar antes`.
- Verifique se o ZIP está no formato esperado para o chip.
- Revise mensagens de ambiguidade no log caso existam arquivos duplicados.

## Segurança e Boas Práticas

- Grave apenas firmware confiável.
- Não desconecte USB durante erase/write.
- Mantenha energia estável durante a gravação.

