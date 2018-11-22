# ai.text.classify

Converts text in one language from one script to another script.

<!-- TOC depthFrom:2 -->

- [Basic usage](#basic-usage)
- [Getting available providers](#getting-available-providers)
    - [Filtering providers by capabilities](#filtering-providers-by-capabilities)
- [Getting information about a provider](#getting-information-about-a-provider)
- [Supported languages](#supported-languages)
    - [Getting list of supported languages](#getting-list-of-supported-languages)
    - [Getting full information on the supported language](#getting-full-information-on-the-supported-language)
- [Setting your own language codes](#setting-your-own-language-codes)

<!-- /TOC -->

## Basic usage

To classify a text, send a POST request to Intento API at [https://api.inten.to/ai/text/classify](https://api.inten.to/ai/text/classify). Specify the source text, source languages and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/classify' -d '{
    "context": {
        "text": "I want to book a flight ticket to Moscow",
        "lang": "en",
        "format": "text"
    },
    "service": {
        "provider": "ai.text.classify.ibm.natural_language_understanding"
    }
}'
```

The response contains the classify results grouped by part of speech and a service information. Parts of speech are formatted in snake_case style:

```json
{
    "results": "...",
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.classify.ibm.natural_language_understanding"
        }
    }
}
```

If the provider doesn't have capabilities (e.g. language) to process request, 413 error will be returned:

```json
{
  "error": {
    "code": 413,
    "message": "Provider ai.text.classify.microsoft.translator_text_api.3-0 constraint(s) violated: fromscript (Specifies the script used by the input text.), language (Specifies the language of the text to convert from one script to another.), toscript (Specifies the output script.)"
  },
  "request_id": "..."
}
```

## Getting available providers

To get a list of available Dictionary providers, send a GET request to [https://api.inten.to/ai/text/classify](https://api.inten.to/ai/text/classify).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/classify'
```

The response contains a list of the providers available for given constraints:

```json
[
    {
        "id": "ai.text.classify.twinword.text_classification",
        "name": "Twinword Text Classification",
        "vendor": "Twinword",
        "description": "Text Classification",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    }
]
```

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/classify?lang=ko'
```

Response:

```json
[
    {
        "id": "ai.text.classify.ibm.natural_language_understanding",
        "name": "IBM Watson Natural Language Understanding categories",
        "description": "Natural Language Understanding categories",
        "integrated": false,
        "own_auth": true,
        "stock_model": false,
        "custom_model": false,
        "lang": ["en", "ar", "fr", "de", "it", "ko", "pt", "es", "ja"]
    },
    ...
]
```

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/text/classify/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/classify/ai.text.classify.tencent.text_classification_api'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.text.classify.tencent.text_classification_api",
    "vendor": "Tencent",
    "logo": "https://inten.to/static/img/api/tmt_translate.png",
    "billing": true,
    "description": "Text Classification API",
    "production": false,
    "integrated": false,
    "billable": true,
    "own_auth": true,
    "stock_model": true,
    "custom_model": false,
    "languages": { ... }
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/classify/languages?locale=ru'
```

Response:

```json
[
    {
        "iso_name": "Hebrew (modern)",
        "name": "иврит",
        "intento_code": "he"
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/classify/languages/he?locale=ru'
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
    "provider_codes": {}
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
