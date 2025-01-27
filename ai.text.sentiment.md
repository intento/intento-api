# ai.text.sentiment

This is an intent to analyze the sentiment of the provided text.

<!-- TOC depthFrom:2 -->

- [Basic usage](#basic-usage)
    - [Bulk mode](#bulk-mode)
    - [:lock: Multi mode](#lock-multi-mode)
- [Getting available providers](#getting-available-providers)
    - [Filtering providers by capabilities](#filtering-providers-by-capabilities)
- [Getting information about a provider](#getting-information-about-a-provider)
    - [:lock: Enrichment mode](#lock-enrichment-mode)
- [:lock: Domain support](#lock-domain-support)
- [:lock: Aspect-based Sentiment Analysis](#lock-aspect-based-sentiment-analysis)
- [:lock: Custom Sentiment Models](#lock-custom-sentiment-models)
    - [:lock: Migrating custom models between providers](#lock-migrating-custom-models-between-providers)

<!-- /TOC -->

## Basic usage

To analyze a text for sentiments, send a POST request to Intento API at [https://api.inten.to/ai/text/sentiment](https://api.inten.to/ai/text/sentiment). Specify the text to analyze, the language of the text, and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/sentiment' -d '{
    "context": {
        "text": "We love this trail and make the trip every year. The views are breathtaking and well worth the hike!",
        "lang": "en"
    },
    "service": {
        "provider": "ai.text.sentiment.meaningcloud.sentiment_analysis_api.2-1"
    }
}'
```

The response contains the processed text and a service information:

```json
{
    "results": [
        {
            "sentiment_label": "positive",
            "sentiment_score": 1.0,
            "sentiment_confidence": 1.0,
            "sentiment_subjectivity": "subjective",
            "agreement": true,
            "irony": false
        }
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.sentiment.meaningcloud.sentiment_analysis_api.2-1",
            "name": "MeaningCloud Sentiment Analysis API"
        }
    }
}
```

If the provider doesn't have capabilities (e.g. does not support a specific language) to process request, 413 error will be returned:

```json
{
    "error": {
        "code": 413,
        "message": "Provider ai.text.sentiment.meaningcloud.sentiment_analysis_api.2-1 constraint(s) violated: lang (Source language)"
    }
}
```

### Bulk mode

We provide a bulk fulfillment mode to process an array of texts at once. The mode is activated by sending an array of strings to the `context.text` parameter. The bulk mode is supported for some of the providers (you can check the capability called "bulk").

```sh
curl -XPOST -H "apikey: YOUR_API_KEY" "https://api.inten.to/ai/text/sentiment" -d '{
    "context": {
        "text": [
            "We love this shop!",
            "The quality is not as good as it should"
        ],
        "lang": "en"
    },
    "service": {
        "provider": "ai.text.sentiment.ibm.natural_language_understanding"
    }
}'
```

### :lock: Multi mode
The response contains the processed texts and a service information:

```json
{
    "results": [
        {
            "sentiment_label": "positive",
            "sentiment_score": 0.931392
        },
        {
            "sentiment_label": "neutral",
            "sentiment_score": 0.535453
        }
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.sentiment.ibm.natural_language_understanding",
            "name": "IBM Watson Natural Language Understanding"
        }
    }
}
```

In the multi mode, the processing of the text is performed using a list of providers. The mode is activated by passing an array of provider identificators.

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/sentiment' -d '{
    "context": {
        "text": "We love this trail and make the trip every year. The views are breathtaking and well worth the hike!",
        "lang": "en"
    },
    "service": {
        "provider": [
            "ai.text.sentiment.ibm.natural_language_understanding",
            "ai.text.sentiment.aylien.text_analysis_api.1-0"
        ]
    }
}'
```

The response contains the analyzed text and service information:          ↑

```json
[
    {
        "results": [
            {
                "sentiment_label": "positive",
                "sentiment_score": 0.931392
            }
        ],
        "meta": {},
        "service": {
            "provider": {
                "id": "ai.text.sentiment.ibm.natural_language_understanding",
                "name": "IBM Watson Natural Language Understanding"
            }
        }
    },
    {
        "results": [
            {
                "sentiment_label": "positive",
                "sentiment_score": 1.0,
                "sentiment_confidence": 0.9975973963737488,
                "sentiment_subjectivity": "unknown",
                "subjectivity_confidence": 0.0
            }
        ],
        "meta": {},
        "service": {
            "provider": {
                "id": "ai.text.sentiment.aylien.text_analysis_api.1-0",
                "name": "AYLIEN Text Analysis API"
            }
        }
    }
]
```

## Getting available providers

To get a list of available Sentiment providers, send a GET request to [https://api.inten.to/ai/text/sentiment](https://api.inten.to/ai/text/sentiment).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/sentiment'
```

The response contains a list of the providers available for given constraints with an information on using custom models, etc.:

```json
[
    {
        "id": "ai.text.sentiment.ibm.natural_language_understanding",
        "name": "IBM Watson Natural Language Understanding",
        "own_auth": true,
        "stock_model": true,
        "custom_model": true,
        "lang": [
            "en"
        ]
    },
    {
        "id": "ai.text.sentiment.microsoft.text_analytics_api.2-0",
        "name": "Microsoft Text Analytics API",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false,
        "lang": [
            "en"
        ]
    }
]
```

More on [provider flags and capabilities](providers.md).

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/sentiment?lang=en'
```

Response:

```json
[
    {
        "id": "ai.text.sentiment.ibm.natural_language_understanding",
        "name": "IBM Watson Natural Language Understanding",
        "own_auth": true,
        "stock_model": true,
        "custom_model": true
    },
    {
        "id": "ai.text.sentiment.microsoft.text_analytics_api.2-0",
        "name": "Microsoft Text Analytics API",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    }
]
```

Besides source language, service providers may be filtered by support of specific bulk option (`bulk`) and language detection option (`lang_detect`):

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/sentiment?lang=en&lang_detect=true&bulk=true'
```

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/text/sentiment/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/sentiment/ai.text.sentiment.microsoft.text_analytics_api.2-0'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.text.sentiment.microsoft.text_analytics_api.2-0",
    "name": "Microsoft",
    "logo": "https://inten.to/static/img/api/mcs_translate.png",
    "billing": true,
    "languages": {
        "lang": [
            "en"
        ]
    },
    "lang_detect": false,
    "bulk": false
}
```

### :lock: Enrichment mode

In the enrichment mode, the Intento API tries to obtain as much information from the text as possible. Some providers do perform subjectivity analysis, others can detect irony, others support aspect-based sentiments. By aggregating results from several providers Intento can provide more information in a single call to the Intento API.

TBD

## :lock: Domain support

Some providers support special domains like retail, hospitality, electronics, telecom, automotive, etc.

TBD

## :lock: Aspect-based Sentiment Analysis

Some providers support Aspect-Based Sentiment Analysis, that makes it easier to identify and determine the sentiment towards specific aspects in text, i.e. "hotels", "restaurants", "cars", "airlines".

TBD

## :lock: Custom Sentiment Models

Some services allow for fine-tuning of the sentiment models. Using the Intento API you may have an access to the models you've already trained in the past or train new ones based on the data you have.

TBD

### :lock: Migrating custom models between providers

TBD
