---
title: Conectar o DataScope com o Zapier
---

# Conectar o DataScope com o Zapier

> Prefere ler isso em espanhol ou inglês? [Versão em espanhol](connector_guide_es.html) | [English version](connector_guide_en.html).

Este guia explica como conectar o DataScope com o Zapier para automatizar sua
operação em campo: disparar Zaps quando um formulário é preenchido, quando um
documento é gerado, quando uma tarefa é atribuída ou quando um ticket é
registrado, e agir sobre o DataScope a partir dos outros aplicativos que você
já usa.

<aside class="notice">
Diferente do conector do Power Automate, aqui não há nada para importar. O app do DataScope está publicado no diretório de aplicativos do Zapier, então tudo o que você precisa para conectá-lo é sua API Key.
</aside>

## Antes de começar

Você precisa de:

- Uma conta do Zapier com permissão para criar Zaps no workspace em que você
  vai trabalhar.
- Sua API Key do DataScope. Você a obtém em <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>,
  na seção Integrações da sua conta, na aba API Key.
- Pelo menos um formulário publicado no DataScope. A maioria dos disparadores e
  das actions pede que você escolha um formulário, e a lista é preenchida com
  os formulários da sua conta.

## Passo 1: criar o Zap

1. Acesse <a href="https://zapier.com/apps/datascope-forms/integrations" target="_blank" rel="noopener noreferrer">o app do DataScope no Zapier</a> e faça login.
2. Selecione **Create Zap**, ou comece por um dos modelos listados nessa
   página. A seção Integrações do DataScope lista os mesmos modelos, agrupados
   por caso de uso.
3. Na etapa do disparador, busque **DataScope Forms** e selecione.
4. Escolha o evento ao qual você quer reagir, na lista em
   [O que você pode automatizar](#o-que-voce-pode-automatizar).

## Passo 2: criar a conexão

Na primeira vez que você usar o DataScope em um Zap, o Zapier vai pedir uma
conexão e mostrar um único campo, **Token Key**. Cole ali sua API Key do
DataScope.

Cole exatamente como o DataScope mostra, sem espaços no início ou no final. Se
a conexão for aceita mas o disparador depois não retornar nenhum dado, a API
Key é a primeira coisa a revisar.

A conexão pertence à sua conta do Zapier e você pode reutilizá-la em todos os
Zaps que montar. Cada pessoa que monta Zaps precisa da sua própria conexão,
com a API Key da sua conta.

## Passo 3: escolher o formulário

A maioria dos disparadores mostra uma lista **Form** com os formulários da sua
conta. Mesmo quando o campo é opcional, escolha um formulário.

<aside class="warning">
Deixar <b>Form</b> em branco cria uma assinatura que não fica associada a nenhum formulário, e ela nunca é disparada. Se você precisa da mesma automação em vários formulários, crie um Zap por formulário em vez de um Zap sem formulário selecionado.
</aside>

## Passo 4: testar e ativar o Zap

O botão **Test trigger** do Zapier busca os últimos eventos que correspondem ao
disparador, então ele precisa de pelo menos um para mostrar dados de exemplo.
Se sua conta ainda não tem nenhum, envie uma resposta de formulário, gere um
documento ou atribua uma tarefa, conforme o disparador escolhido, e teste
novamente.

Quando os dados de exemplo estiverem certos, adicione as etapas seguintes e
publique o Zap. A assinatura do lado do DataScope é criada quando você ativa o
Zap, não enquanto você o edita.

## O que você pode automatizar

O app do DataScope oferece os seguintes disparadores. Os nomes são os que você
vê na lista de disparadores do Zapier:

| Disparador | É acionado quando | Campo Form |
|---|---|---|
| Forms: New Form Entry | Uma resposta de formulário é enviada | Obrigatório |
| Forms: New PDF | Um PDF é gerado por backup por email ou por autonotificação | Opcional, escolha um |
| Forms: Status Changed | Uma resposta de formulário muda de status | Opcional, escolha um |
| Tasks: New Assigned Task | Uma tarefa é atribuída | Obrigatório |
| Signatures: New Completed Signature | Todos os signatários assinaram o documento | Opcional, escolha um |
| Signatures: New Rejected Signature | Um signatário rejeita a solicitação de assinatura | Opcional, escolha um |
| Signatures: Updated Signature | As assinaturas de um documento mudam e ainda há pendentes | Opcional, escolha um |
| Tickets: New Ticket (FKA Issue) | Um ticket é criado | Não se aplica, é no nível da conta |
| Tickets: Changed Status (FKA Issue) | Um ticket muda de status | Não se aplica, é no nível da conta |

Cada disparador entrega os dados do evento como campos individuais, prontos
para mapear nas etapas seguintes do Zap sem precisar processar o JSON
manualmente. No caso das respostas de formulário, cada pergunta chega como seu
próprio campo, com o nome, o tipo e o valor.

O app também oferece as seguintes actions, para que um Zap possa agir sobre o
DataScope e não apenas reagir aos seus eventos:

| Action | O que faz |
|---|---|
| Tasks: Assign Task V2 | Atribui uma tarefa de um formulário a um usuário, opcionalmente agendando-a e vinculando-a a um local ou a um lugar |
| Tasks: Assign Task V1 [Legacy] | A versão anterior da mesma action, mantida para que os Zaps existentes continuem funcionando |
| Forms: Send Data / New Answer [Beta] | Gera uma nova resposta de formulário e seu PDF a partir de um formulário existente usado como modelo |
| Change Form Status | Muda o status de uma resposta de formulário, identificada por nome e código do formulário |
| Modify Form Answer | Cria ou modifica uma resposta pontual dentro de um formulário já enviado |
| Tickets: Create Ticket (FKA Issue) | Cria um ticket, opcionalmente herdando os valores padrão de um Ticket Type |

### Assign Task: V1 ou V2

Use a **V2** para tudo que for novo. Ela faz tudo o que a V1 faz, e além disso:

- Aceita o sinal de mais no email do usuário, então
  `usuario+obra@exemplo.com` resolve para o usuário correto.
- Pode apontar para o novo módulo de **Lugares**, e não apenas para o módulo
  antigo de Locais.
- Pode encontrar um local ou um lugar existente pelo seu ID interno, que é o
  que você precisa quando vários lugares têm o mesmo nome.

A **V1** continua disponível para que os Zaps montados sobre ela sigam
rodando. Não há migração automática: se você quiser passar um Zap da V1 para a
V2, substitua a etapa da action e mapeie os campos novamente.

### Locais ou Lugares

**Tasks: Assign Task V2** e **Tickets: Create Ticket** têm um campo
**Location Type** com duas opções, e o comportamento muda de uma forma que
vale conhecer antes de montar o Zap:

| Location Type | Comportamento quando nada corresponde |
|---|---|
| Locations (old module, legacy) | O local é criado no DataScope a partir do nome e dos demais campos de local que você enviou |
| Places (new module) | Nada é criado. A tarefa não é atribuída e o Zap falha com `nestable location not found` |

Locations é a opção padrão, então os Zaps existentes continuam funcionando sem
mudanças. Se você escolher Places, crie o lugar no DataScope antes, ou pela
API de lugares, antes de o Zap rodar.

Na **Assign Task V2** o local é buscado nesta ordem: Location / Place ID,
depois Location Code, depois Location Name e depois Location Address. O
primeiro que corresponder é o escolhido.

Os demais campos de local, Location Phone, Location Email, Company Name,
Company Legal ID, Latitude e Longitude, se aplicam somente ao módulo antigo de
Locais, onde são salvos no local encontrado ou criado. Eles são ignorados com
Places, porque os lugares nunca são modificados por esta action.

### Send Data / New Answer está em Beta

**Forms: Send Data / New Answer** gera uma resposta de formulário e seu PDF a
partir de um formulário existente, o que a torna útil para trazer ao DataScope
dados capturados em outro lugar. Dois limites para levar em conta:

- **Ela não dispara o resto da cadeia de automação.** A sincronização com o
  Google Sheets, a assinatura automática e outros webhooks, incluindo os
  disparadores de Zaps, não rodam para uma resposta criada assim. Um Zap que
  reage a `Forms: New Form Entry` não vai vê-la.
- **Não todos os tipos de pergunta são totalmente suportados**, então alguns
  campos podem não ficar como você espera. Teste com o formulário que você vai
  usar de verdade.

## Considerações importantes

- **Um Zap ativo por formulário e por disparador.** Um disparador associado a
  um formulário admite uma única assinatura ativa por formulário. Ativar um
  segundo Zap com o mesmo disparador sobre o mesmo formulário é rejeitado com
  um erro que nomeia o tipo de disparador e pede que você remova o anterior.
  Disparadores diferentes sobre o mesmo formulário não são problema, e o mesmo
  disparador sobre formulários diferentes também não.
- **Se você precisa que várias coisas aconteçam com um mesmo evento, adicione
  etapas a um único Zap** em vez de criar um segundo Zap com o mesmo
  disparador e formulário.
- **Os disparadores de tickets operam no nível da conta.** `Tickets: New
  Ticket` e `Tickets: Changed Status` cobrem todos os tickets da conta e
  admitem um Zap ativo por conta cada um. Eles não têm campo Form.
- **Os campos disponíveis variam conforme sua configuração.** Alguns dados,
  como os de identidade dos signatários ou os campos estendidos de tarefas,
  aparecem somente se essas funcionalidades estiverem habilitadas na sua
  conta.
- **Os dados de exemplo de um disparador não são a lista completa de campos.**
  O Zapier monta a lista a partir dos eventos recentes, então uma pergunta que
  ninguém respondeu ainda, ou um campo opcional que chegou vazio, pode não
  aparecer até que apareça. Envie uma resposta representativa antes de mapear
  as etapas seguintes.
- **O disparador `Forms: New Form Entry` é visível do lado do DataScope.** Ele
  se registra na lista de Webhooks de
  <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>.
  Os demais disparadores não aparecem nessa lista.

## Se algo não funcionar

- **A conexão falha ao ser criada:** verifique se a API Key está completa e sem
  espaços no início ou no final.
- **Ao ativar o Zap ele falha dizendo que o hook já existe:** outro Zap da sua
  conta, talvez montado por outra pessoa, já usa esse disparador sobre esse
  formulário. Desative o Zap mais antigo e depois ative este. Se o erro
  persistir, escreva para o suporte e nós limpamos a assinatura que ficou.
- **O Test trigger não retorna dados:** o disparador lê eventos recentes, então
  uma conta que ainda não tem nenhum não tem nada para mostrar. Gere um e teste
  novamente.
- **O Zap nunca é disparado:** confirme que há um formulário selecionado no
  disparador, que o Zap está ativo, e revise o histórico do Zap no Zapier. Um
  evento que chegou enquanto o Zap estava desativado não é reenviado.
- **Um Zap que você desativou continua recebendo dados:** se o disparador for
  `Forms: New Form Entry`, abra a lista de Webhooks em
  <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>
  e exclua a entrada ali.
- **Os campos esperados não aparecem:** alguns campos dependem da configuração
  da sua conta. Escreva para o suporte e nós verificamos com você.

Para qualquer dúvida, escreva para o suporte e nós te acompanhamos na
configuração.
