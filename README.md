# 📊 Pesquisa de Satisfação Pós-Atendimento (Asterisk/FreePBX)

Este guia permite implementar uma pesquisa de satisfação (notas de 1 a 5) onde o agente transfere o cliente manualmente para o sistema de avaliação.

## 🛠️ Requisitos

* Audios no formato `.wav` ou `.sln` (8khz, Mono):
* `pesquisa-boas-vindas`
* `pesquisa-agradecimento`
* `pesquisa-opcao-invalida`


* Acesso administrativo à interface web do FreePBX.

---

## 1. Upload dos Áudios

1. Acesse **Admin > Sound Languages**.
2. Clique em **Custom Languages** > **Add Custom Language** (opcional, para organizar).
3. Vá em **Settings > UCP Addons > Recordings** ou utilize **Admin > System Recordings**.
4. Faça o upload dos três arquivos com os nomes exatos: `pesquisa-boas-vindas`, `pesquisa-agradecimento` e `pesquisa-opcao-invalida`.

---

## 2. Criando o Destino da Pesquisa (Custom Context)

Como os parceiros não têm acesso ao SSH, utilizaremos o módulo **Config Edit** (ou similar) disponível na interface do IncrediblePBX para editar o plano de discagem.

1. Vá em **Admin > Config Edit**.
2. Selecione o arquivo `extensions_custom.conf`.
3. Role até o final e cole o seguinte código:

```asterisk
[pesquisa-satisfacao]
exten => s,1,NoOp(Iniciando Pesquisa de Satisfacao - Empresa XYZ)
same => n,Answer()
same => n(inicio),Read(NOTA,pesquisa-boas-vindas,1,,,10)

; Validação da nota (1 a 5)
; Se NOTA for vazia, define como 0 para nao quebrar o GotoIf
same => n,GotoIf($["${NOTA}" = ""]?invalido)
same => n,GotoIf($[${NOTA} >= 1 && ${NOTA} <= 5]?valido:invalido)

same => n(invalido),Playback(pesquisa-opcao-invalida)
same => n,Goto(inicio)

; Gravação no Banco de Dados (Campo Userfield do CDR)
same => n(valido),Set(CDR(userfield)=NOTA_PESQUISA:${NOTA})
same => n,Playback(pesquisa-agradecimento)
same => n,Hangup()

```

4. Clique em **Save** e depois em **Apply Config**.

---

## 3. Criando o Destino Customizado (Custom Destination)

Para que o FreePBX "enxergue" o código que escrevemos:

1. Vá em **Admin > Custom Destinations**.
2. Clique em **Add Destination**.
3. **Target:** `pesquisa-satisfacao,s,1`
4. **Description:** `Pesquisa de Satisfação`
5. Clique em **Submit**.

---

## 4. Criando o Código de Acesso (Misc Application)

Isso criará um ramal virtual (ex: 500) que o agente usará para transferir o cliente.

1. Vá em **Applications > Misc Applications**.
2. Clique em **Add Misc Application**.
3. **Description:** `Enviar para Pesquisa`
4. **Feature Code:** `500` (Este é o número que o agente vai discar).
5. **Destination:** Selecione **Custom Destinations > Pesquisa de Satisfação**.
6. Clique em **Submit** e **Apply Config**.

---

## 📖 Como utilizar (Manual do Agente)

Quando o atendimento terminar, o agente deve seguir este procedimento:

1. Informe ao cliente: *"Por favor, não desligue para avaliar meu atendimento"*.
2. Pressione a tecla de transferência (geralmente `##` ou o botão **Transfer** do telefone IP).
3. Disque **500**.
4. Desligue o telefone. O cliente ouvirá a pesquisa automaticamente.

---

## 📈 Como extrair os resultados

As notas ficarão salvas no campo `Userfield` do relatório de chamadas (CDR).

* Acesse **Reports > CDR Reports**.
* A nota aparecerá como `NOTA_PESQUISA:X`.

---

## 🔍 Como Validar a Gravação das Notas (Sem SSH)

Utilizando as ferramentas nativas da interface do **IncrediblePBX**.

### Opção 1: Relatório de Chamadas (CDR Reports)

Maneira mais simples de verificar se a nota foi salva com sucesso.

1. No menu superior, acesse **Reports > CDR Reports**.
2. Clique no botão **Search** para carregar as chamadas recentes.
3. Localize a coluna **Userfield**.
* Ela conterá o registro formatado como: `NOTA_PESQUISA:5` (ou o valor digitado).


4. **Dica:** Caso a coluna não esteja visível, clique no ícone de engrenagem ou opções da tabela e certifique-se de que o campo **Userfield** está marcado para exibição.

### Opção 2: Monitoramento em Tempo Real (Asterisk Log Files)

Se você quiser ver o "passo a passo" do cliente digitando a nota enquanto a chamada acontece:

1. Acesse **Admin > Asterisk Log Files**.
2. Selecione o arquivo de log chamado `full`.
3. Ative a opção **Auto-Scroll** e defina o intervalo de atualização para **3 Seconds**.
4. Procure por linhas contendo `Iniciando Pesquisa de Satisfacao`.
5. Você verá o Asterisk processando a nota:
* Procure pela linha: `Executing [... Set("CDR(userfield)=NOTA_PESQUISA:X")]`. Isso confirma que o sistema escreveu o dado com sucesso.

---

## 🛠️ Solução de Problemas (Troubleshooting)

Se as notas não estiverem aparecendo no Userfield:

* **Configuração do CDR:** No menu **Settings > Advanced Settings**, verifique se a opção "Log CDR Userfield" está marcada como `Yes`.
* **Fluxo da Chamada:** Certifique-se de que o agente realizou a transferência (Blind Transfer) para o ramal **500**. Se o cliente desligar antes de digitar a nota, nada será gravado.
* **Nomes dos Áudios:** Verifique em **Admin > System Recordings** se os arquivos foram carregados com os nomes exatos utilizados no script: `pesquisa-boas-vindas`, `pesquisa-agradecimento` e `pesquisa-opcao-invalida`.

---
