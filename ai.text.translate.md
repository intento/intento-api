# ai.text.translate

This is an intent to translate text from one language to another.

<!-- TOC depthFrom:2 -->

- [Basic usage](#basic-usage)
    - [Translation domains](#translation-domains)
    - [Bulk mode](#bulk-mode)
    - [Language detection mode](#language-detection-mode)
- [Getting available providers](#getting-available-providers)
    - [Capabilities of providers in async mode](#capabilities-of-providers-in-async-mode)
    - [Filtering providers by capabilities](#filtering-providers-by-capabilities)
- [Getting information about a provider](#getting-information-about-a-provider)
- [Supported languages](#supported-languages)
    - [List of supported languages](#list-of-supported-languages)
    - [Full information on a supported language](#full-information-on-a-supported-language)
- [Setting your own language codes](#setting-your-own-language-codes)
- [All language settings](#all-language-settings)
- [Custom Translation Models](#custom-translation-models)
    - [Using previously trained custom models](#using-previously-trained-custom-models)
    - [:lock: Training custom models](#lock-training-custom-models)
    - [:lock: Migrating custom models between providers](#lock-migrating-custom-models-between-providers)
- [Supported formats](#supported-formats)
    - [List of providers supporting a specified format](#list-of-providers-supporting-a-specified-format)
- [Content processing](#content-processing)

<!-- /TOC -->

## Basic usage

To translate a text, send a POST request to Intento API at [https://api.inten.to/ai/text/translate](https://api.inten.to/ai/text/translate). Specify the source text, the target language and the desired translation provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "A sample text",
        "to": "es"
    },
    "service": {
        "provider": "ai.text.translate.microsoft.translator_text_api.2-0"
    }
}'
```

The response contains the translated text and a service information:

```json
{
    "results": ["Un texto de ejemplo"],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.2-0",
            "name": "Microsoft Translator API"
        }
    }
}
```

If the provider doesn't have capabilities (e.g. bulk support or language pairs) to process request, 413 error will be returned:

```json
{
    "error": {
        "code": 413,
        "message": "Provider ai.text.translate.promt.cloud_api.1-0 constraint(s) violated: bulk (Bulk mode support)"
    }
}
```

### Translation domains

Currently we have a rudimentary support of the translation domains (`category`) with the main purpose to support choice between Statistical Machine Translation and Neural Machine Translation for Microsoft Translation Text API (for Microsoft Translation, the default mode is NMT - `"category": "generalnn"`).

```sh
curl -XPOST -H 'apikey: YOUR_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "A sample text",
        "to": "es",
        "category": "general"
    },
    "service": {
        "provider": "ai.text.translate.microsoft.translator_text_api.2-0"
    }
}'
```

Response:

```sh
{
    "results": [
        "Un texto de muestra"
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.2-0",
            "name": "Microsoft Translator API"
        }
    }
}
```

```sh
curl -XPOST -H 'apikey: YOUR_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "A sample text",
        "to": "es",
        "category": "generalnn"
    },
    "service": {
        "provider": "ai.text.translate.microsoft.translator_text_api.2-0"
    }
}'
```

Response:

```sh
{
    "results": [
        "Un texto de ejemplo"
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.2-0",
            "name": "Microsoft Translator API"
        }
    }
}
```

### Bulk mode

We provide a bulk fulfillment mode to translate an array of segments at once. The mode is activated by sending an array of strings to the `context.text` parameter. The bulk mode is supported for most of the providers.

```sh
curl -XPOST -H "apikey: YOUR_API_KEY" "$HOST/ai/text/translate" -d '{
    "context": {
        "text": [
            "A sample text",
            "Hello world"
        ],
        "from": "en",
        "to": "de"
    },
    "service": {
        "provider": "ai.text.translate.microsoft.translator_text_api.2-0"
    }
}'
```

The response contains the translated texts and a service information:

```json
{
    "results": [
        "Ein Beispieltext",
        "Hallo Welt"
    ],
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.2-0",
            "name": "Microsoft Translator API"
        }
    }
}
```

### Language detection mode

If the `from` parameter is omitted many providers allow automatic source language detection and return results of the detection.

If such information is available from a provider, it is returned in the "meta" section of the response:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "A sample text",
        "to": "es"
    },
    "service": {
        "provider": "ai.text.translate.google.translate_api.2-0"
    }
}'
```

The response contains the translated text, service information and meta information (i.e. detected language):

```json
{
    "results": [
        "Un ejemplo de texto"
    ],
    "meta": {
        "detected_source_language": "en"
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.google.translate_api.2-0",
            "name": "Google Cloud Translation API"
        }
    }
}
```

Depending on the provider, the result may contain either a single value (detected language for all the translated sentences) or an array of values (detected language for each of the translated sentences).

## Getting available providers

To get a list of available Machine Translation providers, send a GET request to [https://api.inten.to/ai/text/translate](https://api.inten.to/ai/text/translate).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate'
```

The response contains a list of the providers available for given constraints with an information on pricing etc:

```json
[
    {
        "id": "ai.text.translate.baidu.translate_api",
        "name": "Baidu Translate API",
        "score": 0,
        "price": 0
    },
    {
        "id": "ai.text.translate.google.translate_api.2-0",
        "name": "Google Cloud Translation API",
        "score": 0,
        "price": 0
    },
    {
        "id": "ai.text.translate.yandex.translate_api.1-5",
        "name": "Yandex Translate API",
        "score": 0,
        "price": 0
    },
    {
        "id": "ai.text.translate.systran.translation_api.1-0-0",
        "name": "SYSTRAN REST Translation API",
        "score": 0,
        "price": 0
    }
]
```

### Capabilities of providers in async mode

Async translation mode grants additional translation capabilities, such as processing of HTML documents by providers that do not support HTML. Accordingly, the features of providers for this mode are slightly different. For example, the list of supported formats may include HTML for providers that do not support HTML. 

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?mode=async'
```

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?from=en&to=es'
```

Response:

```json
[
    {
        "id": "ai.text.translate.baidu.translate_api",
        "name": "Baidu Translate API",
        "score": 0,
        "price": 0
    },
    {
        "id": "ai.text.translate.google.translate_api.2-0",
        "name": "Google Cloud Translation API",
        "score": 0,
        "price": 0
    },
    {
        "id": "ai.text.translate.yandex.translate_api.1-5",
        "name": "Yandex Translate API",
        "score": 0,
        "price": 0
    },
    {
        "id": "ai.text.translate.systran.translation_api.1-0-0",
        "name": "SYSTRAN REST Translation API",
        "score": 0,
        "price": 0
    }
]
```

Besides source and target language, service providers may be filtered by support of specific bulk translate option (`bulk`), language detection option (`lang_detect`) and supported format (`format`):

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?from=en&to=es&lang_detect=true&bulk=true&format=html'
```
It is important that when filtering by format for the async mode, the list of providers will be different, since more providers in this mode can work with HTML:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?mode=async&format=html'
```

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/text/translate/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/ai.text.translate.google.translate_api.2-0'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.text.translate.google.translate_api.2-0",
    "name": "Google Cloud Translation API",
    "logo": "https://inten.to/img/api/ggl_translate.png",
    "billing": true,
    "bulk": true,
    "format": ["text", "html"],
    "languages": {
        "symmetric": [
            "gu",
            "gd",
            "ga",
            "gl",
            "lb"
        ],
        "pairs": [
            {
                "from": "en",
                "to": "de"
            },
            {
                "from": "fr",
                "to": "en"
            }
        ]
    },
    "auth": {
        "key": "YOUR_KEY"
    }
}
```

As well as for the request for the list of available providers, the parameter `mode=async` can be specified: 

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/ai.text.translate.google.translate_api.2-0?mode=async'
```


## Supported languages

### List of supported languages

Will return an array of supported languages, for each language:

- iso name
- localized name (if `locale` parameter is provided); if there is no localized name, `null` is returned
- intento code
- client code (if the client calling the method has its own codes)

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/languages?locale=ru'
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

### Full information on a supported language

For a given language code (intento internal or client’s) will show full metadata:

- iso name
- localized name (if `locale` parameter is provided); if there is no localized name, `null` is returned
- intento code
- iso codes (ones which are applicable)
- providers’ codes (which map to this internal code)
- client code (if the client calling the method has its own codes)

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/languages/he?locale=ru'
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
    "provider_codes": {
        "ai.text.translate.google.translate_api.2-0": "iw"
    },
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

## All language settings

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

## Custom Translation Models

Some of the MT services allow for fine-tuning of the translation models. Using the Intento API you may have an access to the models you've already trained in the past or train new ones based on the data you have, for example:

- glossaries / term bases,
- parallel corpuses,
- monolingual corpuses

### Using previously trained custom models

Right now we support custom models for these providers:
- (SMT) IBM Watson Language Translator Service (`ai.text.translate.ibm-language-translator`). This engine is deprecated and will no longer be available after October 4, 2018.
- (SMT) Microsoft Translator API v2.0 (`ai.text.translate.microsoft.translator_text_api.2-0`). This engine is deprecated since April 30, 2018 and will be discontinued on April 30, 2019.
- (NMT) IBM Watson Language Translator Service v3 (`ai.text.translate.ibm-language-translator-v3`).
- (NMT) Microsoft Translator API v3.0 (`ai.text.translate.microsoft.translator_text_api.3-0`).
- (NMT) Google AutoML Translation API (v1beta1) (`ai.text.translate.google.automl_api.v1beta1`).
- ModernMT Enterprise Edition (`ai.text.translate.modernmt.enterprise`).
- Tilde Machine Translation API (`ai.text.translate.tilde.machine_translation_api`).


More providers to come!

For using a custom model trained with these services just pass the model id in the `category` parameter with your [own credentials](https://github.com/intento/intento-api#using-a-service-provider-with-your-own-keys) (because other users do not have access to your models):

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "Hallo Welt",
        "from": "de",
        "to": "en",
        "category": "cccccccc-cccc-cccc-cccc-cccccccc"
    },
    "service": {
        "provider" : "ai.text.translate.ibm-language-translator",
        "auth": {
            "ai.text.translate.ibm-language-translator": [
                {"user" : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx", "password" : "xxxxxxxx"}
            ]
        }
    }
}'
```

### :lock: Training custom models

TBD

### :lock: Migrating custom models between providers

TBD

## Supported formats

By default the translation engines process input texts as a plain text and do not take tags into account. Many providers support text formats other than plain text. Currently supported formats are: `text` (is the default, plain text), `html`, `xml`.

To translate a text using a specified format just add a `format` field into `context` parameters:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "<p>A <div>sample</div> text</p>",
        "to": "ru",
        "format": "html"
    },
    "service": {
        "provider": "ai.text.translate.google.translate_api.2-0"
    }
}'
```

The response contains the translated text with preserved formatting:

```json
{
    "results": ["<p> <div> \u043e\u0431\u0440\u0430\u0437\u0435\u0446 </div> \u0442\u0435\u043a\u0441\u0442 </p>"],
    "meta": {
        "detected_source_language": ["en"]
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.google.translate_api.2-0",
            "name": "Google Cloud Translation API"
        }
    }
}

```


Note that some MT engines eliminate newline characters from the text. If you rely on the linebreaks e.g. to separate segments, we suggest to split the input text by linebreaks, send all lines of the input text as an array via the bulk request (link) and merge the resulting array of translations.

### List of providers supporting a specified format

You can filter providers supporting a specified format in the same way as for other capabilities ([Filtering providers by capabilities](#filtering-providers-by-capabilities)). The following call will return a list of providers supporting HTML format:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?format=html'
```

Response:

```json
[
    {
        "id": "ai.text.translate.microsoft.translator_text_api.3-0",
        "name": "Microsoft Translator API v3.0",
        "score": 0,
        "price": 0
    },
    {
        "id": "ai.text.translate.google.translate_api.2-0",
        "name": "Google Cloud Translation API",
        "score": 0,
        "price": 0
    },
    ...
]
```

You can see formats supported by a provider in the results of a GET request (see [Getting information about a provider](#getting-information-about-a-provider)). The field "format" contains a list of supported text formats.

## Content processing

Sometimes it's more convenient to preprocess or postprocess text after translation, e.g. eliminate spaces before punctuation. You can easily delegate it to Intento API with `processing` tag. This tag includes a set of rules.

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "A sample text",
        "to": "es"
    },
    "service": {
        "provider": "ai.text.translate.microsoft.translator_text_api.2-0",
        "processing": {
            "pre": [
                "punctuation_set"
            ],
            "post": [
                "punctuation_set"
            ]
        }
    }
}'
```

More information about sets and rules can be found at `https://api.inten.to/settings/processing-rules`. Each rule has a clear `description` what exactly it does.

```json
{
    "results": {
        "punctuation_set": [
            {
                "language": "Generic",
                "exception": "None",
                "intent_type": "ai.text",
                "description": "Remove space(s) before period or comma",
                "rule": {
                    "type": "regex",
                    "pattern": {
                        "search": "(\\S)\\s+(\\.|,|\\u3002)",
                        "replace": "$1$2"
                    },
                    "params": {
                        "dotall": "false",
                        "ignorecase": "true",
                        "multiline": "false",
                        "unicode": "true"
                    }
                }
            },
            ...
        ]
    }
}
```
