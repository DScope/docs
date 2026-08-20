---
title: Conectar o DataScope com o Power Automate
---

# Conectar o DataScope com o Power Automate

> Prefere ler isso em espanhol ou inglês? [Versão em espanhol](connector_guide_es.html) | [English version](connector_guide_en.html).

Este guia explica como habilitar a integração do DataScope no Power Automate
para automatizar sua operação em campo: disparar fluxos quando um formulário
é preenchido, quando um documento é gerado, quando uma tarefa é atribuída ou
quando um ticket é registrado.

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
   arquivo <a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/apiDefinition.swagger.json" target="_blank" rel="noopener noreferrer"><code>apiDefinition.swagger.json</code></a> e continue.

## Passo 2: revisar a configuração geral

O arquivo já traz configurados o host, o caminho base e a descrição, então
não é necessário alterar nada nesta tela.

Se quiser que o conector exiba o logo do DataScope, baixe o
<a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/datascope_icon.png" target="_blank" rel="noopener noreferrer">ícone do DataScope (PNG)</a>
e faça o upload no campo de ícone, depois defina a cor de fundo. Isso é
opcional e afeta apenas a aparência.

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

## Atualizar para uma versão mais nova do conector

Quando publicamos uma versão nova do `apiDefinition.swagger.json` (por exemplo, para adicionar um disparador ou corrigir um campo), você não precisa criar um conector novo — você atualiza o que já existe:

1. Em [make.powerautomate.com](https://make.powerautomate.com), vá em **Dados > Conectores personalizados**.
2. Abra o conector do DataScope, depois abra seu **editor do Swagger** (ou importe novamente o arquivo pela aba **Geral**, dependendo da versão do portal em que você está).
3. Substitua a definição pelo arquivo novo e selecione **Atualizar conector**.

Algumas coisas que vale a pena saber antes de fazer isso:

- **Isso atualiza o conector para todos no ambiente**, não só para você — não é algo restrito a um único fluxo.
- **Os disparadores já configurados continuam funcionando enquanto sua operação subjacente não tiver mudado.** Disparadores novos ficam disponíveis para adicionar a fluxos; os que já existem não são adicionados nem removidos automaticamente dos fluxos que já os usam.
- **Reveja o ícone, a cor de fundo e a configuração de autenticação depois de atualizar.** A documentação da Microsoft não confirma explicitamente se reimportar um arquivo sempre preserva isso, então trate como "verifique, não assuma" em vez de algo garantido.
- **Se o disparador de um fluxo parar de se comportar como esperado depois de uma atualização, remova e recrie a conexão desse disparador** no fluxo afetado. A própria orientação da Microsoft para atualizar a definição de um conector personalizado recomenda isso como a forma de garantir que um fluxo capte a mudança corretamente.

## O que você pode automatizar

O conector oferece os seguintes disparadores:

| Disparador | É acionado quando |
|---|---|
| Nova resposta | Uma resposta de formulário é enviada |
| Novo PDF | Um documento PDF é gerado |
| Mudança de status | Uma resposta de formulário muda de status |
| Nova tarefa atribuída | Uma tarefa é atribuída |
| Novo ticket | Um ticket é registrado |
| Mudança de status de ticket | Um ticket muda de status |
| Assinatura concluída | A assinatura de um documento é concluída |
| Assinatura rejeitada | Uma solicitação de assinatura é rejeitada |
| Assinatura atualizada | As assinaturas de um documento são atualizadas |

Cada disparador entrega os dados do evento como conteúdo dinâmico, prontos
para usar nas próximas etapas do fluxo sem precisar processar o JSON
manualmente.

## Considerações importantes

- **Uma conexão ativa por formulário.** Os disparadores associados a um
  formulário admitem apenas uma conexão ativa por vez. Se precisar de vários
  fluxos sobre o mesmo formulário, encadeie-os a partir de um único
  disparador.
- **Os disparadores de tickets operam no nível da conta.** Admitem
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
  (formulários, PDFs, tarefas, tickets, assinaturas) se limpam
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
