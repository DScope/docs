# Conectar o DataScope com o Power Automate

> Prefere ler isso em espanhol ou inglês? [Versão em espanhol](https://github.com/DScope/docs/blob/main/source/power_automate/connector_guide_es.md) | [English version](https://github.com/DScope/docs/blob/main/source/power_automate/connector_guide_en.md).

Este guia explica como habilitar a integração do DataScope no Power Automate
para automatizar sua operação em campo: disparar fluxos quando um formulário
é preenchido, quando um documento é gerado, quando uma tarefa é atribuída ou
quando uma constatação é registrada.

## Antes de começar

Você precisa de:

- Uma licença do Power Automate que habilite conectores personalizados. Se não
  tiver certeza, consulte o administrador do seu tenant da Microsoft.
- Permissão para criar conectores no ambiente do Power Automate em que você
  vai trabalhar.
- Sua API Key do DataScope. Você a obtém em **app.mydatascope.com/integrations**,
  na seção Integrações da sua conta.
- O arquivo `apiDefinition.swagger.json` fornecido junto com este guia.

## Passo 1: criar o conector

1. Acesse [make.powerautomate.com](https://make.powerautomate.com) e faça
   login.
2. Selecione o ambiente onde deseja criar o conector, no canto superior
   direito.
3. No menu lateral, abra **Mais** e depois **Descobrir tudo**.
4. Selecione **Conectores personalizados**.
5. Clique em **Novo conector** e escolha **Importar um arquivo OpenAPI**.
6. Digite um nome para o conector, por exemplo DataScope, selecione o
   arquivo `apiDefinition.swagger.json` e continue.

## Passo 2: revisar a configuração geral

O arquivo já traz configurados o host, o caminho base e a descrição, então
não é necessário alterar nada nesta tela.

Se quiser que o conector exiba o logo do DataScope, faça o upload no campo de
ícone e defina a cor de fundo. Isso é opcional e afeta apenas a aparência.

## Passo 3: configurar a autenticação

O arquivo já declara o tipo de autenticação. Verifique se a tela de segurança
mostra:

- Tipo de autenticação: **Chave de API**
- Nome do parâmetro: **Authorization**
- Localização: **Cabeçalho**

Continue até o final do assistente e selecione **Criar conector**.

## Passo 4: criar a conexão

Ao usar o conector pela primeira vez em um fluxo, o Power Automate solicitará
uma conexão. Cole ali sua API Key do DataScope.

Cada usuário que criar fluxos precisa criar sua própria conexão com a API Key
da sua conta.

## O que você pode automatizar

O conector oferece os seguintes disparadores:

| Disparador | É acionado quando |
|---|---|
| Nova resposta | Uma resposta de formulário é enviada |
| Novo PDF | Um documento PDF é gerado |
| Mudança de status | Uma resposta de formulário muda de status |
| Nova tarefa atribuída | Uma tarefa é atribuída |
| Nova constatação | Uma constatação é registrada |
| Mudança de status de constatação | Uma constatação muda de status |
| Assinatura concluída | A assinatura de um documento é concluída |
| Assinatura rejeitada | Uma solicitação de assinatura é rejeitada |
| Assinatura atualizada | As assinaturas de um documento são atualizadas |

Cada disparador entrega os dados do evento como conteúdo dinâmico, prontos
para usar nas próximas etapas do fluxo sem precisar processar o JSON
manualmente.

O conector também oferece as seguintes ações, para que um fluxo possa enviar
dados de volta ao DataScope:

| Ação | O que faz |
|---|---|
| Assign Task | Atribui uma tarefa de um formulário a um usuário |
| Change Form Status | Altera o status de uma resposta de formulário |
| Modify Form Answer | Cria ou atualiza a resposta de uma pergunta em uma resposta de formulário |
| Send Data / New Answer [Beta] | Gera uma nova resposta de formulário e seu PDF a partir de um modelo existente |
| Create Ticket | Cria um novo ticket |

## Considerações importantes

- **Uma conexão ativa por formulário.** Os disparadores associados a um
  formulário admitem apenas uma conexão ativa por vez. Se precisar de vários
  fluxos sobre o mesmo formulário, encadeie-os a partir de um único
  disparador.
- **Os disparadores de constatações operam no nível da conta.** Admitem
  apenas uma conexão ativa por conta.
- **Os campos disponíveis variam conforme sua configuração.** Alguns dados,
  como os de planejamento ou os campos estendidos de tarefas, aparecem
  somente se essas funcionalidades estiverem habilitadas na sua conta.
- **O conector funciona no ambiente em que foi criado.** Se você trabalha com
  vários ambientes, repita a importação em cada um.
- **O disparador "Nova resposta" não se limpa sozinho.** Se você desativar ou
  excluir um flow que usa o disparador "Nova resposta" (`hooks_flow`), a
  assinatura não é removida automaticamente do lado do DataScope. Para
  interrompê-lo por completo, também é necessário acessar
  [app.mydatascope.com/integrations](https://app.mydatascope.com/integrations)
  e excluir a conexão correspondente ali. Os demais disparadores
  (formulários, PDFs, tarefas, constatações, assinaturas) se limpam
  automaticamente ao desativar o flow no Power Automate.

## Se algo não funcionar

- **A conexão falha ao ser criada:** verifique se a API Key está completa e
  sem espaços no início ou no final.
- **O fluxo não é disparado:** confirme que não existe outra conexão ativa
  sobre o mesmo formulário e revise o histórico de execuções do fluxo no
  Power Automate.
- **Os campos esperados não aparecem:** alguns campos dependem da
  configuração da sua conta. Escreva para o suporte e nós verificamos com
  você.

Para qualquer dúvida, escreva para o suporte e nós te acompanhamos na
configuração.
