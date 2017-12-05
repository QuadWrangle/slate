---
title: QuadWrangle API Reference

language_tabs:
  - shell
  - java

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

Sandbox Host: `http://sandbox.qwd.io/`


# Person

## Bulk Load

Bulk load multiple people records into the system. 


### HTTP Request

`POST /people/load?key=<API KEY>&appClientId=<CLIENT ID>`

## Insert a Single Record

## Update a Single Record

## Get All Changes

## Get a Single Change Set

# Events

## Bulk Load

## Insert a Single Event

## Update a Single Event

## Fetch RSVPs for an Event

## Fetch Check-ins for an Event

## Fetch Ticket Sales for an Event

# News

## Bulk Load

## Insert a Single News Item

## Update a Single News Item

# Isaac

You may use certain Isaac functions to fetch feeds for a given set of constituents. The input is essentially a search query and the result is the curated feed Isaac has created for these people. 

There is a limit built in that limits the search query to active users only (those users who have logged in to the platform)

## Fetch Feeds By User Query 

## Fetch Feed For a Single User

# Models

## Person

The Person model defines each person/constituent record. You can see a full illustration of the model to the right. This section will describe each field and it's appropriate values

> Person model as JSON:

```json
{
"recordId" : "<string>",
"otherId" : "<string>",
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
"isModerator" : <boolean>
}
```

| Field  | Data Type  |   Possible Values  |
|---------------|----------------|---------------|
| `recordId`    |   String   |  Your original database ID for this record (_REQUIRED_) |
| `otherId`    |   String   |   Any additional ID you may want to specify for this record |
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

Only those fields marked as _REQUIRED_ are required and all other fields are optional. 

## Affiliations

We currently have several built-in affiliations you can use to identify record types within the system. However, we also have a facility to add your own "school defined affinities" where you can further tag your constituent records. 

In the `Person` model you can define a list of affiliations:

```json
"affiliations" : ["ALUM", "FRIEND", "Your custom affinity 1", "Another custom affinity" ]
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

## Address

Address can be used to store any contact information you may have for a constituent. You can also use this to store alternate phone numbers as well.

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
| `phoneNumber | String | phone number (_OPTIONAL_) |

## Spouse

The Spouse model used for embedding any spouse information on a Person record

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


