# Usage API

The `/usage` endpoint intended to get usage statistics about all calls to the API.

<!-- TOC depthFrom:2 -->

- [Basic requests](#basic-requests)
    - [Simplest request](#simplest-request)
    - [Change default request parameters](#change-default-request-parameters)
    - [Specify a date/time range](#specify-a-datetime-range)
- [Advanced info](#advanced-info)
    - [Everything about the `range` request field](#everything-about-the-range-request-field)
    - [Everything about the `metrics` field in a response](#everything-about-the-metrics-field-in-a-response)
    - [Errors](#errors)
- [Provider's statistics (`/usage/provider`)](#providers-statistics-usageprovider)
- [Filtering (`/usage/intento`)](#filtering-usageintento)
    - [Filtering by providers](#filtering-by-providers)
    - [Filtering by intents](#filtering-by-intents)
    - [Filtering by response status](#filtering-by-response-status)
    - [Filtering by language pair](#filtering-by-language-pair)
- [Grouping by features (`/usage/intento`)](#grouping-by-features-usageintento)
- [Getting possible values for filter parameters (`/usage/distinct`)](#getting-possible-values-for-filter-parameters-usagedistinct)

<!-- /TOC -->

## Basic requests

### Simplest request

To get usage statistics for the API, send a POST request to Usage API at [https://api.inten.to/usage/intento](https://api.inten.to/usage/intento) with an empty body:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento'
```

This request returns information for the last 12 days. Each bucket corresponds to a single day; the last bucket contains information on today usage.
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
                "words": 2,
                "errors": 10
             }
        },
        ...
    ]
}
```

### Change default request parameters

This request returns per hour information for the last 15 hours:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d {
    "range": {
        "bucket": "1h",
        "items": 15
    }
}'
```

### Specify a date/time range

This example shows how to set a particular date-time range: 10 items from a specific moment (datetime stamp in seconds, UTC timezone, check out [timezone converter](https://www.epochconverter.com/)).

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d {
    "range": {
        "items": 10,
        "from": 1529884800,
        "bucket": "1day"
    }
}'
```

## Advanced info

### Everything about the `range` request field

This field contains the following fields:

- `items`: number of buckets in the result. All buckets are sorted by timestamp, ascending. The default value is `12`.
- `to`: timestamp of the last bucket. An actual timestamp may differ because it aligned by bucket field. Default value: now
- `from`: timestamp of the first bucket. An actual timestamp may differ because it aligned by bucket field.
- `bucket`: width of each bucket. Default value: "1d"

You may specify:

- `from` and `to` fields (`to` must be greater than `from`)
- `from` and `items` fields
- `to` and `items` fields

Values in the `from` and `to` fields are in seconds, UTC timezone (check out [timezone converter](https://www.epochconverter.com/)). Values aligned by the field bucket.

Possible values of the `bucket` field:

- `1min`, `2mins`, `5mins`, `10mins`, `15mins`, `30mins`
- `1hour`, `2hours`, `3hours`, `6hours`, `12hours`
- `1day`

### Everything about the `metrics` field in a response

This request block contains the following fields:

- `requests` - number of requests
- `items` - number of data items processed (not equal to `requests` if batch requests are used)
- `len` - size of items processed
- `words` - number of words processed (for text intents)
- `errors` - number of failed responses

### Errors

Error responses usually include a JSON document in the response body, which contains some information about the error.

Error codes:

- `400` -- Invalid JSON structure
- `401` -- Intento: Auth key is missing
- `403` -- Intento: Auth key is invalid
- `500` -- Intento: Internal error

## Provider's statistics (`/usage/provider`)

Intento API receives requests from the customer and sends requests to the provider. You may get statistics of requests to providers as well. Endpoint needs to be changed to [https://api.inten.to/usage/provider](https://api.inten.to/usage/provider) and you may specify a particular provider or intent:

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

## Filtering (`/usage/intento`)

Usage statistics can be filtered by provider, intent, client or error status.
To do this one can add a `filter` object in a request. Valid `filter` keys are:

- `provider` - by provider like `ai.text.translate.microsoft.translator_text_api.2-0` or `ai.text.translate.google.translate_api.2-0`
- `intent` - by used intents like `ai.text.translate` or `ai.text.sentiment`
- `client` - (for multi-key accounts) to choose a particular account; if no parameter present, the statistics is provided for all keys together.
- `status` - by HTTP response codes like `200` or `4xx` (intended for error monitoring)
- `lang_pair` - by language pair like `en/es` or `zh/fr`

Each value can be omitted. If specified, it should be a string or a list of strings.
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

To get statistics for a particular intent or for a list of some intents, specify the filter block in your request:

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

For filtering segments by a response status, specify the `status` key in the request's `filter` block.
The `status` value should be a list of strings. Valid string values are:

- an exact value of a status like `200` or `403`
- enumerated value like `3xx` or `4xx` or `5xx` combining errors of the same type

The `status` list may contain any combination of exact status values and enumerated status values.

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

### Filtering by language pair

To get statistics for a particular language pair or for a list of some language pairs, specify the filter block in your request:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d {
    "range": {
        "from": 1529280000,
        "to": 1529884800,
        "bucket": "1day"
    },
    "filter": {
        "lang_pair": "en/zh"
    }
}'
```

## Grouping by features (`/usage/intento`)

One can group usage statistics by providers, intents or language pairs.
To do this one can add a `group` parameter to a request. The `group` value should be a list of strings. Valid string values are:

- `provider`
- `intent`
- `lang_pair`

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/usage/intento' -d '{
    "range": {
        "from": 1529280000,
        "to": 1529884800,
        "bucket": "1day"
    },
    "group": [
        "provider",
        "intent"
    ]
}'
```

The response contains the list of buckets with statistics. Each `bucket` describes metrics binded to a specific `timestamp`, every number in `metrics` is an aggregated value for a certain set of features, mentioned in the `group` parameter of the each element of the response:

```json
{
    "data":[
        {
            "timestamp": 1529884800,
            "group": {
                "intent": "ai.text.translate",
                "provider": "ai.text.translate.microsoft.translator_text_api.2-0"
            },
            "metrics": {
                "requests": 100,
                "items": 100,
                "len": 3000,
                "errors": 10,
                "words": 2
             }
        },
        ...
        {
            "timestamp": 1529280000,
            "group": {
                "intent": "ai.text.translate",
                "provider": "ai.text.translate.amazon.translate"
            },
            "metrics": {
                ...
            }
        },
        ...
    ]
}
```

## Getting possible values for filter parameters (`/usage/distinct`)

This API endpoint helps to find out what intents or providers were used at least once during specified (in `range`) period of time:

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

The `fields` value should be a list of strings. Valid string values are:

- `provider`
- `intent`
- `lang_pair`

The response contains all possible values of mentioned fields during the period specified in `range`:

```json
{
    "status": "OK",
    "data":{
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
