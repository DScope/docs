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

`POST https://www.mydatascope.com/api/external/change_form_answer`

### Query Parameters

Parameter |  Description
--------- | -----------
form_name | name of ID of the form
form_code | Code of the response
question_name | Name of the question to change
question_value | Value of the question to change
subform_index | Number to specify the subform index (Starting from 1). Leave blank if question it's not inside subform*.


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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
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
Remember — user your own header Authorization
</aside>

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
Remember — use your own header Authorization
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
Remember — use your own header Authorization
</aside>
