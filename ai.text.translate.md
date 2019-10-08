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
    - [Listing available models and glossaries](#listing-available-models-and-glossaries)
    - [Using previously trained custom models](#using-previously-trained-custom-models)
    - [Using glossaries](#using-glossaries)
    - [:lock: Training custom models](#lock-training-custom-models)
    - [:lock: Migrating custom models between providers](#lock-migrating-custom-models-between-providers)
- [Supported formats](#supported-formats)
    - [Supported formats in async mode](#supported-formats-in-async-mode)
        - [HTML/XML formats](#htmlxml-formats)
            - [Overview](#overview)
            - [Important notes](#important-notes)
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
        "provider": "ai.text.translate.microsoft.translator_text_api.3-0"
    }
}'
```

The response contains the translated text, meta and a service information:

```json
{
    "results": ["Un texto de ejemplo"],
    "meta": {
        "detected_source_language": [ "en" ],
        "timing": { "total": 0.11, "providers": 0.1 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.3-0",
            "name": "Microsoft Translator API v3.0",
            "timing": { "provider": 0.1 }
        }
    }
}
```

All timings are in seconds.

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
        "provider": "ai.text.translate.microsoft.translator_text_api.3-0"
    }
}'
```

Response:

```sh
{
    "results": [
        "Un texto de muestra"
    ],
    "meta": {
        "detected_source_language": [ "en" ],
        "timing": { "total": 0.15, "providers": 0.14 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.3-0",
            "name": "Microsoft Translator API v3.0",
            "category": {"id": "general"}
            "timing": { "provider": 0.14 }
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
        "provider": "ai.text.translate.microsoft.translator_text_api.3-0"
    }
}'
```

Response:

```sh
{
    "results": [
        "Un texto de muestra"
    ],
    "meta": {
        "detected_source_language": [ "en" ],
        "timing": { "total": 0.078236, "providers": 0.06625 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.3-0",
            "name": "Microsoft Translator API v3.0",
            "category": {"id": "generalnn"}
            "timing": { "provider": 0.06625 }
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
        "provider": "ai.text.translate.microsoft.translator_text_api.3-0"
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
    "meta": {
        "detected_source_language": [ "en", "en" ],
        "timing": { "total": 0.13, "providers": 0.12 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.microsoft.translator_text_api.3-0",
            "name": "Microsoft Translator API v3.0",
            "timing": { "provider": 0.12 }
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
        "provider": "ai.text.translate.google.translate_api.v3beta1"
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
        "detected_source_language": [ "en" ],
        "timing": { "total": 0.12, "providers": 0.1 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.google.translate_api.v3beta1",
            "name": "Google Cloud Translation API (v3beta1)",
            "timing": { "provider": 0.1 }
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

The response contains a list of the providers available for given constraints with an information on using custom models, etc.:

```json
[
    {
        "id": "ai.text.translate.google.translate_api.v3beta1",
        "vendor": "Google Cloud",
        "description": "Translation API (v3beta1)",
        "own_auth": true,
        "stock_model": true,
        "custom_model": true
    },
    {
        "id": "ai.text.translate.sdl.language_cloud_translation_toolkit",
        "vendor": "SDL",
        "description": "Language Cloud Translation Toolkit",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    },
    {
        "id": "ai.text.translate.deepl.api",
        "vendor": "DeepL",
        "description": "API",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    },
    {
        "id": "ai.text.translate.yandex.cloud-translate.v2",
        "vendor": "Yandex Cloud",
        "description": "Translate API v2",
        "own_auth": false,
        "stock_model": true,
        "custom_model": false
    },
    {
        "id": "ai.text.translate.baidu.translate_api",
        "vendor": "Baidu",
        "description": "Translate API",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    }
]
```

### Capabilities of providers in async mode

Async translation mode grants additional translation capabilities. All providers support [bulk](#bulk-mode) and HTML, XML formats in the async mode.

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
        "id": "ai.text.translate.amazon.translate",
        "vendor": "Amazon",
        "description": "Translate",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    },
    {
        "id": "ai.text.translate.google.translate_api.v3beta1",
        "vendor": "Google Cloud",
        "description": "Translation API (v3beta1)",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    },
    {
        "id": "ai.text.translate.systran.translation_api.1-0-0",
        "vendor": "SYSTRAN",
        "description": "REST Translation API",
        "own_auth": true,
        "stock_model": true,
        "custom_model": false
    },
    {
        "id": "ai.text.translate.ibm-language-translator-v3",
        "vendor": "IBM Watson",
        "description": "Language Translator Service v3",
        "own_auth": false,
        "stock_model": true,
        "custom_model": true
    },
    {
        "id": "ai.text.translate.modernmt.enterprise",
        "vendor": "ModernMT",
        "description": "Enterprise Edition",
        "own_auth": true,
        "stock_model": true,
        "custom_model": true
    }
]
```

Besides source and target language, service providers may be filtered by support of specific bulk translate option (`bulk`) and language detection option (`lang_detect`):

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?from=en&to=es&lang_detect=true&bulk=true'
```

It is important that when filtering by HTML/XML format in the async mode, the list of providers will be different, since all providers in this mode can work with HTML/XML:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?mode=async&format=html'
```

And when filtering by bulk in the async mode, the list of providers will be different, all providers in this mode can work with bulk:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?mode=async&bulk=true'
```

More on [provider flags and capabilities](providers.md).

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/text/translate/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/ai.text.translate.google.translate_api.2-0'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.text.translate.google.translate_api.v3beta1",
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
        "ai.text.translate.google.translate_api.v3beta1": "iw"
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

### Listing available models and glossaries

If you have an MT provider account with trained models and glossaries, and you [connected](./delegated-credentials) this account to your Intento account, you can access these models and glossaries through Intento API.

For now, Intento supports glossaries and models for Google v3.

To set up glossaries for your Google v3 project follow instructions [here](https://cloud.google.com/translate/docs/glossary).

To set up Google v3 models follow instructions [here](https://cloud.google.com/translate/automl/docs/).

To continue you will need a Google v3 account connected to your Intento account. You can connect accounts in [Intento Console](https://console.inten.to/credentials) or using [our API](./delegated-credentials)

To access assets for a provider `example-provider` you will need an Intento `example-provider-credential_id` of a provider account connected to Intento. This id you can get on the [Connected account page](https://console.inten.to/credentials), where you have already connected some accounts.

Getting a list of glossaries:

```sh
curl -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate/glossaries?provider=example-provider&credential_id=example-provider-credential_id'
```

Response:

```json
{
    "response": [
        {
            "id": "projects/automl-123456/locations/us-central1/glossaries/my_EN_ES_Glossary",
            "name": "my_EN_ES_Glossary",
            "from": "en",
            "to": "es"
        },
        {
            ...
        },
        ...
    ]
}
```

Getting a list of models:

```sh
curl -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate/models?provider=example-provider&credential_id=example-provider-credential_id'
```

Response:

```json
{
    "response": [
        {
            "id": "projects/mytest-123456/locations/us-central1/models/my_EN_ES_Model654321",
            "name": "my_EN_ES_Model",
            "from": "en",
            "to": "es",
            "stock": false
        },
        {
            ...
        },
        ...
    ]
}
```

### Using previously trained custom models

Right now we support custom models for these providers:

- (SMT) IBM Watson Language Translator Service (`ai.text.translate.ibm-language-translator`). This engine is deprecated and will no longer be available after October 4, 2018.
- (SMT) Microsoft Translator API v2.0 (`ai.text.translate.microsoft.translator_text_api.2-0`). This engine is deprecated since April 30, 2018 and will be discontinued on April 30, 2019.
- (NMT) IBM Watson Language Translator Service v3 (`ai.text.translate.ibm-language-translator-v3`).
- (NMT) Microsoft Translator API v3.0 (`ai.text.translate.microsoft.translator_text_api.3-0`).
- (NMT) Google Cloud Translation API (v3beta1) (`ai.text.translate.google.translate_api.v3beta1`).
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
        "provider" : "ai.text.translate.google.translate_api.v3beta1",
        "auth": {
            "ai.text.translate.google.translate_api.v3beta1": [
                {"credential_id": "credentials_for_google_v3_123456"}
            ]
        }
    }
}'
```

Response

```json
{
    "results": [
        "Hello world"
    ],
    "meta": {
        "timing": { "total": 0.21, "providers": 0.2 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.google.translate_api.v3beta1",
            "name": "Google Cloud Translation API (v3beta1)",
            "timing": { "provider": 0.2 },
            "category": {
                "id": "cccccccc-cccc-cccc-cccc-cccccccc"
            },
            "credential_id": "credentials_for_google_v3_123456"
        }
    }
}
```

### Using glossaries

Some of MT services allow using a glossary. A glossary is a custom dictionary which to make sure that you translate terms that are specific to you precisely the way you need it.

Translate with glossary:

```sh
curl -H 'apikey: YOUR_API_KEY' -XPOST 'https://api.inten.to/ai/text/translate' -d'{
    "context": {
        "text": "Hello World",
        "from": "en",
        "to": "es",
        "glossary": "projects/automl-123456/locations/us-central1/glossaries/my_EN_ES_Glossary"
    },
    "service": {
        "provider" : "ai.text.translate.google.translate_api.v3beta1",
        "auth": {
            "ai.text.translate.google.translate_api.v3beta1": [
                {"credential_id": "credentials_for_google_v3_123456"}
            ]
        }
    }
}'
```

Response

```json
{
    "results": [
        "Hola Mundo"
    ],
    "meta": {
        "timing": { "total": 0.21, "providers": 0.2 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.google.translate_api.v3beta1",
            "name": "Google Cloud Translation API (v3beta1)",
            "timing": { "provider": 0.2 },
            "glossary": {
                "id": "projects/automl-123456/locations/us-central1/glossaries/my_EN_ES_Glossary"
            },
            "credential_id": "credentials_for_google_v3_123456"
        }
    }
}
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
        "provider": "ai.text.translate.google.translate_api.v3beta1"
    }
}'
```

The response contains the translated text with preserved formatting:

```json
{
    "results": ["<p> <div> \u043e\u0431\u0440\u0430\u0437\u0435\u0446 </div> \u0442\u0435\u043a\u0441\u0442 </p>"],
    "meta": {
        "detected_source_language": [ "en" ],
        "timing": { "total": 0.18, "providers": 0.17 }
    },
    "service": {
        "provider": {
            "id": "ai.text.translate.google.translate_api.v3beta1",
            "name": "Google Cloud Translation API (v3beta1)",
            "timing": { "provider": 0.17 }
        }
    }
}

```

Note that some MT engines eliminate newline characters from the text. If you rely on the linebreaks e.g. to separate segments, we suggest to split the input text by linebreaks, send all lines of the input text as an array via the bulk request (link) and merge the resulting array of translations.

### Supported formats in async mode

In addition to the provider's format support, we have built-in support for HTML, XML and XLIFF formats.

#### HTML/XML formats

If a provider doesn't support HTML/XML format, you can still translate HTML/XML input with this provider and get valid HTML translation. Note, that this is supported only in [async mode](https://github.com/intento/intento-api#async-mode).

##### Overview

For the HTML/XML format support, we transform an HTML document into a tree. Then we traverse the tree and get all the text content from tags. This procedure converts HTML/XML input into an array of text, which doesn't contain tags. We send this array for translation and insert back the response into the initial HTML input.

##### Important notes

- The HTML/XML format support fixes invalid HTML with missing opening or closing tags by adding missing one.
- We send for translation only text content of tags (`<tagname>CONTENT</tagname>`).
- We don't send for translation content of these tags:
    - `area`
    - `basefont`
    - `datalist`
    - `link`
    - `meta`
    - `noembed`
    - `noframes`
    - `param`
    - `rp`
    - `script`
    - `source`
    - `style`
    - `template`
    - `track`
    - `noscript`

Let's examine how it works. There is an HTML input:

```html
<html>
<head>
<title>Title of this HTML</title>
</head>
<body>
<div id='tag1'>Hi!</div>
<div id='tag2'>Buy!</div>
</body>
</html>
```

After conversion, there is three elements in the request for translation:

```python
[
'Title of this HTML',
'Hi!',
'Buy!'
]

```

It is worth mentioning that converting HTML to text may not work well with inline tags. Take this HTML for example:

```html
<body>
<html>
<head>
<title></title>
<meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
</head>
<body>
<p>The young Princess Bolkónskaya had brought some work in a gold-embroidered velvet bag.<br> Her pretty little upper lip, on which a delicate dark down was just perceptible, was too short for her teeth, but it lifted all the more sweetly, and was especially charming when she occasionally drew it down to meet the lower lip. <br>As is always the case<br> with a thoroughly attractive woman, her defect&mdash;the shortness of her upper lip and her half-open mouth&mdash;seemed to be her own special and peculiar form of beauty. Everyone brightened at the sight of this pretty young woman, so soon to become a mother, so full of life and health, and carrying her burden so lightly. Old men and dull dispirited young ones who <i>looked<i> at her, after being in her company and talking to her a little while, felt as if they too were becoming, like her, full of life and health. All who talked to her, and at each <span>word saw her bright smile</span> and the constant gleam of her white teeth, thought that they were in a specially amiable mood that day.</p>
</body>
</html>
</body>
</html>
```

After conversion there is nine items in the array:

```python
[
'The young Princess Bolkónskaya had brought some work in a gold-embroidered velvet bag.',
'Her pretty little upper lip, on which a delicate dark down was just perceptible, was too short for her teeth, but it lifted all the more sweetly, and was especially charming when she occasionally drew it down to meet the lower lip.',
'As is always the case',
' with a thoroughly attractive woman, her defect&mdash;the shortness of her upper lip and her half-open mouth&mdash;seemed to be her own special and peculiar form of beauty. Everyone brightened at the sight of this pretty young woman, so soon to become a mother, so full of life and health, and carrying her burden so lightly.',
'Old men and dull dispirited young ones who ',
'looked',
'at her, after being in her company and talking to her a little while, felt as if they too were becoming, like her, full of life and health. All who talked to her, and at each ',
'word saw her bright smile',
' and the constant gleam of her white teeth, thought that they were in a specially amiable mood that day.'
]
```

The first and the second items are produced after splitting by `<br>` tags. These elements seem fine. Next item 'As is...'  is not adequately extracted because the `<br>` tag is in the middle of a sentence. Instead of one element for the sentence, two are formed, each of which is a separate item in the array. As you can see, the following elements starting from the fifth are also extracted not correctly due to the `<i>` and `<span>` tags in the middle of sentences. To avoid such splittings, be sure that the inline tags do not divide a sentence.


#### XLIFF format

#### Memsource version of the XLIFF

We support the [Memsource version of the XLIFF (v1.2)](https://help.memsource.com/hc/en-us/articles/360004658951) format in [async mode](https://github.com/intento/intento-api#async-mode). To translate Memsource version of the XLIFF add `format`: `xliff-memsource`:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
    "context": {
        "text": "<xliff version=\"1.2\" xmlns....",
        "format": "xliff-memsource"
    },
    "service": {
        "provider": "ai.text.translate.microsoft.translator_text_api.3-0",
        "async": true
    }
}'
```

`from` and `to` parameters are specified in the XLIFF file as [source-language](http://docs.oasis-open.org/xliff/v1.2/os/xliff-core.html#source-language) tag and [target-language](http://docs.oasis-open.org/xliff/v1.2/os/xliff-core.html#target-language) tag respectively.

We also support translation of the Memsource XLIFF (MXLIFF) files in [Intento Console](https://console.inten.to).

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
        "vendor": "Microsoft",
        "description": "Translator API v3.0",
        "own_auth": true,
        "stock_model": true,
        "custom_model": true
    },
    {
        "id": "ai.text.translate.google.translate_api.v3beta1",
        "vendor": "Google Cloud",
        "description": "Translation API (v3beta1)",
        "own_auth": true,
        "stock_model": false,
        "custom_model": true
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
        "provider": "ai.text.translate.microsoft.translator_text_api.3-0",
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
