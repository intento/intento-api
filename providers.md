# Providers

<!-- TOC -->

- [Providers](#providers)
    - [General info](#general-info)
    - [Technical info](#technical-info)
    - [Flags](#flags)
        - [`billable`](#billable)
        - [`bulk`](#bulk)
        - [`custom_glossary` - an intent-specific flag](#custom_glossary---an-intent-specific-flag)
        - [`custom_model`](#custom_model)
        - [`delegated_credentials`](#delegated_credentials)
        - [`integrated`](#integrated)
        - [`lang_detect` - an intent-specific flag](#lang_detect---an-intent-specific-flag)
        - [`own_auth`](#own_auth)
        - [`production`](#production)
        - [`stock_model`](#stock_model)
    - [Filtering providers by capabilities](#filtering-providers-by-capabilities)
        - [Async mode](#async-mode)

<!-- /TOC -->

One can get detailed information about providers with GET requests:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' https://api.inten.to/ai.text.translate/
```

or

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' https://api.inten.to/ai.speech.transcribe/
```

See "Getting available providers" section in the intent docs like [ai.text.detect-intent.md](./ai.text.detect-intent.md#getting-information-about-a-provider).

A typical response looks like this

```json
[
    {
        "id": "ai.image.tagging.alibaba.cloud_image_tagging",
        "name": "Alibaba Cloud Image Tagging",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.baidu.image_classify_api",
        "name": "Baidu Image Classify",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.tencent.cloud_image_tag",
        "name": "Tencent Cloud Image tag",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.amazon.recognition_detect_labels_api",
        "name": "Amazon Rekognition API Detect Labels",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.google.cloud_vision_annotate_api.1-0",
        "name": "Google Cloud Vision API",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.ibm.watson_visual_recognition_api.3-0",
        "name": "IBM Watson Developer Cloud Visual Recognition API",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.microsoft.computer_vision_api.2-0",
        "name": "Microsoft Computer Vision API",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.google.automl_vision_api.v1beta1",
        "name": "Google Cloud AutoML Vision API (v1beta1)",
        "other_fields": "..."
    },
    {
        "id": "ai.image.tagging.hpe.haven_image_classification",
        "name": "HPE Haven Image Classification",
        "other_fields": "..."
    }
]
```

One can get profider `id` and requests detailed information:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' https://api.inten.to/ai.text.sentiment/ai.text.sentiment.twinword.sentiment
```

A typical response looks like this

```json
{
    "id": "ai.text.sentiment.twinword.sentiment",
    "vendor": "Twinword",
    "logo": "https://inten.to/static/img/api/twinword_sentiment.png",
    "billing": true,
    "description": "Sentiment Analysis",
    "production": true,
    "integrated": true,
    "billable": true,
    "own_auth": true,
    "stock_model": true,
    "custom_model": false,
    "delegated_credentials": false,
    "auth": {
        "key": "YOUR_KEY"
    },
    "languages": {
        "lang": [
            "en"
        ]
    },
    "lang_detect": false,
    "bulk": false
}
```

## General info

**`id`** (string) - unique identifier

**`vendor`** (string) - vendor name

**`description`** (string) - details about the provider

**`logo`** (string) - path to the image (url)

**`auth`** (dictionary) - structure required for the authentification

## Technical info

It may vary depending on the intent (for example, translate intent or image tagging intent).

**`languages`** (dictionary `{ "symmetric": ["list", "of", "strings"], "pairs": ["list", "of", "strings"] }`) - supports languages if applicable to the intent.

**`format`** (list of strings) supported data formats. Like `text`, `html`, `xml` for the translation intent.

## Flags

Flag's purpose is to reveal provider's properties at Intento. All of them are boolean.

### `billable`

A provider can be called without passing own keys.
If `billable:false`, one must pass their own keys to the provider.

### `bulk`

A provider supports bulk requests. In our async mode we have sophisticatd workaround for providers with `bulk: false`, so when using Intentio in async mode (in Web Tools for example) there is no need to worry about this flag.

### `custom_glossary` - an intent-specific flag

*Machine translation intent only.*

A provider supports passing custom glossaries.

### `custom_model`

A provider supports training models (only on their side for now) and passing them as a `custom_model` parameter when requesting a job.

### `delegated_credentials`

Intento supports connecting accounts to this provider. For example to Google AutoML. See more details in the [delegated credentials](./delegated_credentials.md)

### `integrated`

Intento can route requests to a provider.

### `lang_detect` - an intent-specific flag

*Machine translation, text classify and sentiment intents only.*

A provider supports language detection.

### `own_auth`

A provider supports requests with own keys.

### `production`

A provider advertises their service as beta or preview.

### `stock_model`

A provider can perform requests without uploading training data.

## Filtering providers by capabilities

Intento providers support different features. Some of these features are common for all intents (like own auth support or stock model support), other features (like language detection) are intent-specific. So it is useful to check the "Filtering providers by capabilities" section for the certain intent. For example [Filtering providers by capabilities for Sentiment analysis](./ai.text.sentiment.md#filtering-providers-by-capabilities).

You can get a list of providers that support a specific feature.
All providers supporting html can be filtered with a request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' https://api.inten.to/ai.text.translate?format=html
```

### Async mode

You can request, say, machine translation jobs or image recognition jobs in the [Intento async mode](./README.md#async-mode). It is not only a way to process large volumes of data, but also a way to use providers which do not support some features (like html format) by itself.

You can get a list of providers which support html in async mode with the following request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' https://api.inten.to/ai.text.translate?format=html&mode=async
```

Compare the response of this request with the response to the request without `mode=async` - you'll see significantly more providers in it.
