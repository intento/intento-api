# ai.text.dictionary

This is an intent to get meanings of text in selected language.

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

To dictionary a text, send a POST request to Intento API at [https://api.inten.to/ai/text/dictionary](https://api.inten.to/ai/text/dictionary). Specify the source text, source and target languages and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/dictionary' -d '{
    "context": {
        "text": "kick",
        "from": "en",
        "to": "ru"
    },
    "service": {
        "provider": "ai.text.dictionary.yandex.dictionary_api.1-0"
    }
}'
```

The response contains the dictionary results grouped by part of speech and a service information. Parts of speech are formatted in snake_case style:

```json
{
    "results": [
        {
            "noun": [
                "пинок"
            ],
            "verb": [
                "пинать"
            ]
        }
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.dictionary.yandex.dictionary_api.1-0",
            "name": "Yandex Dictionary API"
        }
    }
}
```

The response can contain an empty json key if some dictionary results do not contain part of speech:

```json
{
    "results": [
        {
            "": [
                "Apple"
            ],
            "noun": [
                "яблоко"
            ],
            "adjective": [
                "яблочный"
            ]
        }
    ],
    "meta": {
    },
    "service": {
        "provider": {
            "id": "ai.text.dictionary.yandex.dictionary_api.1-0",
            "name": "Yandex Dictionary API"
        }
    }
}
```

If the provider doesn't have capabilities (e.g. language pairs) to process request, 413 error will be returned:

```json
{
    "error": {
        "code": 413,
        "message": "Provider ai.text.dictionary.yandex.dictionary_api.1-0 constraint(s) violated: from (Source language)"
    }
}
```

## Getting available providers

To get a list of available Dictionary providers, send a GET request to [https://api.inten.to/ai/text/dictionary](https://api.inten.to/ai/text/dictionary).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/dictionary'
```

The response contains a list of the providers available for given constraints with an information on using custom models, etc.:

```json
[
    {
        "id": "ai.text.dictionary.yandex.dictionary_api.1-0",
        "vendor": "Yandex",
        "description": "Dictionary API",
        "production": true,
        "integrated": true,
        "billable": true,
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    }
]
```

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/dictionary?from=en&to=es'
```

Response:

```json
[
    {
        "id": "ai.text.dictionary.yandex.dictionary_api.1-0",
        "name": "Yandex dictionary API",
        "symmetric": [...],
        "pairs": [...]
    }
]
```

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/text/dictionary/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/dictionary/ai.text.dictionary.yandex.dictionary_api.1-0'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.text.dictionary.yandex.dictionary_api.1-0",
    "name": "Yandex",
    "logo": "https://inten.to/static/img/api/ynd_dictionary.png",
    "billing": true,
    "languages": {
        "symmetric": [],
        "pairs": [
            {
                "from": "be",
                "to": "be"
            },
            {
                "from": "be",
                "to": "ru"
            },
            {
                "from": "bg",
                "to": "ru"
            },
            {
                "from": "cs",
                "to": "en"
            },
            {
                "from": "cs",
                "to": "ru"
            },
            {
                "from": "da",
                "to": "en"
            },
            ...
        ]
    }
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/dictionary/languages?locale=ru'
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/dictionary/languages/he?locale=ru'
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
