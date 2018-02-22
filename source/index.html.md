---
title: WIAS API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript

toc_footers:
  - <a href='https://devs.wias.kopjra.com/signup'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - commonquery
  - errors

search: true
---

# Introduction

Welcome to Web Interaction Acquisition Service API! You can use our API to access Web Interaction Acquisition Service
API endpoints, which is everything you need to do to forensically acquire an interaction happening between your customer
and your services.

If you are still unfamiliar with the process used by WIAS, it'd be better if you read our [introductory documentation](https://www.kopjra.com/wias).

We have language bindings in Shell and Javascript (Node.js)! You can view code examples in the dark area to the right, and you can
switch the programming language of the examples with the tabs in the top right. You can [download our clients here](https://devs.wias.kopjra.com/clients).

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "https://gateway.wias.kopjra.com/api"
  -H "Accept: application/json"
  -H "Authorization: apikey"
```

```javascript
const wias = require('@kopjra/wias');

let api = wias.authorize('apikey');
```

> Make sure to replace `apikey` with your API key.

WIAS uses API keys to allow access to the API. You can register a new WIAS API key at our [developer portal](https://devs.wias.kopjra.com/).

WIAS expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: apikey`

<aside class="notice">
You must replace <code>apikey</code> with your personal API key.
</aside>

# Interactions

The Interaction represents the minimum entity that can be forensically acquired, stored and retrieved. It represents a
specific action done by one of your users, plus the context (ie: the web page that he/she sees when he/she does that
action).

## Create a New Interaction

Before you can acquire an action done by one of your users (and its context), you need to create a new Interaction, and
redirect your user to the forward URL specifically provided by WIAS: you will find this URL (<code>fURL</code>) inside
the body of the response of this call.

<aside class="notice">
At the end of this call, remember to redirect your user to <code>fURL</code>!
</aside>

Since we implement none of your backend business logic, the user's request will eventually be re-submitted directly by
the user to your web server, so you must provide us with the <code>oURL</code> locating the endpoint where the user would
re-submit his request.

In addition, you need to provide us with some HTML that will be the context on which the user is going to do his action.
The user's action could be for instance a form submission (a <code>POST</code> or a <code>GET</code>) or the click on a
link (only a <code>GET</code>). In both cases, you have to pre-render the HTML on your server and give it to us, as well
as the indication about the HTTP Method of your user's action.

<aside class="notice">
Within your pre-rendered HTML, remember to replace the URL where you originally intended to handle the user action with
 the <code><%= e_url %></code> placeholder. You'd put that URL in the field <code>oURL</code> of the Create Interaction
 request.
</aside>

Finally, you can also decide to put any tag on your interaction.

<aside class="warning">
Future retrievals requests can only done by matching Interaction ID or custom tags! We won't be able to fetch the data
contained in your pre-rendered HTML or in the content submitted by your user's action.
</aside>


```shell
curl "https://gateway.wias.kopjra.com/api/interactions"
  -X DELETE
  -d '{"oURL": "http://someurl.com", "oMethod": "POST", "prerenderedHtml": "<!doctype html><html><head><title>Title</title></head><body><p><strong>E_URL: </strong><%= e_url %></p><p><strong>F_URL: </strong><%= f_url %></p><p>Sample text.</p><form action="<%= e_url %>" method="post"><fieldset><legend>Stuffity stuff:</legend><input type="radio" name="st1" value="true">Yep</input><input type="radio" name="st1" value="false">Nay</input></fieldset><fieldset><legend>Lalaland:</legend><input type="radio" name="st2" value="true">Great movie</input><input type="radio" name="st2" value="false">Bad movie</input></fieldset><input type="submit" value="Invia" /></form></body></html>", "tags": {"foo": "bar", "baz": "woo"}}'
  -H "Accept: application/json"
  -H "Content-Type: application/json"
  -H "Authorization: apikey"
```

```javascript
const wias = require('@kopjra/wias');

let api = wias.authorize('apikey');
let interaction = {
  "oURL": "http://someurl.com",
  "oMethod": "POST",
  "prerenderedHtml": "<!doctype html><html><head><title>Title</title></head><body><p><strong>E_URL: </strong><%= e_url %></p><p><strong>F_URL: </strong><%= f_url %></p><p>Sample text.</p><form action="<%= e_url %>" method="post"><fieldset><legend>Stuffity stuff:</legend><input type="radio" name="st1" value="true">Yep</input><input type="radio" name="st1" value="false">Nay</input></fieldset><fieldset><legend>Lalaland:</legend><input type="radio" name="st2" value="true">Great movie</input><input type="radio" name="st2" value="false">Bad movie</input></fieldset><input type="submit" value="Invia" /></form></body></html>",
  "tags": {
    "foo": "bar",
    "baz": "woo"
  }
};
let max = api.interactions.create();
```

> The above command returns JSON structured like this:

```json
{
  "id": 86,
  "fUrl" : "https://gateway.wias.kopjra.com/api/i/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpX2lkIjo4OSwiaWF0IjoxNTE5MjEwODc4LCJleHAiOjE1MTkyMTQ0Nzh9.QLoyvZPl2_KgxyupbjWJXESwOnkhzGDo4vr4i8GjYa8"
}
```

### HTTP Request

`POST https://gateway.wias.kopjra.com/api/interactions`

### JSON Body Keys

Parameter | Mandatory | Description
--------- | ----------- | -----------
oURL | Yes | The URL where the request was originally intended to go. At the end of the acquisition process, the user will re-submit the request to this URL. Your service should be ready to serve the user's request here.
oMethod | Yes | The type of request to acquire (and re-submit to your service). Only 'POST' and 'GET' supported.
prerenderedHtml | Yes | The pre-prendered HTML to be displayed by our web server. It must contain the string '<%= e_url %>' somewhere.
tags | No | An object representing the tags to be added to the Interaction. Note: tag names can only contain US-ASCII letters and/or numbers, whereas tag values can be any UTF-8 string.

## Get All Interactions

You can use this endpoint to get all of your Interactions. In the future, you will also be able to search among
Interactions using custom tags (still to be implemented).

```shell
curl "https://gateway.wias.kopjra.com/api/interactions"
  -H "Authorization: apikey"
```

```javascript
const wias = require('@kopjra/wias');

let api = wias.authorize('apikey');
let interactions = await api.interactions.get();
```

> The above command returns JSON structured like this:

```json
[
    {
        "id": 83,
        "oURL": "http://node-dumb-listening-server.herokuapp.com/",
        "hasPrerenderedHtml": true,
        "expirationPeriod": 3600,
        "expiresAt": "2018-02-14T14:25:57.000Z",
        "interactionSetId": 1,
        "currentStatus": "SUBSCRIBED",
        "pcapLocation": null,
        "pcapDigest": null,
        "keysLocation": null,
        "keysDigest": null,
        "activePodNodeId": null,
        "activePodPort": null,
        "createdAt": "2018-02-14T14:25:54.000Z",
        "updatedAt": "2018-02-14T14:25:54.000Z"
    },
    {
        "id": 85,
        "oURL": "http://node-dumb-listening-server.herokuapp.com/",
        "hasPrerenderedHtml": true,
        "expirationPeriod": 3600,
        "expiresAt": "2018-02-14T16:11:45.000Z",
        "interactionSetId": 1,
        "currentStatus": "CONCLUDED",
        "pcapLocation": "cws/uuid/a18e65b8-6feb-4dd1-a382-773b5c40f62c.pcap",
        "pcapDigest": "bda2ec6cf68ad6193c25b1cc038b2b0a2a077cba505b73c1181b9e5dc8611bdd",
        "keysLocation": "cws/uuid/a18e65b8-6feb-4dd1-a382-773b5c40f62c.log",
        "keysDigest": "751e28e65fe32a04a3c8d97f0a898db089c6ae247d7ee458d70df966c6908185",
        "activePodNodeId": null,
        "activePodPort": null,
        "createdAt": "2018-02-14T15:11:45.000Z",
        "updatedAt": "2018-02-14T15:12:11.000Z"
    }
]
```

This endpoint retrieves all of your interactions. Please note that tags are not shown here.

### HTTP Request

`GET https://gateway.wias.kopjra.com/api/interactions`


## Get a Specific Interaction

```shell
curl "https://gateway.wias.kopjra.com/api/interactions/85"
  -H "Accept: application/json"
  -H "Authorization: apikey"
```

```javascript
const wias = require('@kopjra/wias');

let api = wias.authorize('apikey');
let interaction = await api.interactions.get(85);
```

> The above command returns JSON structured like this:

```json
{
    "id": 85,
    "oURL": "http://node-dumb-listening-server.herokuapp.com/",
    "hasPrerenderedHtml": true,
    "expirationPeriod": 3600,
    "expiresAt": "2018-02-14T16:11:45.000Z",
    "interactionSetId": 1,
    "currentStatus": "CONCLUDED",
    "pcapLocation": "cws/uuid/a18e65b8-6feb-4dd1-a382-773b5c40f62c.pcap",
    "pcapDigest": "bda2ec6cf68ad6193c25b1cc038b2b0a2a077cba505b73c1181b9e5dc8611bdd",
    "keysLocation": "cws/uuid/a18e65b8-6feb-4dd1-a382-773b5c40f62c.log",
    "keysDigest": "751e28e65fe32a04a3c8d97f0a898db089c6ae247d7ee458d70df966c6908185",
    "activePodNodeId": null,
    "activePodPort": null,
    "createdAt": "2018-02-14T15:11:45.000Z",
    "updatedAt": "2018-02-14T15:12:11.000Z"
}
```

This endpoint retrieves a specific interaction.

### HTTP Request

`GET https://gateway.wias.kopjra.com/api/interactions/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the interaction to retrieve
