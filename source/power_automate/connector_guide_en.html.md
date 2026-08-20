---
title: Connect DataScope with Power Automate
---

# Connect DataScope with Power Automate

> ¿Prefieres leer esto en español o portugués? [Ve a la guía en español](connector_guide_es.html) | [Versão em português](connector_guide_pt.html).

This guide explains how to enable the DataScope integration in Power Automate
to automate your field operation: trigger flows when a form is completed,
when a document is generated, when a task is assigned, or when a ticket is
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
   <a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/apiDefinition.swagger.json" target="_blank" rel="noopener noreferrer"><code>apiDefinition.swagger.json</code></a> file and continue.

## Step 2: review the general configuration

The file already sets the host, the base path and the description, so you
don't need to change anything on this screen.

If you want the connector to display the DataScope logo, download the
<a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/datascope_icon.png" target="_blank" rel="noopener noreferrer">DataScope icon (PNG)</a>
and upload it in the icon field, then set the background color. This is
optional and only affects appearance.

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

## Updating to a newer version of the connector

When we publish a new version of `apiDefinition.swagger.json` (for example, to add a trigger or fix a field), you don't need to create a new connector — you update the existing one:

1. In [make.powerautomate.com](https://make.powerautomate.com), go to **Data > Custom connectors**.
2. Open the DataScope connector, then open its **Swagger editor** (or re-import the file from the **General** tab, depending on the version of the maker portal you're on).
3. Replace the definition with the new file, and select **Update connector**.

A few things worth knowing before you do this:

- **This updates the connector for everyone in the environment**, not just for you — it isn't scoped to a single flow.
- **Already-configured triggers keep working as long as their underlying operation didn't change.** New triggers become available to add to flows; existing ones aren't automatically added to or removed from flows that already use them.
- **Re-check the icon, background color and authentication settings after updating.** Microsoft's own documentation doesn't explicitly confirm whether re-importing a file always preserves those, so treat them as "verify, don't assume" rather than guaranteed to survive the update.
- **If a flow's trigger stops behaving as expected after an update, remove and re-add that trigger's connection** in the affected flow. Microsoft's own guidance for updating a custom connector's definition recommends this as the way to make sure a flow picks up the change cleanly.

## What you can automate

The connector provides the following triggers:

| Trigger | Fires when |
|---|---|
| New answer | A form answer is submitted |
| New PDF | A PDF document is generated |
| Status changed | A form answer changes status |
| New assigned task | A task is assigned |
| New ticket | A ticket is registered |
| Ticket status changed | A ticket changes status |
| Completed signature | A document signature is completed |
| Rejected signature | A signature request is rejected |
| Updated signature | The signatures on a document are updated |

Each trigger delivers the event data as dynamic content, ready to use in the
following steps of the flow without having to parse the JSON manually.

## Important considerations

- **One active connection per form.** Triggers associated with a form support
  only one active connection at a time. If you need several flows on the
  same form, chain them from a single trigger.
- **Ticket triggers operate at the account level.** They support only one
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
  tickets, signatures) does clean up automatically when the flow is turned
  off in Power Automate.

## If something doesn't work

- **The connection fails to create:** check that the API Key is complete,
  with no leading or trailing spaces.
- **The flow doesn't trigger:** confirm there is no other active connection
  on the same form, and review the flow's run history in Power Automate.
- **Expected fields don't appear:** some fields depend on your account's
  configuration. Reach out to support and we'll look into it with you.

For any question, contact support and we'll help you with the setup.
