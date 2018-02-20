# ai.text.translate

This is an intent to translate text from one language to another.

## Basic usage

To translate a text, send a POST request to Intento API at https://api.inten.to/ai/text/translate. Specify the source text, the target language and the desired translation provider in JSON body of the request as in the following example:

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

```sh
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

```sh
{
    "error": {
        "code": 413,
        "message": "Provider ai.text.translate.promt.cloud_api.1-0 constraint(s) violated: bulk (Bulk mode support)"
    }
}
```

### Translation domains

Currently we have a rudimentary support of the translation domains (`category`) with the main purpose to support choice between Statistical Nachine Translation and Neural Machine Translation for Microsoft Translation Text API (for Microsoft Translation, the default mode is NMT - `"category": "generalnn"`). 

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
{"results": ["Un texto de muestra"], "meta": {}, "service": {"provider": {"id": "ai.text.translate.microsoft.translator_text_api.2-0", "name": "Microsoft Translator API"}}}
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
{"results": ["Un texto de ejemplo"], "meta": {}, "service": {"provider": {"id": "ai.text.translate.microsoft.translator_text_api.2-0", "name": "Microsoft Translator API"}}}
```

### Bulk mode
We provide a bulk fulfillment mode to translation an array of segments at once. The mode is activated by sending an array of strings to the `context.text` parameter. The bulk mode is supported for most of the providers.

```sh
curl -XPOST -H "apikey: YOUR_API_KEY" "$HOST/ai/text/translate" -d '
{
  "context": {
    "text": ["A sample text", "Hello world"],
    "from": "en",
    "to": "de"
  },
  "service": {
    "provider" : "ai.text.translate.microsoft.translator_text_api.2-0"
  }
}'
```

The response contains the translated texts and a service information:

```sh
{
 "results": ["Ein Beispieltext", "Hallo Welt"],
 "meta": {},
 "service": {
  "provider": {
   "id": "ai.text.translate.microsoft.translator_text_api.2-0",
   "name": "Microsoft Translator API"
  }
 }
}
```

### :lock: Multi mode
In the multi mode, the translation of the text is performed using a list of providers. The mode is activated by passing an array of provider identificators.

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
 "context": {
  "text": "A sample text",
  "to": "es"
 },
 "service": {
  "provider": ["ai.text.translate.google.translate_api.2-0", "ai.text.translate.yandex.translate_api.1-5"]
 }
}'
```
 
The response contains the translated text and a service information:          ↑

```sh
[
{
 "results": ["Un ejemplo de texto"],
 "meta": {},
 "service": {
  "provider": {
   "id": "ai.text.translate.google.translate_api.2-0",
   "name": "Google Cloud Translation API"
  }
 }
},
{
 "results": ["Un texto de ejemplo"],
 "meta": {},
 "service": {
  "provider": {
   "id": "ai.text.translate.yandex.translate_api.1-5",
   "name": "Yandex Translate API"
  }
 }
}
]
```


### :lock: Language detection mode
If the "from" parameter is omitted many providers allow automatic source language detection and return results of the detection. 

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

```sh
{
 "results": ["Un ejemplo de texto"],
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

Depending on the provider, the result may contain either a single value (detected language for all the translated sentences) or an array of values (detected language for each of the translated sentences).          ↑


## Getting available providers

To get a list of available Machine Translation providers, send a GET request to https://api.inten.to/ai/text/translate. 

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate'
```
 
The response contains a list of the providers available for given constraints with an information on pricing etc:

```sh
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

### Advanced usage

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?from=en&to=es'
```

```sh
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

Besides source and target language, service providers may be filtered by support of specific bulk translate option (`bulk`) and language detection option (`lang_detect`):

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?from=en&to=es&lang_detect=true&bulk=true'
```

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to https://api.inten.to/ai/text/translate/PROVIDER_ID. 

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/ai.text.translate.google.translate_api.2-0'
```

The response contains a list of the metadata fields and values available for the provider:

```sh
{
  "id": "ai.text.translate.google.translate_api.2-0",
  "name": "Google Cloud Translation API",
  "logo": "https://inten.to/img/api/ggl_translate.png",
  "billing": true,
  "bulk": true,
  "languages": {
   "symmetric": [
       "gu","gd","ga","gl","lb"
    ],
   "pairs": [
     [
       ["en", "de"],
       ["fr", "en"] 
     ]
   ]
  }
}
```

## Supported languages

### Getting list of supported languages

Will return an array of supported languages, for each language:

* iso name
* localized name (if `locale` parameter is provided); if there is no localized name, `null` is returned
* intento code
* client code (if the client calling the method has its own codes)
 
```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/languages?locale=ru'
```

```
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

* iso name
* localized name (if `locale` parameter is provided); if there is no localized name, `null` is returned
* intento code
* iso codes (ones which are applicable)
* providers’ codes (which map to this internal code)
* client code (if the client calling the method has its own codes)

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate/languages/he?locale=ru'
```

```
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

To define your aliases to language codes, send a POST request to Intento API at https://api.inten.to/settings/languages. After 60 seconds, you can start using them.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/settings/languages' --data '{"aliasforen":"en"}'
{
    "aliasforen": "en"
}
```

Settings can be retrieved using the GET request

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/settings/languages'
{
    "aliasforen": "en"
}
```

## :lock: Custom Translation Models

Some of the MT services allow for fine-tuning of the translation models. Using the Intento API you may have an access to the models you've already trained in the past or train new ones based on the data you have, for example:
- glossaries / term bases,
- parallel corpuses,
- monolingual corpuses

TBD

### :lock: Migrating custom models between providers

TBD