---
title: ProAvalon Bot API Specification

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript
  - python
  - shell

toc_footers:
  - "<a href='https://www.proavalon.com/'>Play The Resistance: Avalon online</a>"
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - api

search: true
---

# Introduction

Welcome to the ProAvalon Bot API Specification. Users wishing to write bots to play on ProAvalon.com should implement this API.

This specification is a work in progress. This is version `v0`.

# Getting Started

> Requests will be sent to your API as follows:

```python
import requests
import json

HEADERS = {
  'Authorization': '<Your shared API key>',
  'Content-Type': 'application/json'
}

CONTENT = {} # Your json payload goes here.

r = requests.post( # Requests are made with either POST or GET.
  'https://<your api>/v0/<your endpoint>',
  data=json.dumps(CONTENT),
  headers=HEADERS
)

result = r.json() # ProAvalon expects a properly formatted JSON response.
```

```shell
curl "https://<your api>/v0/<your endpoint>" \
  -H "Authorization: <Your shared API key>" \
  -H "Content-Type: application/json" \
  -X POST \ # Requests are made with either POST or GET.
  -d "{}" # A properly formatted JSON payload sent by ProAvalon.com
```

```javascript
const axios = require('axios');

axios.request({ 
  method: 'post', // Requests are made with either POST or GET
  url: 'https://<your api>/v0/<your endpoint>',
  headers: {
    'Authorization': '<Your shared API key>',
    'Content-Type': 'application/json'
  }, 
  data: {}, // A JSON payload sent by ProAvalon.com
}).then(function (response) {
  // Your API's response is available as response.data
}).catch(function (error) {
  // Your API returned an error.
});
```

### API Format

ProAvalon will communicate with your bot via a JSON REST API. All request payloads will contain a JSON object. Your API should return a properly-formatted JSON response.

### Authentication

All requests made to a ProAvalon bot will be secured with an `Authorization` header. Your API should share an API key with ProAvalon.com to be sent on every API request. This will ensure only ProAvalon.com can make requests, preventing denial-of-service attacks.

<aside class="notice">
Ensure that your API always returns properly formatted JSON. Make sure to include the proper <code>Content-Type: application/json</code> header.
</aside>
