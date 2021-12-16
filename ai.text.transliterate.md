# ai.text.transliterate

Converts a text from one script to another script (e.g. katakana to latin)

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

To transliterate a text, send a POST request to Intento API at [https://api.inten.to/ai/text/transliterate](https://api.inten.to/ai/text/transliterate). Specify the source text, source languages and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/transliterate' -d '{
    "context": {
        "text": "こんにちは",
        "language": "ja",
        "fromscript": "jpan",
        "toscript": "latn"
    },
    "service": {
        "provider": "ai.text.transliterate.microsoft.translator_text_api.3-0"
    }
}'
```

The response contains the transliterate results grouped by part of speech and a service information. Parts of speech are formatted in snake_case style:

```json
{
    "results": [
        "Kon'nichiwa"
    ],
    "meta": {
        "timing": {
            "total": 0.246434,
            "providers": 0.201087
        },
        "client_key_label": "production"
    },
    "service": {
        "provider": {
            "id": "ai.text.transliterate.microsoft.translator_text_api.3-0",
            "name": "Microsoft Translator API v3.0",
            "timing": {
                "provider": 0.201087
            }
        }
    }
}
```

Transliterating multiple segments with single request:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/transliterate' -d '{
    "context": {
        "text": ["こんにちは", "文字変換の例"],
        "language": "ja",
        "fromscript": "jpan",
        "toscript": "latn"
    },
    "service": {
        "provider": "ai.text.transliterate.microsoft.translator_text_api.3-0"
    }
}'
```

If the provider doesn't have capabilities (e.g. language) to process request, 413 error will be returned:

```json
{
  "error": {
    "code": 413,
    "message": "Provider ai.text.transliterate.microsoft.translator_text_api.3-0 constraint(s) violated: fromscript (Specifies the script used by the input text.), language (Specifies the language of the text to convert from one script to another.), toscript (Specifies the output script.)"
  },
  "request_id": "..."
}
```

## Getting available providers

To get a list of available Dictionary providers, send a GET request to [https://api.inten.to/ai/text/transliterate](https://api.inten.to/ai/text/transliterate).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/transliterate'
```

The response contains a list of the providers available for given constraints with an information on using custom models, etc.:

```json
[
    {
        "production": true,
        "integrated": true,
        "billable": true,
        "own_auth": true,
        "stock_model": true,
        "custom_model": false,
        "delegated_credentials": null,
        "async_only": false,
        "id": "ai.text.transliterate.microsoft.translator_text_api.3-0",
        "name": "Microsoft Translator API v3.0",
        "vendor": "Microsoft",
        "api_id": "microsofttranslator30",
        "picture": "img/api/mcs_translate.png",
        "type": "instance",
        "description": "Translator API v3.0",
        "language": [
            "ar",
            ...
        ],
        "fromscript": [
            "latn",
            ...
        ],
        "toscript": [
            "arab",
            ...
        ]
    }
]
```

More on [provider flags and capabilities](providers.md).

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/transliterate?fromscript=arab'
```

Response:

```json
[
    {
        "production": true,
        "integrated": true,
        "billable": true,
        "own_auth": true,
        "stock_model": true,
        "custom_model": false,
        "delegated_credentials": null,
        "async_only": false,
        "id": "ai.text.transliterate.microsoft.translator_text_api.3-0",
        "name": "Microsoft Translator API v3.0",
        "vendor": "Microsoft",
        "api_id": "microsofttranslator30",
        "picture": "img/api/mcs_translate.png",
        "type": "instance",
        "description": "Translator API v3.0"
    }
]
```

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/text/transliterate/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/transliterate/ai.text.transliterate.microsoft.translator_text_api.3-0'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.text.transliterate.microsoft.translator_text_api.3-0",
    "vendor": "Microsoft",
    "logo": "https://inten.to/img/api/mcs_translate.png",
    "billing": true,
    "description": "Translator API v3.0",
    "production": true,
    "integrated": true,
    "billable": true,
    "own_auth": true,
    "stock_model": true,
    "custom_model": false,
    "delegated_credentials": null,
    "async_only": false,
    "auth": {
        "key": "YOUR_KEY"
    },
    "languages": {
        "language": [
            "ar",
            ...
        ],
        "fromscript": [
            "latn",
            ...
        ],
        "toscript": [
            "arab",
            ...
        ]
    },
    "bulk": true
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/transliterate/languages?locale=ru'
```

Response:

```
[
    {
        "iso_name": "Hebrew",
        "intento_code": "he",
        "direction": "right-to-left",
        "name": "иврит"
    },
    ...
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/transliterate/languages/he?locale=ru'
```

Response:

```json
{
    "iso_name": "Hebrew",
    "intento_code": "he",
    "iso_639_1_code": "he",
    "iso_639_2t_code": "heb",
    "iso_639_2b_code": "heb",
    "iso_639_3_code": "heb",
    "provider_codes": {},
    "name": "иврит"
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
