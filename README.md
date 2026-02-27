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
exten => s,1,NoOp(Iniciando Pesquisa de Satisfacao - Ivann Ribeiro)
exten => s,n,Answer()
exten => s,n(inicio),Read(NOTA,pesquisa-boas-vindas,1,,,10)

; Validação da nota (1 a 5)
exten => s,n,GotoIf($[${NOTA} >= 1 && ${NOTA} <= 5]?valido:invalido)

exten => s,n(invalido),Playback(pesquisa-opcao-invalida)
exten => s,n,Goto(inicio)

; Gravação no Banco de Dados (Campo Userfield do CDR)
exten => s,n(valido),Set(CDR(userfield)=NOTA_PESQUISA:${NOTA})
exten => s,n,Playback(pesquisa-agradecimento)
exten => s,n,Hangup()

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
