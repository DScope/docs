---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby

toc_footers:
  - <a href='https://app.mydatascope.com/integrations'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the DataScope API! You can use our API to access DataScope API endpoints, which can get information collected from the DataScope platform and App.

We have language bindings in Shell and Ruby! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

See the [Changelog](#changelog) for what changed recently.




# Authentication



DataScope uses API keys to allow access to the API. You can register a new DataScope API key at our [developer portal](https://app.mydatascope.com/integrations).

![alt text](https://i.imgur.com/M4awUbe.jpg "Logo Title Text 1")

DataScope expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: b1cd93mfls9fdmfkadn23`

<aside class="notice">
You must replace <code>b1cd93mfls9fdmfkadn23</code> with your personal API key.
</aside>

# Data Export

If your goal is a copy of your DataScope data somewhere else, start here rather than with the endpoint reference below.

## Choosing a method

There are several ways to move data out, and they answer different questions.

Method | Direction | Fits when
------ | --------- | ---------
[Webhooks](#webhooks) | DataScope pushes, your endpoint reacts | Something has to happen the moment a form arrives
The REST endpoints | You pull, on your own schedule | You are building a custom integration, or you need a specific slice on demand
[Airbyte Cloud connectors](#airbyte-cloud-connectors) | You pull, Airbyte runs it | The destination is a data warehouse and you want the paging, the incremental state and the deduplication handled for you
[Zapier](https://zapier.com/apps/datascope-forms/integrations) | DataScope pushes, Zapier reacts | You want the same webhook push as a no-code connector, without building your own receiving endpoint
[Microsoft Power Automate](#microsoft-power-automate-beta) | DataScope pushes, Power Automate reacts | Same idea, wired into a Power Automate flow with the event data exposed as dynamic content

A **webhook** fits when something has to happen the moment a form arrives: notify a system, start a workflow, post to a channel. DataScope pushes, your endpoint reacts. It covers new submissions, and edits too when you enable **Send modifications** on the webhook.

The **[Answers V5 endpoint](#get-all-answers-v5-beta)** fits when you need a queryable copy of your data, a table in BigQuery, Snowflake or Postgres that stays current. You pull on a schedule rather than receive pushes, and you get three things in exchange:

What the API adds | How
----------------- | ---
Access to everything, not only what comes next | A webhook starts sending at the moment you configure it and cannot replay what your endpoint missed while it was down. The API takes a `start_date`, so a first sync backfills history and any later sync can re-read a window with `date_modified=true`
A schema you can model | `answers_data_in_array` returns the answers inside a nested array rather than as top-level keys, so the response shape stays the same regardless of the form or how many questions it has. That is what makes a relational destination practical
Control over the payload | `custom_fields` selects exactly which fields you receive, so you can start minimal and add only what your model needs. `version` reproduces the response shape of any earlier API version if you are migrating from one

If the destination is a data warehouse, you do not have to write the receiver, the retry handling or the deduplication yourself: the connectors below cover all of it.

Using both is normal. A webhook for the immediate reaction, a connector for the warehouse copy.

## Airbyte Cloud connectors

DataScope publishes ready to use low-code connector manifests for Airbyte. Each one is plain YAML: import it in the Connector Builder, fill in your token and a start date, and Airbyte handles the paging, the incremental state and the deduplication.

There are two, and they are separate sources on purpose:

Connector | Streams | Covers
--------- | ------- | ------
[Answers](#answers-connector) | `form_answers`, `answers`, `answer_metadata_comments` | Form submissions and their answers
[Signatures](#signatures-connector) | `signature_requests`, `signature_signers` | Signature requests and who signed them

They read different endpoints, with different cursors and different pagination contracts, so one source cannot serve both. Keeping them apart also means that syncing answers does not carry signer personal data into your warehouse unless you ask for it. Import only the one you need.

<aside class="notice">
Both manifests carry a <code>Last updated</code> date in the header comment. Compare it against your imported copy to tell whether you are running the current version.
</aside>

### Answers connector

[Get All Answers V5](#get-all-answers-v5-beta) with the pagination and the incremental state already wired:

<a href="https://raw.githubusercontent.com/DScope/docs/main/source/airbyte/datascope_source_manifest.yaml" target="_blank" rel="noopener noreferrer">Download the manifest (YAML)</a>

It defines three related streams that you can join in your warehouse:

Stream | One row per | Primary key
------ | ----------- | -----------
form_answers | Form answer (one submission) | `form_answer_id`
answers | Answer (one question and its value) | `form_answer_id`, `real_question_id`, `subform_index`, `answer_row_key`
answer_metadata_comments | Checklist comment (text or photo) | the four above plus `data_type` and `data_index`

<aside class="notice">
In the <code>answers</code> and <code>answer_metadata_comments</code> streams, <code>subform_index</code> arrives as <code>-1</code> when the question does not live inside a Group of Repeatable Fields, where the API itself returns <code>null</code>. The manifest normalizes it because Airbyte removes null-valued keys from records, and a primary key cannot reference a field that is missing. Both streams use the same convention, so joining them on <code>(form_answer_id, real_question_id, subform_index, answer_row_key)</code> works without coalescing. <code>-1</code> never collides with a real row 0: a given question is either always inside a repeatable group or never, so the two values cannot both occur for the same <code>real_question_id</code>.
</aside>

To install it: in Airbyte Cloud go to Settings, Sources, "Build a connector", then use the "..." menu and "Import YAML". Configure your API token and a start date, optionally restrict it to specific forms with `form_id`, publish the connector and create the connection with Sync mode "Incremental | Append + Deduped". Older Airbyte versions label that same option "Incremental | Dedup".

<aside class="warning">
Testing a stream shows "Detected schema and declared schema are different", with an <b>Overwrite declared schema</b> button. Do not press it, and do not press <b>Merge properties</b> either. There is no dismiss button: leaving the notice alone is the correct action. It stays as an indicator on the Schema tab but blocks nothing, because the schema declared in the manifest is what runs.
</aside>

<aside class="notice">
Why overwriting breaks things: Airbyte drops any field that is <code>null</code> across every sampled record, since it cannot infer a type from nulls alone, and several fields here are legitimately null on a quiet account. One of them, <code>subform_index</code>, is part of the <code>answer_metadata_comments</code> primary key, so overwriting leaves that key pointing at a field that no longer exists and the stream fails with "Path [] does not have field <code>subform_index</code> in the schema". Everything else the Builder reports is its own normalization: it rewrites <code>$schema</code>, reorders type unions, collapses <code>integer</code> into <code>number</code>, and drops <code>format</code>.
</aside>

### Signatures connector

<a href="https://raw.githubusercontent.com/DScope/docs/main/source/airbyte/datascope_signatures_source_manifest.yaml" target="_blank" rel="noopener noreferrer">Download the manifest (YAML)</a>

It reads [Get Signature Requests](#get-signature-requests) and builds two streams out of the same response:

Stream | One row per | Primary key
------ | ----------- | -----------
signature_requests | Signature request on a form answer | `form_signature_request_id`
signature_signers | Signer inside a request | `user_form_signature_request_id`

Join them on `form_signature_request_id`. Both carry `form_answer_id`, so either one joins to the `form_answers` stream of the answers connector.

To install it, follow the same steps as above. It asks for your API token and a start date, plus an optional `signature_custom_fields` to switch on the field groups described in [Get Signature Requests](#get-signature-requests). It does not take `form_id`: the endpoint has no per-form filter.

<aside class="notice">
A cancelled request is not dropped from the export. It keeps arriving with <code>status</code> set to <code>deleted</code> and a <code>deleted_at</code> timestamp. That matters for an incremental sync, which never observes a disappearance: without the explicit status your warehouse would keep showing a cancelled request as awaiting signature forever.
</aside>

<aside class="notice">
Selecting both streams walks the endpoint twice per sync. Airbyte syncs each stream independently and the declarative connector cannot share one response between two streams, so the cost is two passes rather than one. If you only need request-level data, select just <code>signature_requests</code> and deselect its <code>signers</code> field.
</aside>

<aside class="warning">
The <code>signer_identity</code> group writes names, national ID numbers and personal email addresses into your destination, and they stay there. Turn it on only if your warehouse is an appropriate place for that data.
</aside>

### Customizing a manifest

Either manifest is a starting point, not a black box. It is plain YAML and you can edit it in the Connector Builder before publishing. Some parts are cosmetic, but others are load-bearing: the primary keys, the cursor fields, the extractor paths and a handful of `custom_fields` are what keep the incremental sync and the deduplication correct. Changing those without knowing what they do tends to produce silent problems rather than errors, usually duplicated rows, missing rows, or a stream that returns nothing at all.

The tables below name the answers manifest, and the same rules hold for the signatures one. Its load-bearing parts are the two primary keys, the `since` cursor built from `(updated_at, form_signature_request_id)`, and the extractor paths that flatten `signers` into its own stream.

**Safe to change**

What | Notes
---- | -----
Stream names | `form_answers`, `answers` and `answer_metadata_comments` become table names in your destination. Rename them to match your own conventions
`page_size` | 200 is the endpoint maximum, so you can only lower it. Lower values mean more requests for the same data
Optional `custom_fields` | Add any field from the Custom Fields table above. When you add one, add it to that stream's schema too, so the destination types the column instead of guessing
`form_id` | A configuration field, meant to be set per source. Answers manifest only, the signatures endpoint has no per-form filter
`signature_custom_fields` | A configuration field on the signatures manifest, meant to be set per source

**Change with care**

What | What breaks
---- | -----------
`order_date`, or the sort direction | Incremental sync depends on ascending `updated_at`. Descending puts the newest record on the first page, the cursor jumps to it, and everything behind it is never synced again
`primary_key` on any stream | These tuples are what make deduplication correct across edits. Shortening one collapses rows that are actually distinct; adding a mutable field creates a new row on every edit
`cursor_field`, or the `incremental_sync` blocks | The child streams inherit the parent form answer's `updated_at`. Pointing them somewhere else stalls the sync state
Extractor `field_path` | The wildcard shape is exact. A wrong path returns zero records with no error, which looks like an empty account
The structural `custom_fields` | `answers_data_in_array`, `answers_extra_data`, `answers_row_key`, `answers_form_answer_updated_at` and `answers_metadata_comments_array` feed the primary keys and the cursors. Removing one breaks whatever depended on it
Adding `limit` to `request_parameters` | The paginator already injects it, and duplicating it fails the sync with a request collision

<aside class="notice">
Rename streams before the first sync if you can. Airbyte treats a renamed stream as a new one, so renaming later creates a fresh table in your destination and re-syncs the history into it, leaving the previous table behind.
</aside>

<aside class="warning">
If you edit the manifest and the sync starts behaving oddly, re-import the published version and reapply your changes one at a time. Comparing against a known good copy is faster than debugging a modified manifest from scratch.
</aside>

### Rate limiting

> Uncomment the `api_budget` block that ships in the manifest to cap the request rate:

```yaml
api_budget:
  type: HTTPAPIBudget
  status_codes_for_ratelimit_hit: [429]
  policies:
    - type: MovingWindowCallRatePolicy
      rates:
        - limit: 1
          interval: "PT1S"
      matchers:
        - method: GET
          url_base: "https://www.mydatascope.com"
          url_path_pattern: "^/api/external/"
```

The manifests as published do not cap their request rate, and out of the box they do not need to. Airbyte requests a stream's pages one at a time, waiting for each response before asking for the next, so page latency already spaces the calls out.

Two situations change that, and both manifests ship a commented `api_budget` block for them:

When | Why
---- | ---
Several streams syncing in parallel | Each stream opens its own sequence of requests, so two streams at once are two simultaneous requests
Both connectors running at the same time | The answers and the signatures connector each make their own calls

The part worth knowing before you tune anything: **the limit is counted per source IP, not per token.** Everything leaving the same Airbyte deployment shares one budget, even when each source is configured with a different token.

The value in the block is deliberately conservative rather than set to the actual ceiling. Sitting exactly on a limit crosses it anyway, because requests do not arrive evenly spaced inside a second.

<aside class="warning">
An <code>api_budget</code> applies to the source that declares it, and Airbyte does not share it across sources. Two sources each capped at a given rate add up to twice that rate against the same IP, so if you run the answers and the signatures connector concurrently, lower the rate on each or stagger their schedules.
</aside>

<aside class="notice">
A rejected request answers 429 with an HTML body and no <code>Retry-After</code> header, so a wait strategy that reads timing from headers has nothing to read. Prefer a fixed rate like the one above, or a constant backoff.
</aside>

## Microsoft Power Automate [Beta]

DataScope publishes a custom connector for Microsoft Power Automate, so you can trigger flows from form answers, generated PDFs, task assignments, findings and signature events, delivered as dynamic content instead of raw webhook JSON.

<aside class="notice">
We have had ongoing trouble keeping the connector published in the official Power Automate connector gallery in sync with the latest version. Until that is resolved, the fastest way to connect is to build the custom connector yourself from the definition file below.
</aside>

<a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/apiDefinition.swagger.json" target="_blank" rel="noopener noreferrer">Download the connector definition (JSON)</a>

Follow the <a href="power_automate/connector_guide_en.html" target="_blank" rel="noopener noreferrer">step-by-step guide</a> to import it and create your first connection. Spanish and Portuguese versions are linked from the top of that guide.

# Answers

## Get All Answers

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/v2/answers'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/v2/answers"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
      "form_name":"Example Form",
      "form_state":"Accepted",
      "user_name":"Example User",
      "user_identifier":"user@email.com",
      "code":"2342",
      "form_id":432,
      "created_at":"2018-04-16T16:52:05.000Z",
      "form_answer_id":257189,
      "latitude":-33.398803,
      "longitude":-70.559834,
      "[question_name1]": "[question_value1]",
      "[question_name2]": "[question_value2]",
      "[question_name3]": "[question_value3]"
   }
]

```

This endpoint retrieves last answers (Limit 200)

### HTTP Request

`GET https://www.mydatascope.com/api/external/v2/answers`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
form_id | blank | If set, only get values of one form. This ID is in the URL at the moment of modify one form. eg. https://mydatascope.com/task_forms/XXXX/edit
  user_id | blank | If set, get values of only one user
  start | last 7 days | set the start date range
  end | today | set the end date range (Max range is 90 days)
  location_id | blank | set the answers of onlye one location
  date_modified | false | Bring forms by modification date or just new ones
  limit | 200 | Number of submission, default and max is 200 
  offset | 0 | pagination to bring next 200 submissions in some date range


### Output Parameter

Parameter | Type | Description
--------- | ------- | -----------
  form_code | String | Public identifier of the form answer.
  form_state | String | Last status of the form answer.
  form_id | integer | Internal code of the form, fixed to all answers of that form.
  form_answer_id | Integer | Internal code of the form answer.
  form_name | String | Name of the form.
  user_name | String | Name of the user.
  created_at | Date | When the form was received.
  latitude | Float | Latitude where the form was answered.
  longitude | Float | Longitude where the form was answered.
  question_name | (String, Date, Number) | String with each question name and value.



## Get All Answers with metadata

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/answers'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/answers"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
      "answers":[  
         {  
            "question_name":"Client Name", 
            "name":"Client", 
            "question_value":"Company 1",
            "question_type":"select_metadata",
            "subform_index":0,
            "metadata_type": "locations",
            "metadata_id": 4234,
            "question_id":1, 
            "form_code":"2342", 
            "form_state": "Accepted",
            "form_id":34543,
            "form_answer_id":432432
         }
      ],
      "form_name":"Example Form",
      "form_state":"Accepted",
      "user_name":"Example User",
      "code":"2342",
      "form_id":432,
      "created_at":"2018-04-16T16:52:05.000Z",
      "form_answer_id":257189,
      "latitude":-33.398803,
      "longitude":-70.559834
      "assign_id":"A32",
      "assign_internal_id":"4322",
      "assign_location_name":"Client 1 Factory A",
      "assign_location_description":"description Client 1",
      "assign_location_code":"client1",
   }
]

```

This endpoint retrieves last answers (Limit 600)

### HTTP Request

`GET https://www.mydatascope.com/api/external/answers`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
form_id | blank | If set, only get values of one form
  user_id | blank | If set, get values of only one user
  start | last 7 days | set the start date range
  end | today | set the end date range
  location_id | blank | set the answers of onlye one location


### Output Parameter

Parameter | Type | Description
--------- | ------- | -----------
answers | Array | array with all the question of the specific form answer.
  question_name | String | The name of the grouped question.
  name | String | Name of the specific Question.
  question_type | String | Type of the question.
  subform_index | Integer | If use a subform indicate the iteration inside the subform.
  metadata_id | Integer | Identifier of the metadata object of the List
  metadata_type | String | Identifier of the list of metadata (location, products, and more)
  question_id | Integer | Internal identifier of the question.
  form_code | String | Public identifier of the form answer.
  form_state | String | Last status of the form answer.
  form_id | integer | Internal code of the form, fixed to all answers of that form.
  form_answer_id | Integer | Internal code of the form answer.
  form_name | String | Name of the form.
  user_name | String | Name of the user.
  created_at | Datetime | When the form was received.
  latitude | Float | Latitude where the form was answered.
  longitude | Floar | Longitude where the form was answered.
  finished | Boolean | Identify if synchronization process finished.
  updated_at | Datetime | Last date and time When the form was updated
  assign_id | Integer | Assign ID generated by user
  assign_internal_id | Integer | Assign ID generated by system
  assign_location_name | String | Name of the location of the assigned Task
  assign_location_description | String | Description of the location assigned
  assign_location_code" | String | Code of the location assigned

## Get All Answers V5 (Beta)

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/v5/answers'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {
   :date_modified => true,
   :order_date => true,
   :custom_fields => 'answers_data_in_array,answers_extra_data'
 }
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/v5/answers?date_modified=true&order_date=true&custom_fields=answers_data_in_array,answers_extra_data"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> With `answers_data_in_array` the questions are nested in an `answers` array instead of becoming top-level keys, so the response shape stays the same across forms:

```json
[
   {
      "form_answer_id":257189,
      "form_id":432,
      "form_name":"Example Form",
      "form_code":"2342",
      "form_state":"Accepted",
      "user_name":"Example User",
      "user_identifier":"user@email.com",
      "created_at":"2026-07-16T16:52:05.000Z",
      "updated_at":"2026-07-18T09:14:22.000Z",
      "latitude":-33.398803,
      "longitude":-70.559834,
      "answers":[
         {
            "question_value":"Example value",
            "question_name":"Example question",
            "question_type":"text",
            "question_id":1180,
            "real_question_id":1180,
            "subform_index":null,
            "metadata_type":null,
            "metadata_id":null
         },
         {
            "question_value":"Alternative 1",
            "question_name":"Example checklist",
            "question_type":"select_check_metadata",
            "question_id":1181,
            "real_question_id":1181,
            "subform_index":null,
            "metadata_type":"list_objects",
            "metadata_id":90412
         }
      ]
   }
]
```

Answers V5 is a single endpoint that can reproduce the response shape of every previous version (v1, v2, v3, v4) and adds opt-in fields on top. Instead of a new endpoint per feature, you pick the version you want as a baseline through `version` and switch extra data on through `custom_fields`.

The most requested use case is the `answers_data_in_array` shape shown above. In v1 to v4 each question becomes its own top-level key, so every form produces a different set of columns and a relational destination (BigQuery, Snowflake, Postgres) cannot model it cleanly. With `answers_data_in_array` the questions move into a nested `answers` array and the response schema stops changing per form.

<aside class="notice">
This endpoint is in Beta. It is stable and in production use, but the field list keeps growing. The companion page <a href="https://datascopers.notion.site/Explicaci-n-Answers-V5-Beta-1a03acd7c7a08005b827d66010b5ff1c">Explicación Answers V5</a> (written in Spanish) walks through the same material and tracks the change history release by release. Feedback and bug reports are welcome.
</aside>

<aside class="warning">
The user whose token you use needs export permission in DataScope. Ask your account administrator to confirm it before setting up a recurring integration.
</aside>

### HTTP Request

`GET https://www.mydatascope.com/api/external/v5/answers`

### Query Parameters

Parameter | Type | Default | Description
--------- | ---- | ------- | -----------
version | String | v5 | Reproduce the response shape of a previous version. One of `v1`, `v2`, `v3`, `v4`, `v5`.
custom_fields | String | blank | Comma separated list of extra data to include. See the table below.
form_id | Integer or String | blank | One form ID, or several comma separated. This ID is in the URL when you edit a form, eg. https://app.mydatascope.com/task_forms/XXXX/edit
user_id | Integer | blank | If set, only answers from that user
location_id | Integer | blank | If set, only answers from that location
days_limit | Integer | 7 | Days between start and end when you do not send them (max range 90 days)
start | String | last `days_limit` days | Start of the date range, ISO 8601 (eg. `2026-07-01T00:00:00Z`)
end | String | date of the last answer | End of the date range, ISO 8601. Max range 90 days
date_modified | Boolean | false | Filter by modification date (`updated_at`) instead of creation date. Use it to pull edits, not only new submissions
order_date | Boolean | false | Sort by `updated_at` instead of the default question order. Required whenever you page through modified answers
sort_order | String | asc | Direction for `order_date`. `asc` or `desc`. Keep `asc` for any incremental integration
limit | Integer | 200 | Records per page. Default and max are 200
page | Integer | 1 | Page number. With a limit of 200, page 2 returns the next 200 records
offset | Integer | 0 | Shifts the start of the pagination
since | String | blank | Keyset pagination cursor, built from the `updated_at` and `form_answer_id` of the last record you read. More efficient than `page` or `offset` on large exports. See "Pagination with since" below

<aside class="notice">
Send <code>date_modified=true</code> and <code>order_date=true</code> together whenever you page through modified answers. Filtering by modification date while sorting by something else drops records at page boundaries.
</aside>

### Pagination with since

`page` and `offset` work as in previous versions. For large or recurring exports prefer `since`, which pages from the last record you already read instead of counting rows from the beginning:

1. Request the first page normally, with `date_modified=true` and `order_date=true`.
2. Take `updated_at` and `form_answer_id` from the last record of the response.
3. Send them back as `since=<updated_at>|<form_answer_id>` to get the next page.
4. Repeat until a page returns fewer records than `limit`.

A malformed cursor returns `400` with `{"error": "invalid_since_cursor"}`. When `since` is present the response is always sorted ascending, regardless of `sort_order`.

### Custom Fields

`custom_fields` unlocks extra data. Send the names comma separated, for example `custom_fields=answers_data_in_array,answers_extra_data,answers_selected_metadata`. Unknown names are ignored silently.

Custom Field | Description
------------ | -----------
answers_data_in_array | Nest the questions in an `answers` array instead of top-level keys. This is what keeps the schema stable across forms (already in v1)
answers_extra_data | Per answer metadata: `question_id`, `real_question_id`, `subform_index`, `question_type`, `metadata_type`, `metadata_id`, `name`
answers_latitude_longitude | Latitude and longitude of each answer (already in v2 and v4)
answers_selected_metadata | Name, description, code and attributes of the list object or location selected in an answer
answers_activity_data | `start_time`, `end_time`, `duration`, `day_start` and `full_duration` for Activity List answers
answers_activity_order | Period index for Activity List answers. Distinguishes several periods that selected the same alternative
answers_row_key | Stable per row discriminator. Combined with `form_answer_id`, `real_question_id` and `subform_index` it gives each answer row a primary key that survives edits
answers_form_answer_updated_at | `updated_at` of the parent form answer repeated inside each answer. Useful as an incremental cursor when you consume the answers as their own table
answers_metadata_comments_array | Checklist comments as a nested `metadata_comments` array on each answer, instead of flat prefixed columns
answers_form_identification | `form_name` and `form_code` repeated inside each answer, so you do not need to join back to the form answer
answers_comments | Checklist comments as flat prefixed columns (`comment`, `comment_1`, ...). Already in versions above 2. Do not combine with `answers_metadata_comments_array`
answers_as_v3 | Reproduce the special v3 answer format
code_as_form_code | Include the answer code as `form_code` (already in versions above 2)
form_answer_id_as_id | Include the form answer ID as `id` (already in v3)
form_finished | Include whether the form synchronized completely (already in v3)
form_update_variations | Include `updated_date` and `updated_at_unix`
assign_base_data | Basic data of the assigned Task: internal ID, custom ID, name, description, code. Required by the `task_*` and `assign_location_*` fields below
task_description | Description of the assigned Task
task_mandatory | Whether the assigned Task is mandatory
task_late_response_allowed | Whether the assigned Task accepts answers after the deadline
task_group_id | Group ID of the assigned Task
task_mobile_user_id | User ID of the assigned Task
task_start_time | Start datetime of the assigned Task
task_gap | Hour window to complete the assigned Task
assign_location_city | City of the location of the assigned Task (already in v3)
assign_location_region | Region of the location of the assigned Task
assign_location_country | Country of the location of the assigned Task
assign_location_email | Email of the location of the assigned Task
assign_location_latitude | Latitude of the location of the assigned Task
assign_location_longitude | Longitude of the location of the assigned Task
assign_location_company_email | Company email of the location. Only for Locations
assign_location_company_name | Company name of the location. Only for Locations
assign_location_company_code | Company code of the location. Only for Locations

<aside class="notice">
If your destination is a data warehouse, you do not need to implement the paging and the incremental logic yourself. The <a href="#answers-connector">Answers connector</a> wires this endpoint into Airbyte Cloud for you.
</aside>

## Change Answer

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/change_form_answer'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/change_form_answer"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
      "status": "ok",
      "form_answer": {
        "id": "4325235",
        "form_id": "6344234"
      }
   }
]

```


### HTTP Request

`POST https://www.mydatascope.com/api/external/change_form_answer`

### Query Parameters

Parameter |  Description
--------- | -----------
form_name | name of ID of the form
form_code | Code of the response
question_name | Name of the question to change
question_value | Value of the question to change
subform_index | Number to specify the subform index (Starting from 1). Leave blank if question it's not inside subform*.


# Signatures

## Get Signature Requests

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/signatures/list'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {
   :start => '2026-01-01',
   :limit => 200,
   :custom_fields => 'signer_identity,documents'
 }
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/signatures/list?start=2026-01-01&limit=200&custom_fields=signer_identity,documents"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The response wraps the rows in an object, together with the cursor for the next page:

```json
{
   "signature_requests":[
      {
         "form_signature_request_id":48210,
         "form_answer_id":257189,
         "status":"pending",
         "deleted_at":null,
         "required":true,
         "sequential":true,
         "sequence_step":1,
         "required_signatures_count":2,
         "reject_reason":null,
         "rejected_by_external":false,
         "created_at":"2026-07-16T16:52:05Z",
         "closed_at":null,
         "updated_at":"2026-07-18T09:14:22Z",
         "signer_ids":[90411, 90412],
         "signers":[
            {
               "user_form_signature_request_id":90411,
               "form_signature_request_id":48210,
               "form_answer_id":257189,
               "form_signature_request_updated_at":"2026-07-18T09:14:22Z",
               "signed":true,
               "external_user":false,
               "sequence_order":1,
               "cancelled_by_quorum":false,
               "rejected":false,
               "created_at":"2026-07-16T16:52:05Z",
               "updated_at":"2026-07-17T11:03:41Z"
            },
            {
               "user_form_signature_request_id":90412,
               "form_signature_request_id":48210,
               "form_answer_id":257189,
               "form_signature_request_updated_at":"2026-07-18T09:14:22Z",
               "signed":false,
               "external_user":true,
               "sequence_order":2,
               "cancelled_by_quorum":false,
               "rejected":false,
               "created_at":"2026-07-16T16:52:05Z",
               "updated_at":"2026-07-16T16:52:05Z"
            }
         ]
      }
   ],
   "next_cursor":"2026-07-18T09:14:22Z|48210"
}
```

Returns the signature requests of your account with their signers nested, ordered by `updated_at` ascending and paginated with a keyset cursor. It is built for incremental sync: walk from the beginning once, then keep sending back the cursor you received.

This is what answers "which of the five signers is still missing", over time. Each signer keeps a stable identity across syncs, unlike the positional columns of the spreadsheet integration.

If your destination is a data warehouse, you do not need to implement any of the pagination below: the [Signatures connector](#signatures-connector) does it for you.

<aside class="warning">
The user whose token you use needs export permission in DataScope, the same permission the Answers endpoints require. Ask your account administrator to confirm it before setting up a recurring integration.
</aside>

### HTTP Request

`GET https://www.mydatascope.com/api/external/signatures/list`

### Query Parameters

Parameter | Type | Default | Description
--------- | ---- | ------- | -----------
start | String | blank | Only requests modified from this moment on. ISO 8601. There is no default window: leaving it out walks your whole history, which is what a first sync wants
end | String | blank | Only requests modified up to this moment. ISO 8601. Defaults to now, excluding the second in progress
limit | Integer | 200 | Rows per page. 200 is also the maximum, so you can only lower it
since | String | blank | Cursor for the next page. Pass back the `next_cursor` you received
custom_fields | String | blank | Comma separated list of optional field groups. See below

<aside class="notice">
<code>start</code> and <code>end</code> only accept ISO 8601. A format like <code>08/01/2026</code> is rejected with a 400 rather than interpreted, because day-first and month-first cannot be told apart and guessing would silently return data from the wrong month with a 200.
</aside>

### Pagination

The cursor is the pair `(updated_at, form_signature_request_id)`, serialized as `<ISO8601>|<id>`. Read `next_cursor` from the response and send it back as `since` to get the following page. When a page comes back with fewer rows than `limit`, you have reached the end.

It is a pair rather than a timestamp alone for a concrete reason: `updated_at` has one-second resolution, and a bulk signature writes a whole batch of rows within the same second. A cursor carrying only the timestamp, whose page boundary falls inside such a batch, would skip the rest of it permanently. Adding the id makes the ordering total.

<aside class="warning">
A cursor whose timestamp is in the future is rejected with a 400. It can only come from clock skew or a corrupted token, and it would otherwise make every later page come back empty, which reads as "fully caught up".
</aside>

### Response

Field | Description
----- | -----------
signature_requests | The page of requests, each with its signers nested
next_cursor | Cursor for the next page, or `null` when the page is empty

### Custom Fields

Both groups are opt-in because each has a real cost when you do not want it.

Value | Adds
----- | ----
signer_identity | Identity of each signer (name, national ID, emails, company, role, country) and of whoever requested the signatures. This is personal data, and it lands in your destination permanently
documents | The signed PDF link, when that PDF was generated, and a link to the answer

<aside class="notice">
An unrecognized value returns a 400 listing the accepted ones, instead of omitting the field silently. A silent omission would answer 200 with the field missing, which reads as "that field does not exist" and turns a typo into a long debugging session.
</aside>

### Request Fields

Field | Description
----- | -----------
form_signature_request_id | ID of the signature request. Primary key
form_answer_id | The answer being signed. Join key against the Answers endpoints
status | `pending`, `completed`, `rejected`, or `deleted` for a cancelled request
deleted_at | When the request was cancelled. Only present on a `deleted` request
required | Whether signing is mandatory for this answer
sequential | Whether signers have to sign in order
sequence_step | Current step, on a sequential request
required_signatures_count | Signatures needed to close the request. Already resolved: it accounts for a total quorum, a per-step quorum, and the case with no quorum, where it equals the number of signers
reject_reason | Free text reason, when the request was rejected
rejected_by_external | Whether the rejection came from an external signer
signer_ids | The authoritative list of signer IDs for this request
signers | The signers, described below
created_at | When the request was created
closed_at | When the request was completed or rejected
updated_at | Last modification. This is the cursor field

<aside class="notice">
Use <code>signer_ids</code> to retire signers. A signer removed while editing the request is deleted outright rather than flagged, so diffing your stored rows against this list is the only way to notice it is gone.
</aside>

### Signer Fields

Field | Description
----- | -----------
user_form_signature_request_id | ID of the signer row. Primary key
form_signature_request_id | Parent request
form_answer_id | Denormalized from the parent, so the join key is on the row itself
form_signature_request_updated_at | The parent's `updated_at`. Present because the cursor is built from the request, so a signers-only consumer can still page
signed | Whether this signer signed
external_user | Whether the signer is external, rather than a DataScope user
sequence_order | Position, on a sequential request
cancelled_by_quorum | True when this signer's turn became moot because the request already reached its quorum
rejected | True on the signer who rejected. Always present, no need for `signer_identity`
created_at | When the signer was added
updated_at | Last modification. On a signer who signed, this is the moment of the signature

### Signer Identity Fields

Only with `signer_identity`.

Field | Description
----- | -----------
full_name | Signer's name. Falls back to the DataScope user's name when the signer has no signature configuration yet
rut | National ID number, when recorded
personal_email | Personal email, falling back to the DataScope user's address
company_email | The DataScope user's address
company_name | Company recorded in the signature configuration
role | Role recorded in the signature configuration
country | Country recorded in the signature configuration
mobile_user_id | The DataScope user, when the signer is internal
requester_mobile_user_id | On the request: who asked for the signatures
requester_full_name | On the request: that person's name
requester_email | On the request: that person's email

<aside class="notice">
A signer who has not signed yet may have no signature configuration at all, since it is created at the moment of signing. That is why the name and the personal email fall back to the DataScope user: without the fallback these come back empty for exactly the signers you most want to chase.
</aside>

### Document Fields

Only with `documents`.

Field | Description
----- | -----------
pdf_url | Link to the signed PDF
pdf_generated_at | When that PDF was generated
answer_view_url | Link to the answer in DataScope

<aside class="notice">
The PDF is regenerated asynchronously as signatures come in, so <code>pdf_url</code> can point at a version that predates the newest signature. Compare <code>pdf_generated_at</code> against the request's <code>updated_at</code> to detect it.
</aside>

# Locations

## Get All Locations

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/locations'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/locations"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
    "id":432432,
    "name":"Client ABC",
    "description":"Company ABC description",
    "code":"534534",
    "address":"1600 Amphitheatre Parkway, Mountain View, CA",
    "city":"SF",
    "country":"US",
    "latitude":37.395013,
    "longitude":-122.084374,
    "region":"CA",
    "phone":"4324234",
    "company_code":"432432",
    "company_name":"Client ABC S.A" 
   }
]

```

This endpoint retrieves all locations

### HTTP Request

`GET https://www.mydatascope.com/api/external/locations`

### Output Parameter

Parameter | Type | Description
--------- | ------- | -----------
id | Integer | Internal identifier of the location
name | String | Name of the location
description | String | Description of the location
code | String | Code of the location
address | Integer | Address of the location
city | Integer | City
country | String | Country
latitude | String |  Latitude GPS Coordinates
longitude | integer | Longitude GPS Coordinates
region | Integer | Region
phone | String | Phone
company_code | String | Code of the company
company_name | Date | Name of the Company

<aside class="success">
Remember — use your own Authorization header
</aside>

## Create a Location

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/locations'
response = RestClient.post url, {
  location: {
    name: "Test Location",
    description: "This is a test location created by API",
    code: "LOC_TEST01",
    company_code: "DSCODE_1",
    company_name: "Datascope Home",
    address: "P. Sherman, 42 Wallaby Way",
    city: "Sydney",
    country: "Australia",
    latitude: -33.673992,
    longitude: 151.285829,
    phone: "+18888888",
    email: "location@test.com"
  }
}.to_json, {
 :Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/locations"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
  -X POST
  -d '{
    "location": {
      "name": "Test Location",
      "description": "This is a test location created by API",
      "code": "LOC_TEST01",
      "company_code": "DSCODE_1",
      "company_name": "Datascope Home",
      "address": "P. Sherman, 42 Wallaby Way",
      "city": "Sydney",
      "country": "Australia",
      "latitude": -33.673992,
      "longitude": 151.285829,
      "phone": "+18888888",
      "email": "location@test.com"
    }
  }'
```

> When successfull the above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
    "id": "123456",
    "name": "Test Location",
    "description": "This is a test location created by API",
    "code": "LOC_TEST01",
    "company_code": "DSCODE_1",
    "company_name": "Datascope Home",
    "address": "P. Sherman, 42 Wallaby Way",
    "city": "Sydney",
    "country": "Australia",
    "latitude": -33.673992,
    "longitude": 151.285829,
    "phone": "+18888888",
    "email": "location@test.com"
}

```

This endpoint create a location

### HTTP Request

`POST https://www.mydatascope.com/api/external/locations`

### Input Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the location
description | String | Description of the location
code | String | Code of the location
address | Integer | Address of the location
city | Integer | City
country | String | Country
latitude | String |  Latitude GPS Coordinates
longitude | integer | Longitude GPS Coordinates
phone | String | Phone
company_code | String | Code of the company
company_name | Date | Name of the Company
email | String | Email of the Company

### Return Codes:

Code  | Description
----  | -----------
201   | Successfull
403   | Forbidden
422   | Wrong parameters, check documentation

<aside class="success">
Remember — use your own Authorization header
</aside>


## Update a Location

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/locations/123456'
response = RestClient.post url, {
    location: {
      name: "Test Location",
      description: "This is a test location created by API",
      code: "LOC_TEST01",
      company_code: "DSCODE_1",
      company_name: "Datascope Home",
      address: "P. Sherman, 42 Wallaby Way",
      city: "Sydney",
      country: "Australia",
      latitude: -33.673992,
      longitude: 151.285829,
      phone: "+18888888",
      email: "location@test.com"
    }
}.to_json, {
 :Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/locations/123456"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
  -X POST
  -d '{
    "location": {
      "name": "Test Location",
      "description": "This is a test location created by API",
      "code": "LOC_TEST01",
      "company_code": "DSCODE_1",
      "company_name": "Datascope Home",
      "address": "P. Sherman, 42 Wallaby Way",
      "city": "Sydney",
      "country": "Australia",
      "latitude": -33.673992,
      "longitude": 151.285829,
      "phone": "+18888888",
      "email": "location@test.com"
    }
  }'
```

> When successfull the above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
    "id": "123456",
    "name": "Test Location",
    "description": "This is a test location created by API",
    "code": "LOC_TEST01",
    "company_code": "DSCODE_1",
    "company_name": "Datascope Home",
    "address": "P. Sherman, 42 Wallaby Way",
    "city": "Sydney",
    "country": "Australia",
    "latitude": -33.673992,
    "longitude": 151.285829,
    "phone": "+18888888",
    "email": "location@test.com"
}

```

This endpoint updates a location

### HTTP Request

`POST https://www.mydatascope.com/api/external/locations`

### Input Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the location
description | String | Description of the location
code | String | Code of the location
address | Integer | Address of the location
city | Integer | City
country | String | Country
latitude | String |  Latitude GPS Coordinates
longitude | integer | Longitude GPS Coordinates
phone | String | Phone
company_code | String | Code of the company
company_name | Date | Name of the Company
email | String | Email of the Company

### Return Codes:

Code  | Description
----  | -----------
201   | Successfull
403   | Forbidden
422   | Wrong parameters, check documentation

<aside class="success">
Remember — use your own Authorization header
</aside>



# Lists

## Get All List elements

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/metadata_objects'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => { metadata_type: 'products_examples'}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/metadata_objects"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
      "id":505680,
      "name":"Product 1",
      "description":"Product Description",
      "attribute1":"Atribute example 1",
      "attribute2":"Attribute example 2",
      "list_id":424324,
      "account_id":4234234,
      "code":"prod0",
      "created_at":"2015-12-03T17:48:47.000-02:00",
      "updated_at":"2015-12-03T17:48:47.000-02:00"
   },
   {  
      "id":505689,
      "name":"Product 2",
      "description":"Product Description",
      "attribute1":"Atribute example 1",
      "attribute2":"Attribute example 2",
      "list_id":424324,
      "account_id":4234234,
      "code":"prod9",
      "created_at":"2018-12-03T17:48:47.000-02:00",
      "updated_at":"2018-12-03T17:48:47.000-02:00"
   }
]


```

This endpoint retrieves all list items of a specific list

### HTTP Request

`GET https://www.mydatascope.com/api/external/metadata_objects`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
metadata_type | blank | Internal code to identify the list (products, locations and more)

### Output Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the element of the list
description | String | Description of the element of the list
attribute1 | String | Custom attribute of the element of the list
attribute2 | String | Custom attribute of the element of the list
created_at | Datetime | Date when the list element was created
updated_at | Datetime | Date when the list element was updated

## Get an element of the list

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/metadata_object'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => { metadata_type: 'products_examples', metadata_id: 4324324}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/metadata_object"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
{  
  "id":505680,
  "name":"Product 1",
  "description":"Product Description",
  "attribute1":"Atribute example 1",
  "attribute2":"Attribute example 2",
  "list_id":424324,
  "account_id":4234234,
  "code":"prod0",
  "created_at":"2015-12-03T17:48:47.000-02:00",
  "updated_at":"2015-12-03T17:48:47.000-02:00"
}


```

This endpoint retrieves a specific element of the list

### HTTP Request

`GET https://www.mydatascope.com/api/external/metadata_object`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
metadata_type | blank | Internal code to identify the list (products, locations and more)
metadata_id | blank | Internal identifier of the element of the list

### Output Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the element of the list
description | String | Description of the element of the list
attribute1 | String | Custom attribute of the element of the list
attribute2 | String | Custom attribute of the element of the list
created_at | Datetime | Date when the list element was created
updated_at | Datetime | Date when the list element was updated


## Create a List Element

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/metadata_object?metadata_type=LIST_TEST2'
response = RestClient.post url, {
  list_object: {
    name: "Test List Object 2",
    description: "This is a test Object created by API",
    code: "LIST_OBJECT_TEST2",
    attribute1: "ATTR1",
    attribute2: "ATTR2"
  }
}.to_json, {
 :Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/metadata_object?metadata_type=LIST_TEST2"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
  -X POST
  -d '{
    "list_object": {
      "name": "Test List Object 2",
      "description": "This is a test Object created by API",
      "code": "LIST_OBJECT_TEST2",
      "attribute1": "ATTR1",
      "attribute2": "ATTR2"
    }
  }'
```

> When successfull the above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
    "id": 123456,
    "name": "Test List Object 2",
    "description": "This is a test Object created by API",
    "code": "LIST_OBJECT_TEST2",
    "created_at": "2024-09-05T06:31:59.000-03:00",
    "updated_at": "2024-09-05T06:31:59.000-03:00",
    "metadata_type": "LIST_TEST2"
}

```

This endpoint creates a list element

### HTTP Request

`POST https://www.mydatascope.com/api/external/metadata_object`

### Query params:
Parameter | Type | Description
--------- | ------- | -----------
metadata_type | blank | Internal code to identify the list (products, and more*)

*For locations objects use locations API

### Input Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the element of the list
description | String | Description of the element of the list
code | String | Internal code of the element of the list
attribute1 | String | Custom attribute of the element of the list
attribute2 | String | Custom attribute of the element of the list

### Return Codes:

Code  | Description
----  | -----------
201   | Successfull
403   | Forbidden
422   | Wrong parameters, check documentation

<aside class="success">
Remember — use your own Authorization header
</aside>


## Update a List Element

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/metadata_object/123456'
response = RestClient.post url, {
  list_object: {
    name: "Test List Object 2",
    description: "This is a test Object created by API",
    code: "LIST_OBJECT_TEST2",
    attribute1: "ATTR1",
    attribute2: "ATTR2"
  }
}.to_json, {
 :Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/metadata_object/123456"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
  -X POST
  -d '{
    "list_object": {
      "name": "Test List Object 2",
      "description": "This is a test Object created by API",
      "code": "LIST_OBJECT_TEST2",
      "attribute1": "ATTR1",
      "attribute2": "ATTR2"
    }
  }'
```

> When successfull the above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
    "id": 123456,
    "name": "Test List Object 2",
    "description": "This is a test Object created by API",
    "code": "LIST_OBJECT_TEST2",
    "created_at": "2024-09-05T06:31:59.000-03:00",
    "updated_at": "2024-09-05T06:31:59.000-03:00",
    "metadata_type": "LIST_TEST2"
}

```

This endpoint updates a list object

### HTTP Request

`POST https://www.mydatascope.com/api/external/metadata_object/{id}`

### Input Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the element of the list
description | String | Description of the element of the list
code | String | Internal code of the element of the list
attribute1 | String | Custom attribute of the element of the list
attribute2 | String | Custom attribute of the element of the list

### Return Codes:

Code  | Description
----  | -----------
201   | Successfull
403   | Forbidden
422   | Wrong parameters, check documentation

<aside class="success">
Remember — use your own Authorization header
</aside>


## Create a empty List

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/metadata_types'
response = RestClient.post url, {
  list: {
    name: "Test List",
    description: "This is a test location created by API",
    code: "LIST_TEST",
    list_type: "standard"
  }
}.to_json, {
 :Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/metadata_types"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
  -X POST
  -d '{
    "list": {
      "name": "Test List",
      "description": "This is a test location created by API",
      "code": "LIST_TEST",
      "list_type": "standard"
    }
  }'
```

> When successfull the above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
    "id": 123456,
    "name": "Test List",
    "description": "This is a test location created by API",
    "code": "LIST_TEST",
    "list_type": "standard"
}

```

This endpoint creates a empty list

### HTTP Request

`POST https://www.mydatascope.com/api/external/metadata_types`

### Input Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the element of the list
description | String | Description of the element of the list
code | String | Internal code of the element of the list
list_type | String | Valid values: ("standard", "percent", "price")

### Return Codes:

Code  | Description
----  | -----------
201   | Successfull
403   | Forbidden
422   | Wrong parameters, check documentation

<aside class="success">
Remember — use your own Authorization header
</aside>


## Update a List

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/metadata_types/123456'
response = RestClient.post url, {
  list: {
    name: "Test List",
    description: "This is a test location created by API",
    code: "LIST_TEST",
    list_type: "standard"
  }
}.to_json, {
 :Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/metadata_types/123456"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
  -X POST
  -d '{
    "list": {
      "name": "Test List",
      "description": "This is a test location created by API",
      "code": "LIST_TEST",
      "list_type": "standard"
    }
  }'
```

> When successfull the above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
    "id": 123456,
    "name": "Test List",
    "description": "This is a test location created by API",
    "code": "LIST_TEST",
    "list_type": "standard"
}

```

This endpoint updates a list

### HTTP Request

`POST https://www.mydatascope.com/api/external/metadata_types/{id}`

### Input Parameter

Parameter | Type | Description
--------- | ------- | -----------
name | String | Name of the element of the list
description | String | Description of the element of the list
code | String | Internal code of the element of the list
list_type | String | Valid values: ("standard", "percent", "price")

### Return Codes:

Code  | Description
----  | -----------
201   | Successfull
403   | Forbidden
422   | Wrong parameters, check documentation

<aside class="success">
Remember — use your own Authorization header
</aside>


## Bulk Update List Elements

This endpoint allows bulk updating of metadata list objects, with soft deletion of objects not in the incoming list. If an object with an existing code is provided, it will be updated. If a new code is used, the object will be created.

> **Warning**: This operation will delete all existing objects for the specified metadata_type and replace them with the new objects provided. This endpoint is currently in an **experimental stage**. Changes may be made to functionality or structure as we continue testing and refining its implementation.

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/metadata_objects/bulk_update'
response = RestClient.post url, {
  metadata_type: "your_list_code",
  name: "Safety Equipment List",
  list_objects: [
    {
      code: "PPE001",
      name: "Helmet",
      description: "Safety helmet in good condition",
      attribute1: "Mandatory",
      attribute2: "Daily check"
    },
    {
      code: "PPE002",
      name: "Boots",
      description: "Steel-toed safety boots",
      attribute1: "Mandatory",
      attribute2: "Check for wear"
    },
    {
      code: "PPE003",
      name: "Glasses",
      description: "Safety glasses with side shields",
      attribute1: "Mandatory",
      attribute2: "Clean daily"
    }
  ]
}.to_json, {
 :Authorization => '<YOUR_API_TOKEN>',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/metadata_objects/bulk_update"
  -H "Authorization: <YOUR_API_TOKEN>"
  -X POST
  -d '{
    "metadata_type": "your_list_code",
    "name": "Safety Equipment List",
    "list_objects": [
      {
        "code": "PPE001",
        "name": "Helmet",
        "description": "Safety helmet in good condition",
        "attribute1": "Mandatory",
        "attribute2": "Daily check"
      },
      {
        "code": "PPE002",
        "name": "Boots",
        "description": "Steel-toed safety boots",
        "attribute1": "Mandatory",
        "attribute2": "Check for wear"
      },
      {
        "code": "PPE003",
        "name": "Glasses",
        "description": "Safety glasses with side shields",
        "attribute1": "Mandatory",
        "attribute2": "Clean daily"
      }
    ]
  }'
```

> When successful, the above command returns JSON structured like this:

```json
{
  "id": 1,
  "name": "Safety Equipment List",
  "description": "List for safety equipment",
  "code": "your_list_code",
  "list_type": "standard",
  "length": 3
}
```

### HTTP Request

`POST https://www.mydatascope.com/api/external/metadata_objects/bulk_update`

### Input Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| metadata_type | String | Internal code to identify the list (must not be "locations") |
| name | String | Name of the list to be created or updated |
| list_objects | Array | Array of objects to be created or updated |

### List Object Structure

Each object in the `list_objects` array should have the following structure:

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| code | String | Internal code of the list element |
| name | String | Name of the list element |
| description | String | Description of the list element |
| attribute1 | String | Custom attribute of the list element |
| attribute2 | String | Custom attribute of the list element |

### Return Codes

| Code | Description |
| ---- | ----------- |
| 200 | Successful |
| 400 | Bad Request if metadata_type is `locations` |
| 403 | Forbidden |
| 422 | Wrong parameters, check documentation |
| 500 | Internal Server Error |

### Response

The response includes:

| Field | Type | Description |
| ----- | ---- | ----------- |
| id | Integer | ID of the updated list |
| name | String | Name of the updated list |
| description | String | Description of the updated list |
| code | String | Code for the updated list |
| list_type | String | Type of the list |
| length | Integer | Number of active list objects |

<aside class="success">
Remember — use your own Authorization header
</aside>

# Task Assigns

## Create Task Assign

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/assign_task'
response = RestClient.post url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => {}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/assign_task"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
    "form_id":432432,
    "user_id": "user1@email.com",
    "date":"2021-05-10 15:00",
    "l_code":"l25",
    "task_instruction":"",
    "gap":"5"
   }
]

```


### HTTP Request

`POST https://www.mydatascope.com/api/external/assign_task`

### Query Parameters

Parameter | Type | Description
--------- | ------- | -----------
form_id | Integer | Internal identifier of the Form. ID in the URL https://www.mydatascope.com/task_forms/[ID]/edit
user_id | String | User Email
date | Datetime | Date of the assigned Task (YYY-mm-dd HH:MM)
l_code | String | Code of the Location
location_name | String | Name of the location. Only needed if need to create or update
location_address | String | Address of the Location
l_phone | String | Phone of the Location
l_email | String | Email of the Location
c_name | String | Company Name of the location
c_code | String | Company tax id of the location
latitude | String | Latitude of the location
longitude | String | Longitude of the location
task_instruction | String | Instruction of the task
gap | Integer | Hours to perform task
code | String | Code to identify the task

<aside class="success">
Remember — use your own Authorization header
</aside>

## Get Task Assign by ID

```shell
curl "https://www.mydatascope.com/api/external/task_assigns/4821" \
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/task_assigns/4821'
response = RestClient.get url, {
  :Authorization => 'b1cd93mfls9fdmfkadn23'
}
JSON.parse(response)
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
  "id": 4821,
  "assign_id": "TA-2024-001",
  "response_code": "FA-0099",
  "priority": 1,
  "start_time": "2025-03-10 09:00:00",
  "form_name": "Safety Inspection",
  "user_email": "inspector@company.com",
  "description": "Monthly fire extinguisher check",
  "location_name": "Main Warehouse",
  "location_code": "WH-001",
  "location_type": "Location",
  "location_address": "123 Main Ave",
  "location_email": "warehouse@company.com",
  "location_phone": "+1-555-0100",
  "gap": 2,
  "checklist": "Check extinguisher,Verify seal,Sign log",
  "location_latitude": -33.4489,
  "location_longitude": -70.6693,
  "completed": "Yes",
  "on_time": "Yes",
  "delay_time": "0d/00h/00m",
  "completed_datetime": "2025-03-10 10:45:00",
  "late_response_allowed": false,
  "mandatory": "for_everyone",
  "confirmation_status": "completed",
  "status": "completed",
  "time_to_perform_minutes": 105.5,
  "response_start": "2025-03-10 09:05:00",
  "response_end": "2025-03-10 10:45:00",
  "created_at": "2025-03-01 08:00:00",
  "created_by": "Admin User"
}
```

This endpoint retrieves the full detail of a single task assignment by its internal ID. Returns the same fields as the list endpoint. Returns 404 if the assignment does not belong to the authenticated account.

### HTTP Request

`GET https://www.mydatascope.com/api/external/task_assigns/:id`

### Path Parameters

Parameter | Type | Description
--------- | ------- | -----------
id | Integer | Internal database ID of the task assignment

### Response Fields

Field | Type | Description
--------- | ------- | -----------
id | Integer | Internal database ID — unique across all periods
assign_id | String | User-defined task code — may repeat across periods
response_code | String | Code of the submitted form answer, if completed
priority | Integer | Task priority level (set when creating the task)
start_time | String | Scheduled start time (account timezone)
form_name | String | Name of the associated form
user_email | String | Email of the assigned inspector
description | String | Task description or instructions
location_name | String | Name of the location
location_code | String | Code of the location
location_type | String | `Location` or `NestableLocation`
location_address | String | Address of the location
location_email | String | Email of the location
location_phone | String | Phone of the location
gap | Integer | Estimated hours to perform the task
checklist | String | Comma-separated checklist items
location_latitude | Float | Latitude of the location
location_longitude | Float | Longitude of the location
completed | String | `Yes` if a form answer exists, `No` otherwise
on_time | String | `Yes` if completed before deadline, `No` if late, `null` if not completed
delay_time | String | Delay formatted as `Xd/HHh/MMm`. `null` if not completed
completed_datetime | String | When the last answer was submitted (account timezone)
late_response_allowed | Boolean | Whether late submissions are permitted
mandatory | String | `for_nobody`, `for_one`, or `for_everyone`
confirmation_status | String | Acceptance state: `required`, `accepted`, `rejected`, `completed`, or `null`
status | String | Task state: `completed`, `incomplete`, `assigned`, `accepted`, or `rejected`
time_to_perform_minutes | Float | Minutes between first and last answer submission. `null` if not completed
response_start | String | When the first answer was submitted (account timezone)
response_end | String | When the last answer was submitted (account timezone)
created_at | String | When the task assignment was created (account timezone)
created_by | String | Full name of the user who created the task assignment

<aside class="success">
Remember — use your own Authorization header
</aside>

## Get Task Assigns by Period

```shell
curl "https://www.mydatascope.com/api/external/task_assigns?start=2025-03-01&end=2025-03-31" \
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/task_assigns'
response = RestClient.get url, {
  :Authorization => 'b1cd93mfls9fdmfkadn23',
  :params => { start: '2025-03-01', end: '2025-03-31' }
}
JSON.parse(response)
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
  "task_assigns": [
    {
      "id": 4821,
      "assign_id": "TA-2024-001",
      "response_code": "FA-0099",
  "priority": 1,
      "start_time": "2025-03-10 09:00:00",
      "form_name": "Safety Inspection",
      "user_email": "inspector@company.com",
      "description": "Monthly fire extinguisher check",
      "location_name": "Main Warehouse",
      "location_code": "WH-001",
      "location_type": "Location",
      "location_address": "123 Main Ave",
      "location_email": "warehouse@company.com",
      "location_phone": "+1-555-0100",
      "gap": 2,
      "checklist": "Check extinguisher,Verify seal,Sign log",
      "location_latitude": -33.4489,
      "location_longitude": -70.6693,
      "completed": "Yes",
      "on_time": "Yes",
      "delay_time": "0d/00h/00m",
      "completed_datetime": "2025-03-10 10:45:00",
      "late_response_allowed": false,
      "mandatory": "for_everyone",
      "confirmation_status": "completed",
      "status": "completed",
      "time_to_perform_minutes": 105.5,
      "response_start": "2025-03-10 09:05:00",
      "response_end": "2025-03-10 10:45:00",
      "created_at": "2025-03-01 08:00:00",
      "created_by": "Admin User"
    }
  ],
  "total": 1,
  "limit": 100,
  "offset": 0
}
```

This endpoint retrieves a paginated list of task assignments for the authenticated account. The response fields match the platform's Excel export plus the internal `id`, enabling automated integrations without manual downloads. The `total` field reflects the number of task assignments matching the applied filters — without filters, it returns the count of all historical tasks in the account.

### HTTP Request

`GET https://www.mydatascope.com/api/external/task_assigns`

### Input Parameters

Parameter | Type | Description
--------- | ------- | -----------
start | String | Optional. Start date in `YYYY-MM-DD` format. Filters by `start_time >= date`
end | String | Optional. End date in `YYYY-MM-DD` format. Filters by `start_time <= date`
status | String | Optional. Filter by status: `completed`, `incomplete`, `assigned`, `accepted`, `rejected`
location_id | Integer | Optional. Filter by location ID
user_email | String | Optional. Filter by the assigned inspector's email
limit | Integer | Optional. Max number of results to return. Default: 100, max: 300
offset | Integer | Optional. Number of results to skip (for pagination). Default: 0

### Response Fields

Field | Type | Description
--------- | ------- | -----------
id | Integer | Internal database ID — unique across all periods
assign_id | String | User-defined task code — may repeat across periods
response_code | String | Code of the submitted form answer, if completed
priority | Integer | Task priority level (set when creating the task)
start_time | String | Scheduled start time (account timezone)
form_name | String | Name of the associated form
user_email | String | Email of the assigned inspector
description | String | Task description or instructions
location_name | String | Name of the location
location_code | String | Code of the location
location_type | String | `Location` or `NestableLocation`
location_address | String | Address of the location
location_email | String | Email of the location
location_phone | String | Phone of the location
gap | Integer | Estimated hours to perform the task
checklist | String | Comma-separated checklist items
location_latitude | Float | Latitude of the location
location_longitude | Float | Longitude of the location
completed | String | `Yes` if a form answer exists, `No` otherwise
on_time | String | `Yes` if completed before deadline, `No` if late, `null` if not completed
delay_time | String | Delay formatted as `Xd/HHh/MMm`. `null` if not completed
completed_datetime | String | When the last answer was submitted (account timezone)
late_response_allowed | Boolean | Whether late submissions are permitted
mandatory | String | `for_nobody`, `for_one`, or `for_everyone`
confirmation_status | String | Acceptance state: `required`, `accepted`, `rejected`, `completed`, or `null`
status | String | Task state: `completed`, `incomplete`, `assigned`, `accepted`, or `rejected`
time_to_perform_minutes | Float | Minutes between first and last answer submission. `null` if not completed
response_start | String | When the first answer was submitted (account timezone)
response_end | String | When the last answer was submitted (account timezone)
created_at | String | When the task assignment was created (account timezone)
created_by | String | Full name of the user who created the task assignment

<aside class="notice">
When no `start_date` / `end_date` filters are provided, this endpoint returns <strong>all historical task assignments</strong> for the account. The platform UI applies a default ±7-day window, so counts may differ. Use date filters to match the UI view.
</aside>

<aside class="success">
Remember — use your own Authorization header
</aside>

# Notifications

## List Last notifications

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/notifications'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => { start: '10/12/2019', end: '30/12/2019'}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/notifications"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
      "id":2345,
      "type":"PDF",
      "url":"https://www.mydatascope.com/pdf_url_example",
      "form_name":"Form Name",
      "form_code":"25",
      "user":"user@email.com",
      "created_at":"2019-12-03T17:48:47.000-02:00"
   }]


```

This endpoint retrieves all list items of a specific list

### HTTP Request

`GET https://www.mydatascope.com/api/external/notifications`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
start | last 7 days | set the start date range
end | today | set the end date range

### Output Parameter

Parameter | Type | Description
--------- | ------- | -----------
id | String | Identifier of the notification
type | String | Type of notification: PDF or Excel
url | String | URL of the notified file
form_name | String | Name of the form
form_code | String | Code of the form
user | String | Name of the user
created_at | Datetime | Date when the list element was created
updated_at | Datetime | Date when the list element was updated

# Files

## List Last generated files

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/files'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => { start: '10/12/2019', end: '30/12/2019'}
}
JSON.parse(response)
```

```shell
curl "https://www.mydatascope.com/api/external/files"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[  
   {  
      "id":2345,
      "type":"PDF",
      "url":"https://www.mydatascope.com/pdf_url_example",
      "form_name":"Form Name",
      "form_code":"25",
      "user":"user@email.com"
   }]


```

This endpoint retrieves all list items of a specific list

### HTTP Request

`GET https://www.mydatascope.com/api/external/files`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
start | last 7 days | set the start date range
end | today | set the end date range

### Output Parameter

Parameter | Type | Description
--------- | ------- | -----------
id | String | Identifier of the notification
type | String | Type of notification: PDF or Excel
url | String | URL of the notified file
form_name | String | Name of the form
form_code | String | Code of the form
user | String | Name of the user

<aside class="success">
Remember — use your own Authorization header
</aside>

# Webhooks

Sometimes people call webhooks reverse APIs, but perhaps more accurately a webhook lets you skip a step. With most APIs there’s a request followed by a response. No request is required for a webhook, it just sends the data when it’s available.

To use a webhook, you register a URL with the company providing the service. That URL is a place within your application that will accept the data and do something with it. In some cases, you can tell the provider the situations when you’d like to receive data. Whenever there’s something new, the webhook will send it to your URL.

DataScope Webhook notifications are sent in an HTTP POST request, and their contents (containing the response data) are in JSON format.

## Configuration

To configure the webhook you need to go to the Integrations section and then Webhooks and click on [New Webhook](https://app.mydatascope.com/integrations).

![New Webhook](https://data-scope.s3-us-west-2.amazonaws.com/images/other/Captura+de+pantalla+2020-09-08+a+la(s)+13.23.28.png "New Webhook")

## Output

> The webhook will return a JSON with this structure:

```json
[{
  "form_name": "[Form Name] (String)",
  "code": "[Code Form ID] (String)",
  "latitude": "[latitude] (Float)",
  "longitude": "[longitude] (Float)",
  "[question_name][question_id]": {
    "name": "[Question name] (String)",
    "label": "[Section Name] (String)",
    "row" "[Nº repeatable field] (Integer) Default: null",
    "value": "[Value of the answer] (String)",
    "type": "[Type of question]*",
    "id": "[Internal ID of the question] (Integer)"
  }
}]
```

<aside class="notice">
TIP: It will only start sending information for the new forms done after the integration.
</aside>

<aside class="notice">
That tip is the main reason a webhook alone is not enough for a warehouse: it cannot replay what it never sent. <a href="#choosing-a-method">Choosing a method</a> compares webhooks against the pull-based options.
</aside>

# Tickets (FKA Issues)

## Get Tickets by Period

```shell
curl "https://www.mydatascope.com/api/external/findings/list?start=01-01-2026&end=31-01-2026"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/findings/list'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => { start: '01-01-2026', end: '31-01-2026' }
}
JSON.parse(response)
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
[
  {
    "id": "Krdz3aFoWZ4ZVgpuAart",
    "code": 1,
    "name": "ejemplo",
    "description": "Ejemplo",
    "type": "Safety",
    "status": "closed",
    "priority": "high",
    "creation_date": "30/01/2026 19:53",
    "expiration_date": "23/01/2026 19:53",
    "closure_date": "05/02/2026 15:23",
    "closure_message": "Addressed",
    "location_id": 42,
    "location_name": "Main Office",
    "location_code": "LOC-001",
    "asset_name": null,
    "asset_identifier": null,
    "creator_id": 7,
    "creator_email": "juan.perez@example.com",
    "creator_name": "Juan Perez",
    "assignees_concatenated": "user_id:7;;email:juan.perez@example.com;;name:Juan Perez&&user_id:12;;email:maria.lopez@example.com;;name:Maria Lopez",
    "invitees_concatenated": "user_id:18;;email:carlos.silva@example.com;;name:Carlos Silva",
    "assignees": {
      "0": { "id": 7, "full_name": "Juan Perez", "email": "juan.perez@example.com" },
      "1": { "id": 12, "full_name": "Maria Lopez", "email": "maria.lopez@example.com" }
    },
    "invitees": {
      "0": { "id": 18, "full_name": "Carlos Silva", "email": "carlos.silva@example.com" }
    },
    "last_updated_by": "Juan Perez",
    "form_answer_id": 101,
    "form_answer_code": "FA-12345",
    "task_form_title": "Daily Inspection",
    "task_form_question": "What issues were found?"
  }
]
```

This endpoint retrieves a list of Tickets filtered by creation date period.

### HTTP Request

`GET https://www.mydatascope.com/api/external/findings/list`

### Input Parameters

Parameter | Type | Description
--------- | ------- | -----------
start | String | Optional. Start date in `dd-mm-yyyy` format. Defaults to 7 days ago
end | String | Optional. End date in `dd-mm-yyyy` format. Defaults to today
status | String | Optional. Filter by status: `open`, `in_progress`, `paused`, `closed`
task_form_id | Integer | Optional. Filter by the ID of the associated form. Only returns tickets linked to that form
limit | Integer | Optional. Max number of results to return. Default: 200, max: 200
offset | Integer | Optional. Number of results to skip (for pagination). Default: 0, max: 2000

The maximum allowed date range is **90 days**. Requests with a wider range will return `422 Unprocessable Entity`.

### Pagination

Results are paginated using `limit` and `offset`. `limit` controls how many results are returned, and `offset` controls how many to skip from the beginning.

For example, if there are 400 tickets in the period:

- First page: `limit=200&offset=0` → returns tickets 1–200
- Second page: `limit=200&offset=200` → returns tickets 201–400

### Response Fields

Field | Type | Description
--------- | ------- | -----------
id | String | Firestore document ID
code | Integer | Sequential ticket number within the account
name | String | Ticket name
description | String | Ticket description
type | String | Resolved ticket type name (null if no type assigned)
status | String | Current status: `open`, `in_progress`, `paused`, `closed`
priority | String | Priority level: `low`, `medium`, `high`, `critical`
creation_date | String | Date and time the ticket was created, formatted according to account preferences (e.g. `30/01/2026 19:53`)
expiration_date | String | Date and time the ticket expires, formatted according to account preferences (null if none)
closure_date | String | Date and time the ticket was closed, formatted according to account preferences (null if not closed)
closure_message | String | Message provided when closing the ticket (null if not closed)
location_id | Integer | Database ID of the associated location (null if none)
location_name | String | Name of the associated location (null if none)
location_code | String | Code of the associated location (null if none)
asset_name | String | Name of the linked asset (null if none)
asset_identifier | String | Code/identifier of the linked asset (null if none)
creator_id | Integer | Database ID of the user who created the ticket
creator_email | String | Email of the user who created the ticket
creator_name | String | Full name of the user who created the ticket
assignees_concatenated | String | Assigned users as a `&&`-separated string, each entry formatted as `user_id:{id};;email:{email};;name:{name}` (empty string if none)
invitees_concatenated | String | Invited users as a `&&`-separated string, same format as `assignees_concatenated` (empty string if none)
assignees | Object | Assigned users as an indexed object: `{ "0": { "id": Integer, "full_name": String, "email": String }, ... }` (empty object if none)
invitees | Object | Invited users as an indexed object, same shape as `assignees` (empty object if none)
last_updated_by | String | Full name of the user who last updated the ticket (null if not available)
form_answer_id | Integer | Database ID of the linked form answer (null if none)
form_answer_code | String | Code of the linked form answer (null if none)
task_form_title | String | Title of the linked form (null if no form answer linked)
task_form_question | String | Question from the linked form answer (null if no form answer linked)

### Return Codes

```
200: OK
401: Unauthorized
422: Unprocessable Entity (invalid date range or range exceeds 90 days)
```

<aside class="success">
Remember — use your own Authorization header
</aside>


## Get Ticket

```shell
curl "https://www.mydatascope.com/api/external/findings/get/Krdz3aFoWZ4ZVgpuAart"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/findings/get/Krdz3aFoWZ4ZVgpuAart'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
}
JSON.parse(response)
```

> The above command returns JSON structured like this, you can check the description of each parameter below:

```json
{
  "id": "Krdz3aFoWZ4ZVgpuAart",
  "code": 1,
  "name": "ejemplo",
  "description": "Ejemplo",
  "type": null,
  "status": "closed",
  "priority": "high",
  "creation_date": "2026-01-30T19:53:52.528+00:00",
  "expiration_date": "2026-01-23T19:53:00.000+00:00",
  "closure_date": "2026-02-05T15:23:12.360+00:00",
  "closure_message": "Addressed",
  "location_name": "Main Office",
  "creator_name": "Juan Perez",
  "assignees": "Juan Perez, Maria Lopez",
  "invitees": "Carlos Silva",
  "last_updated_by": "Juan Perez",
  "form_answer_id": 12345,
  "task_form_title": "Daily Inspection",
  "task_form_question": "What issues were found?"
}
```

This endpoint retrieves a single Ticket by its Firestore document ID.

### HTTP Request

`GET https://www.mydatascope.com/api/external/findings/get/:id`

### URL Parameter

Parameter | Type | Description
--------- | ------- | -----------
id | String | Required. The Firestore document ID of the ticket. This can be obtained from the ticket URL in the DataScope web app (`?selected=<id>`)

### Response Fields

Field | Type | Description
--------- | ------- | -----------
id | String | Firestore document ID
code | Integer | Sequential ticket number within the account
name | String | Ticket name
description | String | Ticket description
type | String | Ticket Type ID (null if no type assigned)
status | String | Current status: `open`, `in_progress`, `paused`, `closed`
priority | String | Priority level: `low`, `medium`, `high`, `critical`
creation_date | Datetime | Date and time the ticket was created (ISO 8601)
expiration_date | Datetime | Date and time the ticket expires (ISO 8601)
closure_date | Datetime | Date and time the ticket was closed (null if not closed)
closure_message | String | Message provided when closing the ticket (null if not closed)
location_name | String | Name of the associated location (null if none)
creator_name | String | Full name of the user who created the ticket
assignees | String | Comma-separated list of assigned users' full names
invitees | String | Comma-separated list of invited users' full names (empty string if none)
last_updated_by | String | Full name of the user who last updated the ticket (null if not available)
form_answer_id | Integer | ID of the linked form answer (null if none)
task_form_title | String | Title of the linked form (null if no form answer linked)
task_form_question | String | Question from the linked form answer (null if no form answer linked)

### Return Codes

```
200: OK
404: Not Found
403: Forbidden
```

<aside class="success">
Remember — use your own Authorization header
</aside>

## Get last 5 tickets
```shell
curl "https://www.mydatascope.com/api/external/last_findings"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```
```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/last_findings'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
}
JSON.parse(response)
```

### HTTP Request

`GET https://www.mydatascope.com/api/external/last_findings`

### Return Codes
```
200: OK
403: Forbidden
```

<aside class="success">
Remember — use your own Authorization header
</aside>

## Get Ticket Types
```shell
curl "https://www.mydatascope.com/api/external/findings/types"
  -H "Authorization: b1cd93mfls9fdmfkadn23"
```

```ruby
require 'rest-client'
require 'json'

url = 'https://www.mydatascope.com/api/external/findings/types'
response = RestClient.get url, {
:Authorization => 'b1cd93mfls9fdmfkadn23',
 :params => { search: 'Finding T'}
}
JSON.parse(response)
```
> The above command returns JSON structured like this, you can check the description of each parameter below:
```json
[  
   {  
      "id":"3rRET34tg3f27g6hHfsE",
      "name":"Finding Type 1",
   },
   {
     "id": "fe23fd32x43Dsa3D6y9T",
     "name":"Finding Type 2",
   }
]
```

### HTTP Request

`GET https://www.mydatascope.com/api/external/findings/types`

### Input Parameter
Parameter | Type | Description
--------- | ------- | -----------
search | String | Optional. Name to search

### Return Codes
```
200: OK
403: Forbidden
```

<aside class="success">
Remember — use your own Authorization header
</aside>

## Create Ticket
This endpoint create a ticket

### HTTP Request

`POST https://www.mydatascope.com/api/external/findings/create`

### Input Parameter

Parameter | Type | Description
--------- | ------- | -----------
author_email | String | Ticket's author email. Fallback: Owner of the API Key
name | String | Required. Name of the Ticket
description | String | Required. Description of the location
expiration_date | String or Datetime | Required. Date or String with the date to expire the ticket (YYYY-MM-DD HH:MM)
status | String | Status of ticket. Fallback: `open`. Options: `[open, in_progress, paused, closed]`
priority | String | Status of ticket. Fallback: `medium`. Options: `[low, medium, high, critical]`
location_id | Integer | Optional. ID of the location
location_name | Integer | Optional. Name of the location in case of unrecognized `location_id`
assignees | Array of Integers or Strings | Required. MobileUser IDs or emails of users to be assigned to the ticket
invitees | Array of Integers or Strings | Optional. MobileUser IDs or emails of users to be invited to the ticket (ignoring already assigned ones)
type | String | Optional. Ticket Type ID or Ticket Type Name. If provided, preset data will be retrieved from this type. Fallback: "Other" (no type assigned).
creation_date | String or Datetime | Optional. Date or String with a custom creation date of ticket (YYYY-MM-DD HH:MM)

### Return Codes:

```
201: Created
403: Forbidden
422: Unprocessable Entity
```

<aside class="success">
Remember — use your own Authorization header
</aside>

# Changelog

**07-Aug-2026**

- Added [Get Signature Requests](#get-signature-requests) and its [Airbyte Cloud connector](#signatures-connector), with streams for signature requests and for individual signers
- Grouped everything about getting data out into a new [Data Export](#data-export) section near the top, including the [comparison between webhooks and the pull-based options](#choosing-a-method) that used to sit inside Webhooks
- Split the Airbyte manifest in two, one per source. The answers manifest keeps its URL, so an existing connector is unaffected

**04-Aug-2026**

- Documented three Ticket endpoints that were missing from this reference: last tickets, ticket types and ticket creation
- Updated the developer portal links to https://app.mydatascope.com/integrations
- Moved this changelog into its own section

**30-Jul-2026**

- Added [Answers V5 (Beta)](#get-all-answers-v5-beta), its `custom_fields`, and the [Airbyte Cloud connector manifest](#answers-connector)

**Apr-2026**

- Added the Ticket endpoints for fetching by period and by id
- Added the Task Assign endpoints for fetching by period and by id
- Added CRUD for Lists and List Elements, including bulk update

**Nov-2025**

- Added the first Ticket endpoints
