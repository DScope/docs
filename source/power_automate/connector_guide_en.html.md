---
title: Connect DataScope with Power Automate
---

# Connect DataScope with Power Automate

> ¿Prefieres leer esto en español o portugués? [Ve a la guía en español](connector_guide_es.html) | [Versão em português](connector_guide_pt.html).

This guide explains how to enable the DataScope integration in Power Automate
to automate your field operation: trigger flows when a form is completed,
when a document is generated, when a task is assigned, or when a ticket is
registered.

<aside class="notice">
You may also find DataScope listed in the official Power Automate connector gallery (<a href="https://preview.flow.microsoft.com/en-us/connectors/shared_datascopeforms/datascope-forms/" target="_blank" rel="noopener noreferrer">preview.flow.microsoft.com</a>, documented at <a href="https://learn.microsoft.com/en-us/connectors/datascopeforms/" target="_blank" rel="noopener noreferrer">learn.microsoft.com</a>). That listing is outdated, so this guide walks you through building the custom connector yourself instead.
</aside>

## Before you start

You need:

- A Power Automate license that enables custom connectors. If you are not
  sure, check with your Microsoft tenant administrator.
- Permission to create connectors in the Power Automate environment you are
  working in.
- Your DataScope API Key. You get it from <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>,
  in the Integrations section of your account.
- The <a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/apiDefinition.swagger.json" target="_blank" rel="noopener noreferrer"><code>apiDefinition.swagger.json</code></a> file provided together with this guide.

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

When we publish a new version of <a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/apiDefinition.swagger.json" target="_blank" rel="noopener noreferrer"><code>apiDefinition.swagger.json</code></a> (for example, to add a trigger or fix a field), you don't need to create a new connector — you update the existing one:

1. In [make.powerautomate.com](https://make.powerautomate.com), go to **Data > Custom connectors**.
2. Open the DataScope connector, then open its **Swagger editor** (or re-import the file from the **General** tab, depending on the version of the maker portal you're on).
3. Replace the definition with the new file, and select **Update connector**.

A few things worth knowing before you do this:

- **This updates the connector for everyone in the environment**, not just for you — it isn't scoped to a single flow.
- **Already-configured triggers keep working as long as their underlying operation didn't change.** New triggers become available to add to flows; existing ones aren't automatically added to or removed from flows that already use them.
- **Re-check the icon, background color and authentication settings after updating.** Microsoft's own documentation doesn't explicitly confirm whether re-importing a file always preserves those, so treat them as "verify, don't assume" rather than guaranteed to survive the update.
- **If a flow's trigger stops behaving as expected after an update, remove and re-add that trigger's connection** in the affected flow. Microsoft's own guidance for updating a custom connector's definition recommends this as the way to make sure a flow picks up the change cleanly.

## What you can automate

The connector provides the following triggers. The names are the ones you see
in Power Automate's trigger list:

| Trigger | Fires when |
|---|---|
| New answer v2 (Forms) | A form answer is submitted, with every question available as its own dynamic content |
| New answer (Forms) (deprecated) | A form answer is submitted. Kept so flows already built on it keep running |
| New PDF (Forms) | A PDF document is generated |
| Status changed (Forms) | A form answer changes status |
| New assigned task (Tasks) | A task is assigned |
| New ticket (Tickets) | A ticket is registered |
| Status changed (Tickets) | A ticket changes status |
| Completed signature (Signatures) | A document signature is completed |
| Rejected signature (Signatures) | A signature request is rejected |
| Updated signature (Signatures) | The signatures on a document are updated |

Each trigger delivers the event data as dynamic content, ready to use in the
following steps of the flow without having to parse the JSON manually.

The connector also provides the following actions, so a flow can act back on DataScope instead of only reacting to it:

| Action | What it does |
|---|---|
| Assign Task | Assigns a task on a form to a user, optionally scheduling it |
| Change Form Status | Changes the status of a form answer |
| Modify Form Answer | Creates or updates an answer within a form response |
| Send Data | Generates a new form answer and its PDF from an existing template |
| Create Ticket | Creates a new ticket |

## The New answer v2 trigger

**New answer v2 (Forms)** is the trigger to use when you build a new flow on a
submitted form answer. It fires when the answer is submitted, and its fields
arrive as dynamic content you can pick directly in the following steps, with no
**Parse JSON** step to add and no schema to paste. Repeatable tables come
through as a list of answer items: put an **Apply to each** over that list, and
the table's columns are available as dynamic content inside the loop.

The earlier **New answer (Forms)** trigger is deprecated, not removed. Flows
already built on it keep running exactly as they do today, and there is no
deadline to move off it. What changes is that it is no longer offered when you
build a new flow, so new work starts on v2.

To move an existing flow, build the new one alongside it, confirm it does what
you expect, and only then delete the old one.

<aside class="warning">
While both flows are active the same form answer is delivered twice, in two different formats, once to each flow. Whatever the flow does happens twice: two work orders, two approvals, two emails. Keep that overlap short, and check the result before you leave both of them running.
</aside>

Two things to plan around:

- **`pdf_url` is opportunistic.** The field is there, but it carries a value
  only when the PDF already exists at the moment the answer is delivered. The
  trigger does not wait for the document to be generated, so a flow that binds
  `pdf_url` can work every time in testing and then arrive with it empty in
  production. If the flow needs the document, build it on the
  **New PDF (Forms)** trigger instead.
- **The field list is a snapshot.** The per-question dynamic content a flow
  sees is captured when the trigger is configured. If questions are added to or
  removed from the form afterwards, the flow does not see them until you reopen
  the trigger and save the flow again. The header fields and the list of answer
  items are not affected, only the per-question shortcuts.

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
- **The New answer (Forms) (deprecated) trigger doesn't clean up on its own.**
  If you turn off or delete a flow that uses it (`hooks_flow`), the
  subscription is not automatically removed on DataScope's side. To fully stop
  it, you also need to go to
  <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>
  and delete the connection there. The other triggers,
  **New answer v2 (Forms)** included, are unsubscribed on DataScope's side when
  the flow is deleted or its trigger is edited, which is what Power Automate
  guarantees. If you stop a flow in any other way and want to be sure nothing
  is left subscribed, delete its connection on that same page.

## If something doesn't work

- **The connection fails to create:** check that the API Key is complete,
  with no leading or trailing spaces.
- **The flow doesn't trigger:** confirm there is no other active connection
  on the same form, and review the flow's run history in Power Automate.
- **Expected fields don't appear:** some fields depend on your account's
  configuration. Reach out to support and we'll look into it with you.

For any question, contact support and we'll help you with the setup.
