---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby

toc_footers:
  - <a href='https://www.mydatascope.com/webhooks'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the DataScope API! You can use our API to access DataScope API endpoints, which can get information collected from the DataScope platform and App.

We have language bindings in Shell and Ruby! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.




# Authentication



DataScope uses API keys to allow access to the API. You can register a new DataScope API key at our [developer portal](https://www.mydatascope.com/webhooks).

![alt text](https://i.imgur.com/M4awUbe.jpg "Logo Title Text 1")

DataScope expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: b1cd93mfls9fdmfkadn23`

<aside class="notice">
You must replace <code>b1cd93mfls9fdmfkadn23</code> with your personal API key.
</aside>

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

`GET https://www.mydatascope.com/api/external/change_form_answer`

### Query Parameters

Parameter |  Description
--------- | -----------
form_name | name of ID of the form
code | blank | Code of the response
question_name | Name of the question to change
question_value | Value of the question to change


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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
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

# Webhooks

Sometimes people call webhooks reverse APIs, but perhaps more accurately a webhook lets you skip a step. With most APIs there’s a request followed by a response. No request is required for a webhook, it just sends the data when it’s available.

To use a webhook, you register a URL with the company providing the service. That URL is a place within your application that will accept the data and do something with it. In some cases, you can tell the provider the situations when you’d like to receive data. Whenever there’s something new, the webhook will send it to your URL.

DataScope Webhook notifications are sent in an HTTP POST request, and their contents (containing the response data) are in JSON format.

## Configuration

To configure the webhook you need to go to the Integrations section and then Webhooks and click on [New Webhook](https://mydatascope.com/webhooks/new).

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
