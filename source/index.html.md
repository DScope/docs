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
   }
]

```

This endpoint retrieves last answers

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
  question_id | Integer | Internal identifier of the question.
  form_code | String | Public identifier of the form answer.
  form_state | String | Last status of the form answer.
  form_id | integer | Internal code of the form, fixed to all answers of that form.
  form_answer_id | Integer | Internal code of the form answer.
  form_name | String | Name of the form.
  user_name | String | Name of the user.
  created_at | Date | When the form was received.
  latitude | Float | Latitude where the form was answered.
  longitude | Floar | Longitude where the form was answered.


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

This endpoint retrieves last answers

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


