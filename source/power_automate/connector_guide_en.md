# Connect DataScope with Power Automate

> ¿Prefieres leer esto en español o portugués? [Ve a la guía en español](https://github.com/DScope/docs/blob/main/source/power_automate/connector_guide_es.md) | [Versão em português](https://github.com/DScope/docs/blob/main/source/power_automate/connector_guide_pt.md).

This guide explains how to enable the DataScope integration in Power Automate
to automate your field operation: trigger flows when a form is completed,
when a document is generated, when a task is assigned, or when a finding is
registered.

## Before you start

You need:

- A Power Automate license that enables custom connectors. If you are not
  sure, check with your Microsoft tenant administrator.
- Permission to create connectors in the Power Automate environment you are
  working in.
- Your DataScope API Key. You get it from **app.mydatascope.com/integrations**,
  in the Integrations section of your account.
- The `apiDefinition.swagger.json` file provided together with this guide.

## Step 1: create the connector

1. Go to [make.powerautomate.com](https://make.powerautomate.com) and sign
   in.
2. Select the environment where you want to create the connector, at the top
   right.
3. In the side menu, open **More** and then **Discover all**.
4. Select **Custom connectors**.
5. Click **New connector** and choose **Import an OpenAPI file**.
6. Enter a name for the connector, for example DataScope, select the
   `apiDefinition.swagger.json` file and continue.

## Step 2: review the general configuration

The file already sets the host, the base path and the description, so you
don't need to change anything on this screen.

If you want the connector to display the DataScope logo, upload it in the
icon field and set the background color. This is optional and only affects
appearance.

## Step 3: configure authentication

The file already declares the authentication type. Verify that the security
screen shows:

- Authentication type: **API Key**
- Parameter name: **Authorization**
- Location: **Header**

Continue to the end of the wizard and select **Create connector**.

## Step 4: create the connection

The first time you use the connector in a flow, Power Automate will ask you
for a connection. Paste your DataScope API Key there.

Each user who builds flows needs to create their own connection with their
account's API Key.

## What you can automate

The connector provides the following triggers:

| Trigger | Fires when |
|---|---|
| New answer | A form answer is submitted |
| New PDF | A PDF document is generated |
| Status changed | A form answer changes status |
| New assigned task | A task is assigned |
| New finding | A finding is registered |
| Finding status changed | A finding changes status |
| Completed signature | A document signature is completed |
| Rejected signature | A signature request is rejected |
| Updated signature | The signatures on a document are updated |

Each trigger delivers the event data as dynamic content, ready to use in the
following steps of the flow without having to parse the JSON manually.

The connector also provides the following actions, so a flow can push data
back into DataScope:

| Action | What it does |
|---|---|
| Assign Task | Assigns a task on a form to a user |
| Change Form Status | Changes the status of a form answer |
| Modify Form Answer | Creates or updates a single question's answer on a form answer |
| Send Data / New Answer [Beta] | Generates a new form answer and its PDF from an existing form template |
| Create Ticket | Creates a new ticket |

## Important considerations

- **One active connection per form.** Triggers associated with a form support
  only one active connection at a time. If you need several flows on the
  same form, chain them from a single trigger.
- **Finding triggers operate at the account level.** They support only one
  active connection per account.
- **Available fields vary by configuration.** Some data, such as planning
  fields or extended task fields, only appears when those features are
  enabled on your account.
- **The connector works in the environment where you created it.** If you
  work with several environments, repeat the import in each one.
- **The "New answer" trigger doesn't clean up on its own.** If you turn off or
  delete a flow that uses the "New answer" trigger (`hooks_flow`), the
  subscription is not automatically removed on DataScope's side. To fully
  stop it, you also need to go to
  [app.mydatascope.com/integrations](https://app.mydatascope.com/integrations)
  and delete the connection there. Every other trigger (forms, PDFs, tasks,
  findings, signatures) does clean up automatically when the flow is turned
  off in Power Automate.

## If something doesn't work

- **The connection fails to create:** check that the API Key is complete,
  with no leading or trailing spaces.
- **The flow doesn't trigger:** confirm there is no other active connection
  on the same form, and review the flow's run history in Power Automate.
- **Expected fields don't appear:** some fields depend on your account's
  configuration. Reach out to support and we'll look into it with you.

For any question, contact support and we'll help you with the setup.
