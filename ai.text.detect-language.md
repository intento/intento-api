# ai.text.detect-language

Detect intent from text.

<!-- TOC depthFrom:2 -->

- [Basic usage](#basic-usage)
- [Getting available providers](#getting-available-providers)
- [Getting information about a provider](#getting-information-about-a-provider)
- [Supported languages](#supported-languages)
    - [Getting list of supported languages](#getting-list-of-supported-languages)
    - [Getting full information on the supported language](#getting-full-information-on-the-supported-language)
- [Setting your own language codes](#setting-your-own-language-codes)

<!-- /TOC -->

## Basic usage

To detect language of a text, send a POST request to Intento API at [https://api.inten.to/ai/text/detect-language](https://api.inten.to/ai/text/detect-language). Specify the source text and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/detect-language' -d '{
    "context": {
        "text": "Hello, I would like to take a class at your University.",
    },
    "service": {
        "provider": "ai.text.detect-language.microsoft.text_analytics_api.2-1"
    }
}'
```

The response contains the detect-language results:

```json
{
    "results": [
        [
            {
                "language": "en",
                "confidence": 1
            }
        ]
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.detect-language.microsoft.text_analytics_api.2-1",
            "name": "Microsoft Text Analytics API"
        }
    }
}
```

The detect-language intent supports bulk requests:

```bash
curl -XPOST -H "Trace: true" -H "apikey: YOUR_API_KEY" "https://api.inten.to/ai/text/detect-language" -d '
{
  "context": {
    "text": ["Hallo Welt!", "Hello world"]
  },
  "service": {
    "provider" : "ai.text.detect-language.microsoft.text_analytics_api.2-1"
  }
}'
```

Each segment language is detected separately:

```json
{
    "results": [
        [
            {
                "language": "de",
                "confidence": 1.0
            }
        ],
        [
            {
                "language": "en",
                "confidence": 1.0
            }
        ]
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.detect-language.microsoft.text_analytics_api.2-1",
            "name": "Microsoft Text Analytics API"
        }
    }
}
```

## Getting available providers

To get a list of available Dictionary providers, send a GET request to [https://api.inten.to/ai/text/detect-language](https://api.inten.to/ai/text/detect-language).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/detect-language'
```

The response contains a list of the providers available for given constraints with an information on using custom models, etc.:

```json
[
    {
        "id": "ai.text.detect-language.microsoft.text_analytics_api.2-1",
        "vendor": "Microsoft",
        "description": "Text Analytics API",
        "production": true,
        "integrated": true,
        "billable": true,
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    }
]
```

More on [provider flags and capabilities](./providers).

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/text/detect-language/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/detect-language/ai.text.detect-language.microsoft.text_analytics_api.2-1'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.text.detect-language.microsoft.text_analytics_api.2-1",
    "vendor": "Microsoft",
    "description": "Text Analytics API",
    "logo": "https://inten.to/static/img/api/mcs_translate.png",
    "billing": true,
    ...
}
```

## Supported languages

### Getting list of supported languages

Will return an array of supported languages, for each language:

- iso name
- localized name (if `locale` parameter is provided); if there is no localized name, `null` is returned
- intento code
- client code (if the client calling the method has its own codes)

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/detect-language/languages?locale=ru'
```

Response:

```json
[
    {
        "iso_name": "Hebrew (modern)",
        "name": "иврит",
        "intento_code": "he",
        "client_code": "hebr"
    }
]
```

### Getting full information on the supported language

For a given language code (intento internal or client’s) will show full metadata:

- iso name
- localized name (if `locale` parameter is provided); if there is no localized name, `null` is returned
- intento code
- iso codes (ones which are applicable)
- providers’ codes (which map to this internal code)
- client code (if the client calling the method has its own codes)

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/detect-language/languages/he?locale=ru'
```

Response:

```json
{
    "iso_name": "Hebrew (modern)",
    "name": "иврит",
    "intento_code": "he",
    "iso_639_1_code": "he",
    "iso_639_2t_code": "heb",
    "iso_639_2b_code": "heb",
    "iso_639_3_code": "heb",
    "provider_codes": {},
    "client_code": "hebr"
}
```

## Setting your own language codes

To define your aliases to language codes, send a POST request to Intento API at [https://api.inten.to/settings/languages](https://api.inten.to/settings/languages). After 60 seconds, you can start using them.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/settings/languages' --data '{"aliasforen":"en"}'
```

Response:

```json
{
    "aliasforen": "en"
}
```

Settings can be retrieved using the GET request

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/settings/languages'
```

Response:

```json
{
    "aliasforen": "en"
}
```
