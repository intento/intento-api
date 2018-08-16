# Usage API

This is an intent to get usage statistics about all calls to the API.

<!-- TOC depthFrom:2 -->

- [Basic requests](#basic-requests)
    - [Simplest request](#simplest-request)
    - [More advanced request](#more-advanced-request)
    - [How to specify date/time range](#how-to-specify-datetime-range)
- [More detailed description](#more-detailed-description)
    - [Errors](#errors)
- [Provider's statistics](#providers-statistics)
- [Filtering](#filtering)
    - [Filtering by providers](#filtering-by-providers)
    - [Filtering by intents](#filtering-by-intents)
    - [Filtering by response status](#filtering-by-response-status)
- [Getting possible values for filter parameters](#getting-possible-values-for-filter-parameters)

<!-- /TOC -->

## Basic requests

### Simplest request

To get usage statistics for the API, send a POST request to Usage API at [https://api.inten.to/usage/intento](https://api.inten.to/usage/intento) with an empty body:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento'
```

This request return information for the last 12 days. Each bucket for a single day, last bucket for today.
The response contains the list of buckets with statistics:

```json
{
    "data":[
        {
            "timestamp": 1529280000,
            "metrics": {
                "requests": 100,
                "items": 100,
                "len": 3000,
                "errors": 10
             }
        },
        ...
    ]
}
```

### More advanced request

This request return 1-hour information for last 15 hours:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d {
    "range": {
        "bucket": "1h",
        "items": 15
    }
}'
```

### How to specify date/time range

This example shows how to specify a particular date-time range: 10 items from particular date-time (in seconds, UTC timezone, check out [timezone converter](https://www.epochconverter.com/)).

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d {
    "range": {
        "items": 10,
        "from": 1529884800,
        "bucket": "1day"
    }
}'
```

## More detailed description

range field:

- items: number of buckets in result. All buckets are sorted by timestamp asc. Default value: 12
- to: timestamp of last bucket. Actual timestamp may differ because it aligned by bucket field. Default value: now
- from: timestamp of first bucket. Actual timestamp may differ because it aligned by bucket field.
- bucket: width of each bucket. Default value: "1d"

You may specify:

- `from` and `to` fields (`to` must be greater than `from`)
- `from` and `items` fields
- `to` and `items` fields

Values in `from` and `to` fields are in seconds, UTC timezone (check out [timezone converter](https://www.epochconverter.com/)). Value aligned in accordance with the field bucket.

Possible values of the bucket field:

- `1min`, `2mins`, `5mins`, `10mins`, `15mins`, `30mins`
- `1hour`, `2hours`, `3hours`, `6hours`, `12hours`
- `1day`

A `metrics` block of the response contains following fields:

- `requests` - number of requests
- `items` - number of data items processed (not equal to `requests` if batch requests are used)
- `len` - size of items processed
- `errors` - number of failed responses

### Errors

Error responses usually include a JSON document in the response body, which contains information about the error.

Error codes:

- `400` -- Invalid json structure
- `401` -- Intento: Auth key is missing
- `403` -- Intento: Auth key is invalid
- `500` -- Intento: Internal error

## Provider's statistics

Intento API receives requests from the customer and sends requests to the provider. You may get statistics of requests to providers as well. End point needs to be changed to [https://api.inten.to/usage/provider](https://api.inten.to/usage/provider) and you may specify particular provider or intent:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/provider' -d {
    "range": {
        "from": 1529280000,
        "to": 1529884800,
        "bucket": "1day"
    },
    "filter": {
        "provider": [
            "ai.text.translate.microsoft.translator_text_api.2-0"
        ]
    }
}'
```

## Filtering

Usage statistics can be filtered by provider, intent, client or error status.
To do this one can add `filter` object in a request. Valid `filter` keys are:

- `provider` - by provider like "ai.text.translate.microsoft.translator_text_api.2-0" or "ai.text.translate.google.translate_api.2-0"
- `intent` - by used intents like "ai.text.translate" or "ai.text.sentiment"
- `client` - for multi-key accounts to choose particular account; if no parameter present atatistics provided for all accounts together.
- `status` - by http response codes like "200" or "4xx" (intended for error monitoring)

Each value can be omitted. If specified it should be a string or a list of strings.
Detailed examples below.

### Filtering by providers

To get statistics for a particular provider or for a list of some providers, specify the filter block in your request:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d {
    "range": {
        "bucket": "1hour"
    },
    "filter": {
        "provider": [
            "ai.text.translate.microsoft.translator_text_api.2-0"
        ]
    }
}'
```

### Filtering by intents

To get statistics for a particular provider or for a list of some providers, specify the filter block in your request:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d {
    "range": {
        "from": 1529280000,
        "to": 1529884800,
        "bucket": "1day"
    },
    "filter": {
        "intent": "ai.text.translate"
    }
}'
```

### Filtering by response status

For filtering segment by response status, specify 'status' key in request`s filter block.
Possible value of a 'status' filter:

- exact value of a status like `200` or `403`
- enumerate value like `3xx` or `4xx` or `5xx` combining errors of the same type
- any combination of previous options

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d '{
    "range": {
        "from": 1529280000,
        "to": 1529884800,
        "bucket": "1day"
    },
    "filter": {
        "status": ["200", "3xx", "4xx", "5xx", "403"]
    }
}'
```

## Getting possible values for filter parameters

To find out what intents or providers were used at least once following API endpoit can be used:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/distinct' -d '{
    "range": {
        "from": 1529280000,
        "to": 1529884800
    },
    "fields": [
        "provider",
        "intent"
    ]
}'
```

The response contains all possible values of mentioned fields during the period specified in `range`:

```json
{
    "fields":{
        "intent":[
            "xxx",
            "yyy",
            "zzz"
        ],
        "provider":[
            "aaa",
            "bbb",
            "ccc"
        ]
    }
}
```
