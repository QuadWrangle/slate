---
title: QuadWrangle API Reference

language_tabs:
  - javascript

toc_footers:
  - <a href="#">TOC</a>
  
includes:
  - errors
  
search: true
---

# Introduction

Welcome to the QuadWrangle Data API. This API system is used to load data into the QuadWrangle platform and to pull data out. 

This API documentation is a living document as we continue to build out and improve this API system we will make edits to this documentation. 

All calls to this API require an API Key and a Client ID which can be requested by emailing [support-request@quadwrangle.com](mailto:support-request@quadwrangle.com) with the subject: **API Key Request** 

QuadWrangle provides two primary hosts, one being the production host where you will want to your live data to be written to and read production data. 

The other is our Sandbox environment which can be used to test your code without having any effect on your production environment. 

Production Host: `https://api.qwd.io/`

Sandbox Host: `https://sandbox.qwd.io/`


# Person

## Bulk Load

Bulk load multiple people records into the system. 


### HTTP Request

`POST /people/load?key=<API KEY>&appClientId=<CLIENT ID>`

In the body of your request, you will create a List of `Person` objects to be loaded:

```json
[
	{ <person object> },
	{ <person object> },
	//...
]
```

You may pass in the same payload and the system will determine if a new record needs to be created or simply just update said record. 

We cannot guarantee that field updates from this request will be updated based on the existence of a Change Set for a given field. A Change Set is generated when the actual end-user makes a change to their data via QuadWeb, QuadMail or QuadMobile. If they changed any of their biographical information, a Change Set is created and will prevent any 3rd party system from updating that data (thus overwriting an end-user's explicitly modified data). 

The following fields may have Change Sets "guarding" their being updated:

* First Name
* Last Name
* Primary email
* Maiden Name
* Job Title
* Employer
* Primary address
* Primary address 2
* Primary city
* Primary state
* Primary country
* Primary postal code
* Primary phone number


### HTTP Response

**Error Codes**

| Code | Reason |
|-----|-----|
| 400 | Bad Request, body of the request could have been empty, or the payload did not have any entries |
| 500 | Unexpected Error, could have been caused by a problem processing the payload, logged on our end |

You will receive a bulk load status back which will contain any errors and what those errors were. 

```json
{
	"errors" : [
		{ 
			"recordId" : "<record ID>",
			// ... other person fields,
			"reason": "<error string>"
		},
		// ...
	],
	"successCount" : <int>,
	"errorCount" : <int>,
	"time" : <timestamp>,
	"status" : "<status string>"
}
```

| Field  | Description  |
|---------------|----------------|
| `errors`    |  A list of person records that contained errors. Each person object will include a `reason` field that describes what went wrong   |
| `successCount`    |   Count of successful records loaded   |
| `errorCount` | Count of the number of records that had errors |
| `time` | The time in milliseconds this job ran |
| `status` | A status string for this bulk load request |

It is important to note that the actual saving of a bulk load request happens asynchronously and not part of the HTTP request thread, so after initiating a bulk load request, it may take several minutes for your entire load to be written to the database. 

## Insert a Single Record

`POST /people?key=<API KEY>&appClientId=<CLIENT ID>`

Insert a single person record, the request payload does not depend on the existence of an ID, but will check the existence of this record in the database by the email so as to not insert duplicate entries. 

### HTTP Request

```json
{
	"recordId" : "<recordId>",
	// ... rest of the person fields defined below
}
```

### HTTP Response

**Error Codes**

| Code | Reason |
|------|-------|
| 400 | Bad Request, the body of the request was blank, or there was an error with the person object being submitted, which will be included in this response |


You will receive a response in JSON that looks like:

```json
{
	"time" : <int>,
	"id" : "<system ID>",
	"recordId" : "<original record ID>",
	"person" : {
		// person fields for this record
	}
}
```

| Field  | Description  |
|---------------|----------------|
| `time`    |   the time in milliseconds to complete this request   |
| `id`   |   system ID for this record   |
| `recordId` | the original record ID for this person |
| `person` | a copy of the original person object for this request |
 

## Update a Single Record

`PUT /people?key=<API KEY>&appClientId=<CLIENT ID>`

Update (Save) an individual record. Your payload must contain either a system ID (using the `id` field on the Person object) or a record ID (using the `recordId` on the Person object -- detailed below). 

### HTTP Request

```json
{
	"recordId" : "<recordId>",
	"id" : "<system id>",
	// ... person fields
}
```

You may use either your original `recordId` or use the `id` (system ID). in your payload so that the system an properly look up the record you are wishing to update. 

### HTTP Response

**Error Codes**

| Code | Reason |
|------|--------|
| 400 | Bad Request, the body of the request may have been blank or there was an error with this person object. The error will be included in this response. Or, no ID was provided so no look up could occur. |
| 404 | This person record could not be found |


You will receive a response in JSON that looks like:

```json
{
	"time" : <int>,
	"id" : "<system ID>",
	"recordId" : "<original record ID>",
	"person" : {
		// person fields for this record
	}
}
```

| Field  | Description  |
|---------------|----------------|
| `time`    |   the time in milliseconds to complete this request   |
| `id`   |   system ID for this record   |
| `recordId` | the original record ID for this person |
| `person` | a copy of the original person object for this request |

## Get Changes

Changes are returned as a List of Change Sets. A Change Set represents a change to an individual field for a particular person record. For example, if an end-user logs into your QuadMobile app, and then taps into their profile and updates their first name, a Change Set will be created for the `firstName` field. A Change Set has an "new value" and an "old value" associated with it, along with an "accepted" flag meaning this change has been accepted. 

Using Change Sets is a good way to pull updates down from the QuadWrangle platform safely and enables you to selectively update your own data for this person. 

long start, long end, int limit, boolean accepted, int page

`GET /people/changes?key=<API KEY>&appClientId=<CLIENT ID>&start=<timestamp-in-milliseconds>&end=<timestamp-in-milliseconds>&limit=100&accepted=false&page=0`

### HTTP Request

Fetching Change Sets is based on a `GET` request, and you may provide query string parameters to limit the results. 

**Query String Parameters**

| Parameter  | Description  |
|---------------|----------------|
| `start`    |   start date from which to limit your query. Use an epoch timestamp with milliseconds   |
| `end`    |  end date from which to limit your query. Use an epoch timestamp with milliseconds  |
| `limit` | Limit the number of results returned. Defaults to 100. This enables the ability to "page" through results |
| `accepted` | Only return those results that have been accepted (set to `true`) or not accepted (set to `false`) |
| `page` | Default is 0, pass this to set the cursor for the result set. You will receive a total record count in the response so you can properly set up pagination on your own side |

### HTTP Response

**Error Codes**

| Code | Reason |
|------|-------|
| 500 | An unexpected error occurred fetching change sets. This has been logged on our end |


```json
{
	"recordCount" : <int>,
	"groups" : [
		{
			"userId" : "<string>",
			"changes" : [
				<change set object>,
				<change set object>
			]
		},
		// more change groups
	]
}
```

## Get an Individual's Change Set

`GET /person/changes/<id>?key=<API KEY>&appClientId=<CLIENT ID>`

### HTTP Request

Query String Parameters:

| Parameter | Description |
|-----|-----|
| `id` | the system user ID, or the record ID (your original ID can be used) |

### HTTP Response

**Error Codes**

| Code | Reason |
|-------|-----|
| 400 | Bad Request, missing the ID parameter |
| 404 | Not Found, could not find the user for this request |
| 500 | Unexpected Error, logged on our end usually means an unknown exception was raised while querying the database |

Upon a successful request, you will receive the following JSON:

```json
{
	"count" : <int>,
	"changes" : [
		<change set object>,
		<change set object>
	]
}
```

## Accept Change Sets

You may accept change sets, which provides a time stamp of when the change was accepted and also lets our system create new change sets for a given user/field combination should the end user make later changes to their bio information (prior to accepting a change set any subsequent changes made by the end user update the existing change set "in place.")

`POST /people/acceptChanges?key=<API KEY>&appClientId=<CLIENT ID>`

### HTTP Request

```json
{
	"ids" : [
		<string>,
		<string>
	]
}
```

You may pass in a list of change set IDs to accept. The ID here is the system ID for a particular change set. An example is if you have a change set for user for the `firstName` field, that change set has an ID. You may then include that ID in this list to accept it. 


### HTTP Response

**Error Codes**

| Code | Reason |
|-------|--------|
| 400 | Bad Request, the body of this request may have been blank, or that the list of IDs did not have at least one element |
| 404 | Could not find any change sets for the ID(s) you provided |

`HTTP 200 Ok`

## Search
`POST /people/search?key=<API KEY>&appClientId=<CLIENT ID>`

You may search for constituent records within the QW database by providing a set of filters. This request will then return a list of matches. 

### HTTP Request ###

This endpoint expects JSON in the body of the request.

> Search Filter Model

```json
{
	"classYear" : "<string>",
	"updatedStart" : <long>,
	"updatedEnd" : <long>,
	"email" : "<string>",
	"recordId" : "<string>",
	"name" : "<string>",
	"affiliation" : "<string>",
	"affinity" : "<string>",
	"isActive" : <boolean>,
	"isVerified" : <boolean>,
	"hasFacebook" : <boolean>,
	"hasLinkedin" : <boolean>,
	"hasEmail" : <boolean>,
	"limit" : <int>,
	"offset" : <int>
}
```

Search filter parameters:

| Name | Data Type | Values |
|-------|-------|--------|
| `classYear` | String | if this value can be evaluated as a simple Integer (for example 1996) then this query will use a single class year. If this value contains commas (example: 1996,1997,1998) then the query will use each of these class years. Lastly, if this value contains a dash, we will use a range (example: 1990-1996). You may do "greater than" or "less than": `>2014` (greater than or equal to class year 2014) or: `<2016` (less than or equal to class year 2016) |
| `updatedStart` | Long | Timestamp (in milliseconds) to use as a lower bounds to fetch records by their last updated time stamp |
| `updatedEnd` | Long | Timestamp (in milliseconds) to use as an upper bounds to fetch records by their last updated time stamp |
| `email` | String | Email to search by |
| `recordId` | String | Use the original ID from the database of record |
| `name` | String | Name search. If this value is a single value, then this will be a  last name search, if this value contains spaces, then this will be a first and last name search |
| `affiliation` | String | Search by one of the built in affiliations (ALUM, FACULTY etc.), You may specify a list using a comma separated list: ALUM,FACULTY,... |
| `affinity` | String | Search by one of your custom affinities |
| `isActive` | Boolean | Set to `true` to only return results of active users. `false` will return only those who are not active. Null value means this field is ignored | 
| `isVerified` | Boolean | Set to `true` to only return those constituent records who have been verified (have Directory access). Null value means this field is ignored |
| `hasFacebook` | Boolean | Include person records that have logged into the QW Platform using Facebook. Null value means this field is ignored |
| `hasLinkedin` | Boolean | Include person records that have logged into the QW Platform using LinkedIn. Null values means this field is ignored |
| `hasEmail` | Boolean | Only include records that have an email address. Null values means this field is ignored |
| `limit` | Integer | Limit result set to this many rows per request |
| `offset` | Integer | What "page" of results do you want? |

You do not need to include all of the these fields in your request object, this just helps narrow down your searches. 

### HTTP Response ###

**Error Codes**

| Code | Reason |
|------|--------|
| 400 | Bad Request, request body could have been blank - an error may have occurred validating the search criteria (that error will be included with this response) | 

> Search Result Model

```json
	"results" : [
		<person object>,
		<person object>
	],
	"count" : <int>,
	"page" : <int>
```


# Events

The Events system offers several end points to import event objects into the QW system, and to also report on event activity.

## Bulk Load

`POST /events/load?key=<API KEY>&appClientId=<CLIENT ID>`

You may use this endpoint to bulk load events into the system. 

### HTTP Request

This endpoint is expecting JSON in the body of the request:

```json
[
	<event object>,
	<event object>
]
```

### HTTP Response

**Error Codes**

| Code | Reason |
|--------|--------|
| 400 | Bad Request, the body could have been blank, the JSON could have been malformed |

A successful request will return a JSON response:

```json
{
	"successCount" : <int>,
	"errorCount" : <int>,
	"status" : "<string>",
	"time" : <int>,
	"errors" : [
		<event object>,
		<event object>
	]
}
```

The key field is the `errors` field in that if any event fails validation or cannot be written to the database for any reason, this list will be updated with that event object, and that object will have its `reason` field providing details of the error. 

## Insert a Single Event

`POST /event?key=<API KEY>&appClientId=<CLIENT ID>`

Save an event. 

### HTTP Request

This endpoint expects a JSON in the body of the message that takes the form of the Event model (see below). This request does not expect an `id` to be set and will just do an insert into the database. If an `id` is provided, then an update will be performed for this request. 

### HTTP Response

**Error Codes**

| Code | Reason |
|------|--------|
| 400 | Body of the request was blank, or the JSON was malformed |
| 500 | Errors occurred saving this event, this response will detail any of those errors |

> Save Event Response

```json
	"event" : <event model>,
	"id" : "<string>",
	"time" : <long>
```

| Field | Data Type |
|-----|--------|
| `event` | The event model you saved, this model will include the `id` field |
| `id` | The system ID for this event |
| `time` | The length of time to complete this operation |


## Update a Single Event

`PUT /event?key=<API KEY>&appClientId=<CLIENT ID>`

Save an event

### HTTP Request

This endpoint expects a JSON Event model, and requires an `id` to be set in the model. 

### HTTP Response 

**Error Codes**

| Code | Reason |
|----|---|
| 400 | Bad request, body could have been blank, JSON was malformed, the `id` field was missing or the ID was not a valid ID |
| 404 | Not found, the event was not found |
| 500 | An unknown error saving this event occurred. This response will include any details for this error |

> Save Event Response

```json
{
	"event" : <event model>,
	"id" : "<string>",
	"time" : <long>
}
``` 

| Field | Data Type |
|-----|--------|
| `event` | The event model you saved, this model will include the `id` field |
| `id` | The system ID for this event |
| `time` | The length of time to complete this operation |


## Fetch events

`GET /events/get?start=<long>&end=<long>&page=<int>&limit=<int>&key=<API KEY>&appClientId=<CLIENT ID>`

Fetch events in the system passing in parameters to limit/filter the results

### HTTP Request

**Query String Parameters**

| Parameter | Type | Possible Value |
|------|------|----
| `start` | Long | a time stamp (epoch in milliseconds) to use as the lower bounds. This uses the "start" date of the event to query |
| `end` | Long | a time stamp (epoch in milliseconds) to use as the upper bounds. This uses the "end" date of the event to query |
| `page` | Integer | specify what "page" you are on, for paginating values |
| `limit` | Integer | Limit the result set, defaults to 100 events per "page" |

### HTTP Response

> Get Events Response Model

```json
{
	"count" : <int>,
	"events" : [
		<event model>,
		<event model>
	]
}
```

| Field | Value |
|----|-----|
| `count` | The total count for your query |
| `events ` | List of Event models |

## Fetch Event Detail

` GET /event/<id>?key=<API KEY>&appClientId=<CLIENT ID>`

Fetch the details for a single event, passing in the event's system ID in the URL. 

### HTTP Response

**Error Codes**

| Code | Reason
|----|-----|
| 400 | Bad request, the ID was not provided |
| 404 | Not found, the event was not found for the ID provided |

This request returns an Event model (see below)


## Fetch RSVPs for an Event

`GET /event/rsvps/<eventId>?key=<API KEY>&appClientId=<CLIENT ID>`

Fetch RSVPs for a specific event, as specified by the `eventId` in the URL for this request. 

### HTTP Request

`eventId` may either the the system ID for this event, or the external ID (`extId`) you may have specified (if creating this event was done through this API). 

### HTTP Response 

**Error Codes**

| Code | Reason
|------|-------|
| 400 | Bad request, event ID was not provided in the URL |
| 404 | Not found, event was not found for the ID provided

This request will return a list of RSVP models (see below for details)

## Fetch RSVPs

`GET /events/rsvps?start=<long>&end=<long>&limit=<int>&page=<int>&key=<API KEY>&appClientId=<CLIENT ID>`

Query for a list of RSVPs, regardless of the event (so the results will return RSVPs for a mix of events, but each event is fully included in the payload so that you can understand exactly what event a given RSVP is for)

### HTTP Request

**Query Parameters**

| Parameter | Possible Value |
|------|-------|
| `start` | Start time, lower bounds, (time stamp in milliseconds) using the "created" time for the RSVPs |
| `end` | End time, upper bounds (time stamp in milliseconds) using the "created" time for the RSVPs |
| `limit` | Limit the number of records to return. Defaults to 100 |
| `page` | What page of results to return (used when paging through results. The response will return the full record count so you can properly page through results |

### HTTP Response 

> RSVP Response

```json
{
	"count" : <int>,
	"rsvps" : [
		<rsvp model>,
		<rsvp model>
	]
}
```

| Field | Type |
|-----|------|
| `count` | record count for this query |
| `rsvps` | List of RSVP models |

## Fetch Check-ins for an Event

`GET /event/checkins/<eventId>?key=<API KEY>&appClientId=<CLIENT ID>`

Fetch Check-ins for a specific event, as specified by the `eventId` in the URL for this request. 

### HTTP Request

The `eventId` may either be the system ID for the event, or the `extId` you may have specified for this event if writing this event occurred using this API. 

### HTTP Response 

**Error Codes**

| Code | Reason
|------|-------|
| 400 | Bad request, event ID was not provided in the URL |
| 404 | Not found, event was not found for the ID provided

This request will return a list of Check-in models (see below for details)

## Fetch Check-ins

`GET /events/checkins?start=<long>&end=<long>&limit=<int>&page=<int>&key=<API KEY>&appClientId=<CLIENT ID>`

Query for a list of Check-ins, regardless of the event (so the results will return Check-ins for a mix of events, but each event is fully included in the payload so that you can understand exactly what event a given Check-in is for)

### HTTP Request

**Query Parameters**

| Parameter | Possible Value |
|------|-------|
| `start` | Start time, lower bounds, (time stamp in milliseconds) using the "created" time for the check-in |
| `end` | End time, upper bounds (time stamp in milliseconds) using the "created" time for the check-in |
| `limit` | Limit the number of records to return. Defaults to 100 |
| `page` | What page of results to return (used when paging through results. The response will return the full record count so you can properly page through results |

### HTTP Response 

> RSVP Response

```json
{
	"count" : <int>,
	"checkins" : [
		<rsvp model>,
		<rsvp model>
	]
}
```

| Field | Type |
|-----|------|
| `count` | record count for this query |
| `rsvps` | List of RSVP models |

# Transactions

## Fetch Transactions By Type

`GET /transactions/get/<type>?start=<long>&end=<long>&page=<int>&limit=<int>&key=<API KEY>&appClientId=<CLIENT ID>`

Fetch a list of transactions based on a type. A Type may be for event tickets or gifts made via Giving Pages. 

### HTTP Request

**Query String Parameters**

| Parameter | Possible Value |
|------|------|
| `type` | This may either be: `event` or `gift`. This is parameter is **required** |
| `start` | Time stamp (epoch in milliseconds) for the lower bounds of when the transaction was created |
| `end` | Time stamp (epoch in milliseconds) for the upper bounds of when the transaction was created |
| `page` | For paginating, what current "page" to fetch |
| `limit` | Limit the number of results. Defaults to 100 |

### HTTP Response

**Error Codes**

| Code | Reason |
|----|-----|
| 400 | Bad request, the type was not specified |
| 404 | Not found, route not found if type was not specified |
| 500 | Unexpected error occurred. This has been logged and details will be provided with this response |

> Transactions Response

```json
{
	"count" : <int>,
	"transactions" : [
		<transaction model>,
		<transaction model>
	]
}
```

It is important to note that at no time is any payment information provided in this response, in that QuadWrangle **does not store** payment information at any time. 

## Fetch Ticket Sales for an Event

`GET /transactions/event/<eventId>?start=<long>&end=<long>&limit=<int>&page=<int>&key=<API KEY>&appClientId=<CLIENT ID>`

Fetch transactions for a given event (these transactions are for event ticket sales). 

### HTTP Request 

`eventId` is required and is either the system ID, or the external ID (`extId`) specified for the event. 

**Query Parameters**

| Parameter | Possible Value |
|-----|------|
| `eventId` | Required and is either the system ID, or the external ID (`extId`) specified for this event |
| `start` | Timestamp (epoch in milliseconds) for the lower bounds for when transactions were created |
| `end` | Timestamp (epoch in milliseconds) for the upper bounds for when transactions were created |
| `limit` | Limit the number of records records, defaults to 100 |
| `page` | What page of results to return |


### HTTP Response

**Error Codes**

| Code | Reason |
|---|----|
| 400 | Bad request, ID was not provided |
| 404 | Could not find the event for the ID provided |

> Transaction Response

```json
{
	"count" : <int>,
	"transactions" : [
		<transaction model>,
		<transaction model>
	]
}
```

# News

News items appear within the app, desktop and is also made available in emails (via the dynamic content module for curated news items). 

News (aka Posts) are stored within the system and are then used by the curation system to recommend content to end-users. This API can be used to enter posts into our system and are then made available for distribution and curation immediately. 

Typically news items enter the system via our automated web crawling system, but this API provides direct access to writing these items. 

## Bulk Load

`POST /posts?key=<API KEY>&appClientId=<CLIENT ID>`

Bulk load news items into the system. 

### HTTP Request

This request is expecting a JSON body:

```json
[
	<post model>,
	<post model>
]
```

It is important to note that this API will attempt to look up each post by the `originUrl` to avoid duplicate entries. You may also use the `id` field if you have it. 

### HTTP Response

**Error Codes**

| Code | Reason |
|----|-----|
| 400 | Bad request, body was blank, or JSON was malformed |
| 500 | Unexpected error occurred, and has been logged, and this response will have details on this error |

> Load News Response

```json
{
	"errorCount" : <int>,
	"successCount" : <int>,
	"time" : <int>,
	"errors" : [
		<post model>,
		<post model>
	]
}
```

Post models in the errors list will have its `reason` field populated

## Insert a Single News Item

`POST /post?key=<API KEY>&appClientId=<CLIENT ID>`

Save a new post to the system. 

### HTTP Request

This endpoint expects JSON in the body in the form of a Post model. You may specify an `originUrl` or an `id` to update the post, and if no means of identification are provided, this will insert a new item. 

### HTTP Response

**Error Codes**

| Code | Reason |
|----|-----|
| 400 | Bad request, the body was blank, or the JSON was malformed |
| 500 | Unexpected error while trying to write this post. Details of the error will be provided in this response |

This response will include a full Post model with the system ID set

## Update a Single News Item

`PUT /post?key=<API KEY>&appClientId=<CLIENT ID>`

Update an existing news item.

### HTTP Request

This endpoint expects a JSON body in the form of a Post model, with either an `id` or an `originUrl` set (so this post can be looked up. 

### HTTP Response

**Error Codes**

| Code | Reason
|----|-----|
| 400 | Bad request, JSON was blank, the JSON was malformed, or this request did not include `id` or `originUrl` was not included in the JSON |
| 500 | Unexpected error writing this news item - details will be provided in this response |

This response will include a full Post model. 

## Delete a Single News Item

`DELETE /post/<id>?key=<API KEY>&appClientId=<CLIENT ID>`

Delete a post item

### HTTP Request

This `DELETE` expects a valid Post `id` in the URL

### HTTP Response

**Error Codes**

| Code | Reason |
|----|-----|
| 400 | Bad request, missing ID, or the ID was not a valid ID |
| 404 | Not found, post was not found for this ID |

Returns an HTTP 200 (Ok) response

# Isaac

You may use certain Isaac functions to fetch feeds for a given set of constituents. The input is essentially a search query and the result is the curated feed Isaac has created for these people. 

There is a limit built in that limits the search query to active users only (those users who have logged in to the platform)

## Fetch Feeds By User Query 

`POST /isaac/search?key=<API KEY>&appClientId=<CLIENT ID>`

Search for users, fetching their full curated feeds (if a curated feed has been created). 

### HTTP Request

This request expects a body with JSON:

> Search Filter Model

```json
{
	"classYear" : "<string>",
	"updatedStart" : <long>,
	"updatedEnd" : <long>,
	"email" : "<string>",
	"recordId" : "<string>",
	"name" : "<string>",
	"affiliation" : "<string>",
	"affinity" : "<string>",
	"isActive" : <boolean>,
	"isVerified" : <boolean>,
	"hasFacebook" : <boolean>,
	"hasLinkedin" : <boolean>,
	"hasEmail" : <boolean>,
	"limit" : <int>,
	"offset" : <int>
}
```

Search filter parameters:

| Name | Data Type | Values |
|-------|-------|--------|
| `classYear` | String | if this value can be evaluated as a simple Integer (for example 1996) then this query will use a single class year. If this value contains commas (example: 1996,1997,1998) then the query will use each of these class years. Lastly, if this value contains a dash, we will use a range (example: 1990-1996). You may do "greater than" or "less than": `>2014` (greater than or equal to class year 2014) or: `<2016` (less than or equal to class year 2016) |
| `updatedStart` | Long | Timestamp (in milliseconds) to use as a lower bounds to fetch records by their last updated time stamp |
| `updatedEnd` | Long | Timestamp (in milliseconds) to use as an upper bounds to fetch records by their last updated time stamp |
| `email` | String | Email to search by |
| `recordId` | String | Use the original ID from the database of record |
| `name` | String | Name search. If this value is a single value, then this will be a  last name search, if this value contains spaces, then this will be a first and last name search |
| `affiliation` | String | Search by one of the built in affiliations (ALUM, FACULTY etc.), You may specify a list using a comma separated list: ALUM,FACULTY,... |
| `affinity` | String | Search by one of your custom affinities |
| `isActive` | Boolean | Set to `true` to only return results of active users. `false` will return only those who are not active. Null value means this field is ignored. For this request this is set to `true` in that only active users currently will have curated feeds | 
| `isVerified` | Boolean | Set to `true` to only return those constituent records who have been verified (have Directory access). Null value means this field is ignored |
| `hasFacebook` | Boolean | Include person records that have logged into the QW Platform using Facebook. Null value means this field is ignored |
| `hasLinkedin` | Boolean | Include person records that have logged into the QW Platform using LinkedIn. Null values means this field is ignored |
| `hasEmail` | Boolean | Only include records that have an email address. Null values means this field is ignored |
| `limit` | Integer | Limit result set to this many rows per request |
| `offset` | Integer | What "page" of results do you want? |

You do not need to include all of the these fields in your request object, this just helps narrow down your searches. 
 
### HTTP Response

**Error Codes**

| Code | Reason |
|----|----|
| 400 | Bad request, request body was blank, JSON malformed, search parameters not valid. This response will include details |

> Feed Response 

```json
{
	"feeds" : [
		{
			"userId" : "<string>",
			"recordId" : "<string>",
			"email" : "<string>",
			"fullName" : "<string>",
			"feed" : [
				<feed item model>,
				<feed item model>
			]
		},
		...
	]
}
```

| Field | Values |
|-----|------|
| `userId` | The system ID for this user |
| `recordId` | If this user has a record ID (set from the database-of-record) it will be populated |
| `email` | Email address (primary) for this user (useful for knowing who this is, possibly for display purposes) |
| `fullName` | Full name of this user, again useful for display purposes|



## Fetch Feed For a Single User

`GET /isaac/detail/<id>?key=<API KEY>&appClientId=<CLIENT ID>`

### HTTP Request

This request expects a valid ID (can either be a User system ID or a record ID for a person).

### HTTP Response

**Error Codes**

| Code | Reason |
|---|---|
| 400 | Bad request, the ID provided was not provided |
| 404 | Not found, no person record was found for this ID |

> Feed Response

```json
{
	"feed" : [
		<feed item>,
		<feed item>
	]
}
``` 

## Fetch Recommended CTAs for a Single User

`GET /isaac/ctas/<id>?key=<API KEY>&appClientId=<CLIENT ID>`

You may fetch recommended Calls To Action the curation system found to be relevant for a particular person within the QW platform. 

See below in the Call To Action section for a description of what a Call To Action is. 

### HTTP Request

This request expects a valid ID (which may either be a person system ID, or your original record ID)

### HTTP Response

**Error Codes**

| Code | Reason |
|---|---|
| 400 | Bad request, the ID was not provided
| 404 | Not found, a person record was not found for this ID |

> CTA Response

```json
{
	"ctas" : [
		<call to action model>,
		<call to action model>
	]
}
```


# Gift History

## Bulk Load

`POST /gifts/load?key=<API KEY>&appClientId=<CLIENT ID>`

Bulk load gift history. Each gift history item is associated with a person record and represents a gift, pledge or a paid pledge that you want recorded within the QuadWrangle system. This information is used to aid in curation and other machine learning systems within the platform. 

### HTTP Request

This request expects a JSON body that takes the following form:

```json
{
	"history" : [
		<gift history model>,
		<gift history model>
	]
}
```

Within the gift history model you can specify an `extId` (external ID) that can be used to index each item, and a new system ID will be assigned to each as well. 

Additionally, this operation is done in a detached thread from the request thread so that loading happens asynchronously from your request. 

### HTTP Response

**Error Codes**

| Code | Reason |
|----|----|
| 400 | Bad request, body was blank, malformed JSON, or no valid items to save. Details will be provided with this response |

> Gift History Load Response

```json
{
	"errors" : [
		<gift history model>,
		<gift history model>
	],
	"successCount" : <int>,
	"errorCount" : <int>
}
```

Within the list of any errors there will be a `reason` field populated with the potential cause for this gift history item failing to be written. 

## Insert a Single Item

`POST /gift?key=<API KEY>&appClientId=<CLIENT ID>`

Insert (or update) a new gift history item. 

### HTTP Request

This request expects a JSON body in the form of a Gift History Item model, that may or may not contain and `id` or an `extId` (if any of those are present, the system will attempt to look up this item and perform an update).

### HTTP Response

**Error Codes**

| Code | Reason |
|----|-----|
| 400 | Bad request, request body was blank, JSON was malformed, or there was an error attempting to write this item |

This response will include a full Gift History Item model with the system `id` field populated with a value.

## Update a Single Item

`PUT /gift?key=<API KEY>&appClientId=<CLIENT ID>`

### HTTP Request

This request expects a JSON body, and must contain either a system `id` or the `extId` field to have a valid value. 

### HTTP Response

**Error Codes**

| Code | Reason |
|-----|------|
| 400 | Bad request, request body was blank, JSON was malformed, missing `id` or `extId` within the JSON, or an error occurred writing this item. Details will be provided with this response |
| 404 | Not found, this gift history item was not found (using either `id` or `extId`) |

This response will include a full Gift History Item model. 

## Delete a Single Item

`DELETE /gift/<id>?key=<API KEY>&appClientId=<CLIENT ID>`

### HTTP Request

This request requires an ID to be passed in the URL. This ID may either be a system ID, or an `extId` (external ID) you specified when saving this gift history item. 

### HTTP Response

**Error Codes**

| Code | Reason |
|-----|-----|
| 400 | Bad request, ID was missing |
| 404 | Not found, this gift item was not found using the ID provided to look it up |

Return an HTTP 200 (Ok) response

# QuadMail

Endpoints to expose certain functions for QuadMail.

## Fetch Unsubscribes

`GET /qm/unsubscribes?key=<API KEY>&appClientId=<CLIENT ID>&start=<timestamp>&end=<timestamp>`

Fetch a list of unsubscribes. 

### HTTP Request

| Parameter | Possible Value |
|----|-----|
| `start` | Start time stamp (epoch in milliseconds) for a lower bounds of the query |
| `end` | End time stamp (epoch in milliseconds) for an upper bounds of the query |

### HTTP Response 

Returns a list of Unsubscribe objects (see below for the Unsubscribe model definition)

```json
[
	<unsubscribe object>,
	<unsubscribe object>
]
```

Returns empty list for query that returned no results


### 

# Models

## Person

The Person model defines each person/constituent record. You can see a full illustration of the model to the right. This section will describe each field and it's appropriate values

> Person Model

```json
{
"recordId" : "<string>",
"otherId" : "<string>",
"id" : "<string>",
"firstName" : "<string>",
"lastName" : "<string>",
"maidenName" : "<string>",
"birthday" : "<string>",
"classYear" : <int>,
"nomenclature" : "<string>",
"email" : "<string>",
"altEmails" : [
	"<string>","<string>", ...
	],
"affiliations" : [
	"<string>", "<string>", ...
	],
"addresses" : [
	{
		"label" : "<string>",
		"street" : "<string>",
		"address2" : "<string>",
		"city" : "<string>",
		"state" : "<string>",
		"country" : "<string>",
		"zipCode" : "<string>",
		"phone" : "<string>"
	}, ...
	],
"address" : "<string>",
"address2" : "<string>",
"city" : "<string>",
"state" : "<string>",
"country" : "<string>",
"postalCode" : "<string>",
"phone" : "<string>",
"greeks" : [
	"<string>", "<string>", ...
	],
"teams" : [
	"<string>", "<string>", ...
	],
"clubs" : [
	"<string>", "<string>", ...
	],
"academics" : [
	{
		"degree" : "<string>",
		"major" : "<string>",
		"year" : <int>
	}, ...
	],
"notes" : "<string>",
"spouse": {
	"firstName" : "<string>",
	"lastName" : "<string>",
	"affiliation" : "<string>",
	"classYear" : <int>,
	"recordId" : "<string>"
	},
"employer" : "<string>",
"jobTitle" : "<string>",
"donorStatus" : "<string>",
"lastFiverYearsTotals" : [
	<float>, <float>, <float>, <float>, <float>
	],
"hasDirectoryAccess" : <boolean>,
"publicProfile" : <boolean>,
"doNotContact" : <boolean>,
"doNotEmail" : <boolean>,
"suppressGiving" : <boolean>,
"isModerator" : <boolean>,
"fbPageIds" : [
	"<string>",
	"<string>"
],
"fbAccessToken" : "<string>",
"liAccessToken" : "<string>",
"linkedinBio" : "<string>",
"biography" : "<string>"
}
```

| Field  | Data Type  |   Possible Values  |
|---------------|----------------|---------------|
| `recordId`    |   String   |  Your original database ID for this record (_REQUIRED_) |
| `otherId`    |   String   |   Any additional ID you may want to specify for this record |
| `id`	| String | the QuadWrangle ID for this record. This will be needed if you want to fetch this record, or save it back to QuadWrangle if you make changes to this record | 
| `firstName`	|	String	| first name for this person (_REQUIRED_) |
| `lastName`	|	String	| last name for this person (_REQUIRED_) |
| `maidenName`	| String	| maiden name for this person |
| `birthday`	| String	| Valid formats: `m/d/yyyy` or `m-d-yyyy` |
| `classYear`	| Integer	| Valid format: `yyyy` - This is the preferred class year. Can be 0 if not an alum
| `nomenclature`	| String	| Any valid string if there are preferred class year designations, for example: '96 MA, '02 PhD.	|
| `email`	| String	| Primary email address for this person. May be blank |
| `altEmails`	| List	| A list of other email addresses	|
| `affiliations` | List | A list of valid affiliations. See Affiliations section for a full list |
| `addresses` | List	| A list of Address objects. See Address section for details
| `address`	| String | Primary address |
| `address2` | String | Primary address, line 2 |
| `city` | String | Primary city |
| `state` | String | Primary state |
| `country` | String | Primary country |
| `postalCode` | String | Primary postal code |
| `phone` | String | Primary phone for this person |
| `greeks` | List | List of greek affiliations | 
| `teams` | List | List of teams/athletic orgs for this person |
| `clubs` | List | List of clubs for this person | 
| `academics` | List | List of Academic objects - to define degrees/major and year. See section on Academics |
| `notes` | String | Any relevant gift officer notes for this person |
| `spouse` | Object | A spouse object for this person. See the Spouse section for details |
| `employer` | String | Employer for this person |
| `jobTitle` | String | Job title for this person |
| `donorStatus` | String | A valid donor status. Can be one of: `CURRENT`, `NEVER`, `SYBUNT`, `LYBUNT` |
| `lastFiverYearsTotals` | List`<float>` | A 5-element list where each element is a value for gifts made in the past 5 years. Element 0 is 5 years back, element 1 is 4 years back and so on with element 5 being last financial year total |
| `hasDirectoryAccess` | boolean | Control whether or not this person has access to the Directory feature within the QuadWrangle platform |
| `publicProfile` | boolean | Set if this person will have a public profile or not |
| `doNotContact` | boolean | Set the "do not contact" flag for this person. This means we will not email, SMS, or send any communications other than system-triggered emails |
| `doNotEmail` | boolean | Do not send this person any emails |
| `suppressGiving` | boolean | If set to true, we will not deliver any automated CTA for giving opportunities |
| `isModerator` | boolean | Set if this person is a moderator. Moderators have access to certain features within the platform, like moderating event photos |
| `fbPageIds` | List (of strings) | If you have a Facebook App that has the `user_likes` permission, and your system records pages liked by this person, you can pass in a list of Facebook Page IDs, and we will use that information to build the natural language profile for this person. |
| `fbAccessToken` | String | If you have a Facebook App configured, and the access token your app retrieves for this user can be passed in and we can use the profile information (per the permissions of the Facebook App) to build the natural language profile for this person. |
| `liAccessToken` | String | If you have a configured LinkedIn App and you record the access token your app retrieves, you can pass in this access token and our system can fetch the LinkedIn profile of this person to build the natural language profile | 
| `linkedinBio` | String | If you have a LinkedIn App configured, and you retrieve the LinkedIn biography for this person, you can pass that in to build the natural language profile for this person. |
| `biography` | String | If you have unstructured, descriptive, biographical information for this person, it may be used to help build the natural language profile for this person. |

Only those fields marked as _REQUIRED_ are required and all other fields are optional. 

## Affiliations

We currently have several built-in affiliations you can use to identify record types within the system. However, we also have a facility to add your own "school defined affinities" where you can further tag your constituent records. 

In the `Person` model you can define a list of affiliations:

```json
"affiliations" : [
	"ALUM", 
	"FRIEND", 
	"Your custom affinity 1", 
	"Another custom affinity"
]
```
Below is a list of built-in affiliations:
 - `ALUM`
 
 - `CURRENT` (current student)
 
 - `PARENT`
 
 - `FACULTRY`
 
 - `STAFF`
 
 - `ATTENDED` (attended but did not graduate)
 
 - `FRIEND`
 
 - `CURRENT_PARENT` (current parent of a current student)
 
 - `ALUM_ASSOC_BOARD`  (alumni association board)
 
 - `FORMER_TRUSTEE` (former trustee)
 
 - `FORMER_ALUM_ASSOC_BOARD` (former alumni association board)
 
 - `HONORARY_TRUSTEE` 
 
 - `FORMER_STAFF`
 
 - `FORMER_FACULTY`
 
 - `CORPORATOR`
 
 - `FORMER_CORPORATOR`
 
 - `GRANDPARENT`
 
 - `RETIRED_FACULTY`
 
 - `RETIRED_STAFF`
  
When using the built-in affiliations, it is critical to use the exact spelling and use all upper-case (as the above example shows).

For any other value provided in this `affiliations` list will be treated as a "school defined affinity." These attributes are usable for the purposes of audience segmentation via the List Builder tools. 

## Change Set

A Change Set represents a single change to a field on the person data. The model you will receive is groups Change Sets based on the person record so you will receive a List of Change Sets.

> Change Set Model

```json
{
	"id" : "<string>",
	"fieldName" : "<string>",
	"person" : "<person object>",
	"newValue" : "<string>",
	"oldValue" : "<string>",
	"created" : <long>,
	"modified" : <long>,
	"acceptedTimestamp" : <long>	
}
```

| Field  | Description  |
|---------------|----------------|
| `id` | The system ID for this change set. You'll need this if you choose to programmatically accept change sets |  
| `fieldName`    |   what field does this Change Set represent? Can be: FIRSTNAME, LASTNAME, MAIDENNAME, EMAIL, ADDRESS, ADDRESS2, CITY, STATE, COUNTRY, ZIPCODE, EMPLOYER, TITLE   |
| `person` | A person object of the person who owns this change set | 
| `newValue`    |   The current value for this field   |
| `oldValue` | The original value for this field |
| `created` | Timestamp when this change set was created |
| `modified` | Timestamp when this change set was last updated |
| `acceptedTimestamp` | Timestamp set if this is accepted |


## Address

Address can be used to store any contact information you may have for a constituent. You can also use this to store alternate phone numbers as well.

> Address Model

```json
{
	"label" : "<string>",
	"street" : "<string>",
	"address2" : "<string>",
	"city" : "<string>",
	"state" : "<string>",
	"country" : "<string>",
	"zipCode" : "<string>",
	"phoneNumber" : "<string>"
}
```

| Field  | Data Type  | Possible Value
|---------------|----------------|----------|
| `label`    |   String   | A name identifying what this address is for, for example, Business, or Alternate Phone etc. (_OPTIONAL_)| 
| `street`    |   String   | Street address (_OPTIONAL_) | 
| `address2` | String | Address line 2 (_OPTIONAL_) |
| `city` | String | City (_OPTIONAL_) |
| `state`  | String | State (_OPTIONAL_) | 
| `country` | String | country  (__OPTIONAL_) |
| `zipCode`	| String | zip code  (_OPTIONAL_) |
| `phoneNumber` | String | phone number (_OPTIONAL_) |

## Spouse

The Spouse model used for embedding any spouse information on a Person record

> Spouse Model

```json
{
	"firstName" : "<string>",
	"lastName" : "<string>",
	"affiliation" : "<Affiliation>",
	"classYear" : <int>,
	"recordId" : "<string>"
}
```

| Field  | Data Type  | Possible Value
|---------------|----------------|----------|
| `firstName` | String | First name of this spouse (_REQUIRED_) |
| `lastName` | String | Last name of this spouse (_REQUIRED_) | 
| `affiliation` | Affiliation | Can be a single value of a built-in affiliation type _OPTIONAL_ |
| `recordId` | String | If there is a database of record ID for this person, you may specify it here _OPTIONAL_ |

## Academics

The Academics model is used on the `Person` object used to define majors/degrees for this person. You will notice that on the `Person` object you may specify a list of Academics models (in that many people will have multiple degree/major combinations)

> Academics Model

```json
{
	"year" : <int>,
	"degree" : "<string>",
	"major" : "<string>"
}
```

| Field | Data Type | Possible Value |
|-------|-------|----------|
| `year` | integer | `yyyy` (example: 1996) |
| `degree` | String | Name of this degree. This will be looked up in the degrees collection and if not found there a new one will be added. example: Bachelors of Science |
| `major` | String | Name of the major. This will be looked up in the majors collection and if not found there a new one will be added. example: Computer Science |

## Checkin

The Check In model is used to represent a person's check-in in to an event. The check-in process can also be performed in QuadHub on the Event's report detail screen.

> Checkin Model

```json
{
	"person" : <person object>,
	"timestamp" : <long>,
	"event" : <event object>
}
```

| Field | Data Type | Possible Value |
|-----------|----------------|---------------|
| `person` | Person Object | All possible values for a person per the model defined above |
| `timestamp` | long | Timestamp that this check in occurred |
| `event` | Event Object | The event that this check in occurred |

## Event

The event model represents a particular event within the platform. Events have start and end times as well as the possible addition of a location (venue) or an event may be an on line event (such as a webinar). 

> Event Model

```json
{
	"id" : "<string>",
	"extId" : "<string>",
	"title" : "<string>",
	"content" : "<string>",
	"start" : <long>,
	"end" : <long>,
	"published" : <boolean>,
	"imageUrl" : "<string>",
	"timezone" : "<string>",
	"venue" :  "<string>",
	"venueAddress" : "<string>",
	"venueCity" : "<string>",
	"venueState" : "<string>",
	"venueCountry" : "<string>",
	"venueZip" : "<string>",
	"lat" : <float>,
	"lng" : <float>,
	"created" : <long>,
	"modified" : <long>,
	"paymentProcessor" : "<string>",
	"paymentProcessorId" : "<string>",
	"accountName" : "<string>",
	"accountId" : "<string>",
	"sharableUrl" : "<string>",
	"registrationUrl" : "<string>",
	"eventUrl" : "<string>",
	"childEvents" : [
		<Event Object>
	],
	"tickets" : [
		<Ticket Object>
	],
	"errors" : [ <string> ],
	"reason" : "<string>"
}
```
| Field | Data Type | Possible Value |
|----------|----------|----------|
| `id` | String | System ID for this event - generated by QW |
| `extId` | String | External ID, any unique identifier you wish to use, and this can be used to look up an event as well if this field is used by you. |
| `title` | String | Title for this event, shows up on card titles |
| `content` | String | Content for this event. This may contain basic HTML. See note below on what allowed HTML tags are allowed | 
| `start` | long | The start date/time (in epoch with milliseconds) for this event |
| `end` | long | the end date/time (in epoch with milliseconds) for this event |
| `published` | boolean | is this event published? If not published, it will be set as "Waiting for Review" in our system and will NOT be displayed to any end-user |
| `venue` | String | Name of the venue - if this is an online event (an event with an `eventUrl` then this is optional |
| `venueAddress` | String | Street address for the venue - optional if event is online |
| `venueCity` | String | City for the venue - optional if event is online |
| `venueState` | String | State for this venue - optional if event is online |
| `venueCountry` | String | The country for this venue - optional if event is online |
| `lat` | float | latitude for the venue, this is system generated based on venue address, so you may leave this blank |
| `long` | float | longitude for the venue, this is system generated based on venue address, so you may leave this blank |
| `created` | long | timestamp (epoch in milliseconds) when this event was created |
| `modified` | long | timestamp (epoch in milliseconds) when this event was created |
| `paymentProcessor` | String | Available on reads, if this event has a payment processor (for the handling of ticket purchases and/or gifts being made) this will display the name of the processor. These are setup in QuadHub Admin Tools :: Payment Processors |
| `paymentProcessorId` | String | Set this on write operations if you wish to associate a payment processor with this event. You may use QuadHub Admin Tools :: Payment Processors to get the IDs for your payment processor(s) |
| `accountName` | String | Available on reads, if this event has a payment processor, the school account associated with this event. These are setup in QuadHub Admin Tools :: Payment Processors
| `accountId` | String | Set this on write operations if you are wanting to associate a payment processor for this event. You may get the ID's via QuadHub Admin Tools :: Payment Processors  |
| `sharableUrl` | String | The URL we can use when an end-user wishes to share this event detail screen. By default it's the URL within QuadWeb, but may be overridden with this field |
| `registrationUrl` | String | If this event handles RSVPs and or ticket sales outside of QW, you may provide this URL so that the end-user may be directed to the appropriate location to RSVP/Register or purchase tickets |
| `eventUrl` | String | If this event is an online event (no location, such as a webinar) than you may provide this field with a full URL of where your end-users will go to attend this event |
| `childEvents` |List of Events | List of child events |
| `tickets` | List of Tickets | List of ticket objects. If you are writing new events, you may provide 0 or many tickets for this event. Any ticket object in this list that does not have an `id` value will create a new ticket in our system. If you are adding tickets to an event, it would be a good idea to provide a `paymentProcessorId` and `accountId` fields to your Event model so that the appropriate payment processor/account combination can be associated with this event. If you are unsure of what the ID values are, you may get these from QuadHub Admin Tools :: Payment Processor |
| `errors` | List (String) | This will be set if there are errors on this event during a write operation.
| `reason` | String | If a write failed, a summary will be provided here |

### A note about child events
Our event data model provides for infinite "nesting" of events which enables the creation of complex multi-day events that have "sub" events. These child events show up on the parent event detail screen as "Other Events." The best example to use is Homecoming Weekend, which includes many events throughout a weekend but we have our "umbrella" event of "Homecoming Weekend." Each child event can have its own ticketing, payment processors, everything that can be set on any other event. 

### Valid HTML

Given the nature of HTML and ensuring consistent and stable user experiences across mobile and desktop environments we have a "white list" of allowed HTML for content areas. Those tags include:

* a
* b
* blockquote
* br
* cite
* code
* dd
* dl
* dt
* em
* i
* li
* ol
* p
* pre
* q
* small
* strike
* strong
* sub
* sup
* u
* ul

## Page

The page model represents an individual Giving Page within our system. A Giving Page is a page that has a specific appeal for gifts to be made, has its own description/content, imaging, payment processor, form etc. 

Giving Pages are managed through QuadHub "Giving" section. This model you will see when reporting on transactions made through the QW platform in that any transaction you read that has been the result of a gift will have an associated Giving Page, and we provide critical details for that giving page on those read requests. 

> Page Model

```json
{
	"id" : "<string>",
	"slug" : "<string>",
	"title" : "<string>",
	"created" : <long>,
	"modified" : <long>,
	"paymentProcessor" : "<string>",
	"accountName" : "<string>",
	"externalCode" : "<string>"
}
```

| Field | Data Type | Possible Value |
|----------|------------|----------|
| `id` | String | System generated ID from QW |
| `slug` | String | The slug is the human-and-machine friendly version of the ID. This may be something like "this-is-a-giving-page" |
| `title` | String | Title of this page | 
| `created` | Long | Timestamp (epoch with milliseconds) when this page was created |
| `modified` | Long | Timestamp (epoch with milliseconds) when this page was last modified |
| `paymentProcessor` | String | The name of the payment processor being used for this page |
| `accountName` | String | The name of the account that is being used by the payment processor |
| `externalCode` | String | If this giving page has an external code to make it easier on your to reconcile giving pages with systems on your end, will be provided here |

## RSVP Model

The RSVP model is returned when querying for RSVPs for a given timeframe or for a specific even. 

> RSVP Model

```json
{
	"person" : <person object>,
	"event" : <event object>,
	"response" : "<string>",
	"timestamp" : <long>
}
```

| Field | Data Type | Possible Value |
|-------|---------|----------|
| `person` | Person Object | A full person object of the person who created this RSVP response model | 
| `event` | Event Object | A full event object associated with this RSVP |
| `response` | String | YES or NO |
| `timestamp` | Long | Timestamp (epoch in milliseconds) for when this response was created |

## Transaction

This model would be created by either an event ticket purchase or a gift being made by an end-user (or via the Purchase Tickets feature in QuadHub). 

Several fields are specific to the type of payment processor API being used, so not all values will be populated in that our model has to accommodate several disparate payment processor gateway API systems from 3rd parties (like Authorize.Net, Stripe or ToucheNet and others)

> Transaction Model

```json
{
	"id" : "<string>",
	"person" : <person object>,
	"amount" : <double>,
	"timestamp" : <long>,
	"recurring" : <boolean>,
	"processed" : <boolean>,
	"canceled" : <boolean>
	"canceledTimestamp" : <long>,
	"hasBeenRefunded" : <boolean>,
	"refundedTimestamp" : <long>,
	"refundTransactionId" : "<string>",
	"description" : "<string>",
	"source" : "<string>",
	"wasRecurringPayment" : <boolean>,
	"paymentProcessor" : "<string>",
	"accountName" : "<string>",
	"anon" : <boolean>,
	"payrollDeduction" : <boolean>,
	"payrollDeductionConfirmed" : <boolean>,
	"payrollDeductionConfirmedAt" : <long>,
	"pledge" : <boolean>,
	"pledgeNumber" : "<string>",
	"joint" : <boolean>,
	"spouse" : "<string>",
	"customerId" : "<string>",
	"subscriptionId" : "<string>",
	"details" : {
		"page" : <page object>,
		"event" : <event object>
	}
}
```
| Field | Data Type | Possible Value | 
|-------|--------|----------|
| `id` | String | System generated ID for this transaction |
| `person` | Person Object | Person who created this transaction |
| `amount` | Double | The amount that was charged as part of this transaction |
| `timestamp` | Long | Timestamp (epoch in milliseconds) of when this transaction was processed |
| `recurring` | Boolean | Is this transaction for a recurring gift? |
| `processed` | Boolean | Has this transaction been successfully processed by the underlying payment processor gateway system? |
| `canceled` | Boolean | Was this transaction canceled? End-users initial a cancelation if this transaction is for a recurring gift. This signals that the user has canceled their recurring gift |
| `canceledTimestamp` | Long | Timestamp (in milliseconds) of when this transaction was canceled |
| `hasBeenRefunded` | Boolean | Has this transaction been refunded? |
| `refundedTimestamp` | Long | Timestamp (in milliseconds) when this transaction was refunded |
| `refundTransactionId` | String | The unique code/transaction ID the payment processor gateway assigned for this refund transaction | 
| `description` | String | A brief description of this transaction |
| `source` | String | Short code assigned to the parent object that originated this transaction |
| `wasRecurringPayment` | Boolean | If this transaction was created from the recurring payment system |
| `paymentProcessor` | String | Name of the payment processor used for this transaction. You may see/configure payment processors in QuadHub Admin Tools |
| `accountName` | String | Name of the account associated with this payment processor. You can see/configure accounts in QuadHub Admin Tools |
| `anon` | Boolean | Was this transaction created by an anonymous user? (meaning they were not logged in at the time of this transaction being created) |
| `payrollDeduction` | Boolean | Was this a gift where the donor was either faculty or staff and wanted to use payroll deduction as a means to complete this transaction |
| `payrollDeductionConfirmed` | Boolean | As part of our system of accepting payroll deduction for gifts, we send out an email to the donor to confirm that they initiated this gift. The donor clicks a link in this email which confirms the request |
| `payrollDeductionConfirmedAt` | Long | Timestamp (in milliseconds) when the confirmation occurred |
| `pledge` | Boolean | Was this transaction the payment for a pledge made? |
| `pledgeNumber` | String | The pledge number/ID assigned to the donor's pledge they can provide to help reconcile this transaction to a pledge |
| `joint` | Boolean | Was this transaction a "joint" gift? |
| `spouse` | String | If this was a joint gift, the name of the donor's spouse will be provided |
| `customerId` | String | Some payment processors will create unique customer IDs for a transaction to help you identify/reconcile transactions from one system to the next. |
| `subscriptionId` | String | Some payment processors will create a subscription ID for recurring payments. In these systems recurring payments are considered subscriptions and a unique ID may be assigned. This depends on the payment processor gateway being used |
| `details` | Hash Map | There will be a  key in this hash map that holds the entity where this transaction originated from. Currently the are two keys: `page` and `event` and each will reference the corresponding models |

## Post

The Post model represents a news post

> Post Model

```json
{
	"id" : "<string>",
	"title" : "<string>",
	"content" : "<string>",
	"publishDate" : <long>,
	"published" : <boolean>,
	"imageUrl" : "<string>",
	"athletics" : <boolean>,
	"isFeatured" : <boolean>,
	"expires" : <long>,
	"sourceContent" : "<string>",
	"originUrl" : "<string>",
	"commenting" : <boolean>
}
```

| Field | Data Type | Possible Values |
|--------|--------|---------|
| `id` | String | System ID for this news item |
| `title` | String | Title for this news item, this shows up as the headline on the news item detail screen and as the title on the summary cards in mobile and desktop |
| `content` | String | Body content for this news item. This may contain basic HTML, see note below for details on what HTML is allowed |
| `publishDate` | Long | Time stamp (epoch in milliseconds) of when this news item was posted |
| `published` | Boolean | Is this news item published? If `true` then this will be made LIVE on the platform. If `false` the publish status will be set to WAITING |
| `imageUrl` | String | A full URL of an image you wish to use for this news item. This will appear on the detail screen for this news item and on summary cards. This is not required |
| `athletics` | Boolean | Is this news item an athletics story? The reason you may set this is that there are settings to limit how much athletics content items get served to end-users. This is optional and default is `false` |
| `isFeatured` | Boolean | Is this a featured news item? Featured news items show up at the top of the news feeds on mobile and on desktop (in the "hero" section of the news page). If you are setting this to true, it is a good idea to make sure there is an `imageUrl` set |
| `expires` | Long | Time stamp (epoch in milliseconds) of when this news item expires. Expiration means that passed this date this news item will no longer be served or be part of the "content inventory" for the Curation system |
| `sourceContent` | String | The name of the source for this news item |
| `originUrl` | String | The full URL of where this news item was sourced from |
| `commenting` | Boolean | Set to `true` to allow comments in mobile and desktop for this news item. Defaults to `false` |

### Valid HTML

Given the nature of HTML and ensuring consistent and stable user experiences across mobile and desktop environments we have a "white list" of allowed HTML for content areas. Those tags include:

* a
* b
* blockquote
* br
* cite
* code
* dd
* dl
* dt
* em
* i
* li
* ol
* p
* pre
* q
* small
* strike
* strong
* sub
* sup
* u
* ul

## Call To Action

A Call To Action model essentially represents a URL that is designed to send an end-user to a piece of content. Calls To Action in our system are special in that they have natural language processing/machine learning elements to them in that they include a full description of what this CTA represents so that they can be recommended to the right people. 

Calls To Action can be created/edited within QuadHub. 

> CTA Model

```json
{
	"id" : "<string>",
	"title" : "<string>",
	"published" : <boolean>,
	"content" : "<string>",
	"url" : "<string>",
	"publishDate" : <long>,
	"contentType" : "<string>"
}
```

| Field | Data Type | Possible Values |
|-------|--------|----------|
| `id` | String | The system ID for this CTA |
| `title` | String | The title, as set in the |
| `published` | Boolean | Is this CTA currently "live" in the platform? |
| `content` | String | The content that describes this CTA. If this CTA was created via a Giving Page, then this content will be a direct copy of the Giving Page content |
| `publishDate` | Long | Date published Time stamp (epoch in milliseconds) |
| `contentType` | String | The type of content this CTA represents |

## Feed Item 

A feed item is a brief representation of a piece of content that the curation system "emits."

> Feed Item Model

```json
{
	"id" : "<string>",
	"title" : "<string>",
	"imageUrl" : "<string>",
	"contentType" : "<string>",
	"score" : <double>
}
```

| Field | Data Type | Possible Values |
|----|-----|-----|
| `id` | String | System ID for this feed item (maps to a `Post` ID) |
| `title` | String | Title value for this item |
| `imageUrl` | String | If there is an image URL for this post that this news feed item represents |
| `contentType` | String | The type of content. Currently only `NEWS` (post) is supported with this API |
| `score` | Double | A score value assigned by Isaac to weight this feed item |

## Gift History Item

Gift History Items belong to (or are associated with) a person record within our system and represent a gift made, a pledge made or a pledge paid that was specifically initiated by our Giving Pages feature (those items are covered by the transactions endpoints). 

> Gift History Item

```json
{
	"id" : "<string>",
	"extId" : "<string>",
	"recordId" : "<string>",
	"amount" : <double>,
	"timestamp" : <long>,
	"type" : "<string>",
	"account" : "<string>",
	"donorId" : "<string>",
	"designation" : "<string>",
	"creditType" : "<string>",
	"notes" : "<string>",
	"display" : <boolean>,
	"hasBeenPaid" : <boolean>,
	"created" : <long>,
	"modified" : <long>
}
```

| Field | Data Type | Possible Values |
|-----|------|------|
| `id` | String | System generated ID for this item |
| `extId` | String | External ID, a unique ID for this item generated by you. Can be anything like the original database row ID |
| `recordId` | String | The record ID that maps directly to a person record within the system. This is *required* and if not provided or if the person record is not found you will receive an error specifying this condition |
| `amount` | Double | the amount for this gift, pledge or paid pledge |
| `timestamp` | Long | Timestamp (in epoch milliseconds) of when this item was created |
| `type` | String | Can be either: `GIFT` `PLEDGE` or `PP` (pledge paid) |
| `account` | String | An account identifier or a string if desired, completely optional |
| `donorId` | String | An alternate ID used, not used for look ups |
| `designation` | String | What is this entry for? A description of the fund is typical here |
| `creditType` | String | Specify a credit type for this entry (optional) |
| `notes` | String | Any useful notes for this entry  (optional) | 
| `display` | Boolean | Should this item be displayed to the end-user? (optional) |
| `hasBeenPaid` | Boolean | has this entry been paid? (optional) |
| `created` | Long | Time stamp (epoch in milliseconds) when this was created in the QW system |
| `modified` | Long | Time stamp (epoch in milliseconds) when this was last modified in the QW system |

## Unsubscribe

An unsubscribe model represents a set of unsubscribe preferences set by a constituent (typically through the QuadMail Micro-site, which is accessed via a link in emails sent through QuadMail). This model will specify if a constituent has unsubscribed from all emails, or if to individual QuadMail email campaigns. 

> Unsubscribe Model

```json
{
	"userId" : "<string>",
	"recordId" : "<string>",
	"email" : "<string>",
	"firstName" : "<string>",
	"lastName" : "<string>",
	"altEmails" : [
		"<string>",
		"<string>",
		...		
	],
	"classYear" : <int>,
	"unsubscribeFromAll" : <boolean>,
	"campaignUnsubscribes" : [
		{
			"name" : "<string>",
			"isFundraiser" : <boolean>
		}
	],
	"created" : <long>,
	"modified" : <long>
}
```

| Field | Date Type | Possible Values |
|------|--------|----------|
| `userId` | String | QuadWrangle system ID for this person |
| `recordId` | String | Original ID from database of record (if present) |
| `email` | String | Primary email |
| `firstName` | String | constituent's first name |
| `lastName` | String | constituent's last name |
| `altEmails` | List | A list of alternate email addresses on file |
| `classYear` | Integer | Preferred class year, if set |
| `unsubscribeFromAll` | Boolean | set to 'true' if this constituent has opted to unsubscribe from all emails |
| `campaignUnsubscribes` | List (of objects) | A list of individual campaigns this constituent has unsubscribed from. This list will include fields for the name of the campaign and whether or not this campaign was a fund raising campaign (an option set in QuadMail campaign screen) |
| `created` | Long | Time stamp (epoch in milliseconds) when this unsubscribe object was created | 
| `modified` | Long | Time stamp (epoch in milliseconds) when this object was last modified. This object may be modified by the constituent based on their setting preferences for individual campaigns if they haven't opted to unsubscribe from all |
