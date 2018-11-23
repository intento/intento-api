# ai.speech.transcribe

Text content classification.

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

To transcribe an audio recording, send a POST request to Intento API at [https://api.inten.to/ai/speech/transcribe](https://api.inten.to/ai/speech/transcribe). Specify the source, source language and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/speech/transcribe' -d '{
    "context": {
        "source": "...",
        "language": "en"
    },
    "service": {
        "provider": "ai.speech.transcribe.alibaba.asr_api"
    }
}'
```

The response contains the speech transcribe results:

```json
{
    "results": "...",
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.speech.transcribe.alibaba.asr_api"
        }
    }
}
```

If the provider doesn't have capabilities (e.g. language) to process request, 413 error will be returned:

```json
{
  "error": {
    "code": 413,
    "message": "Provider ai.speech.transcribe.alibaba.asr_api constraint(s) violated."
  },
  "request_id": "..."
}
```

## Getting available providers

To get a list of available providers, send a GET request to [https://api.inten.to/ai/speech/transcribe](https://api.inten.to/ai/speech/transcribe).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/speech/transcribe'
```

The response contains a list of the providers available for given constraints:

```json
[
    {
        "id": "ai.speech.transcribe.ibm.watson_speech_to_text_api",
        "api_id": "ibmwatsonspeechtotext",
        "name": "IBM Watson",
        "description": "Speech to Text",
        "own_auth": true,
    }
]
```

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/speech/transcribe?language=ru'
```

Response:

```json
[
    {
        "id": "ai.speech.transcribe.alibaba.asr_api",
        "name": "Alibaba Cloud",
        "description": "ASR",
        "integrated": false,
        "own_auth": true,
        "stock_model": false,
        "custom_model": false,
        "lang": ["en", "ar", "fr", "ru"]
    },
    ...
]
```

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/speech/transcribe/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/speech/transcribe/ai.speech.transcribe.tencent.asr_api'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.speech.transcribe.tencent.asr_api",
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/speech/transcribe/languages?locale=ru'
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
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/speech/transcribe/languages/he?locale=ru'
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
