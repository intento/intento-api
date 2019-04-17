# Providers

<!-- TOC -->

- [Providers](#providers)
    - [General info](#general-info)
    - [Technical info](#technical-info)
    - [Flags](#flags)
        - [`billable`](#billable)
        - [`bulk`](#bulk)
        - [`custom_glossary`](#custom_glossary)
        - [`custom_model`](#custom_model)
        - [`delegated_credentials`](#delegated_credentials)
        - [`integrated`](#integrated)
        - [`lang_detect` - an intent-specific flag](#lang_detect---an-intent-specific-flag)
        - [`own_auth`](#own_auth)
        - [`production`](#production)
        - [`stock_model`](#stock_model)
        - [Filtering providers by capabilities](#filtering-providers-by-capabilities)

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

**`languages`** (dictionary `{ 'symmetric': ['list', 'of', 'strings'], 'pairs': ['list', 'of', 'strings'] }`) - supportes languages if applicable to the intent.

**`format`** (list of strings) supported data formats. Like `text`, `html`, `xml` for the translation intent.

## Flags

Flag's purpose is to reveal provider's properties at Intento. All of them are boolean.

### `billable`

A provider can be called without passing own keys.
If `billable:false`, one must pass their own keys to the provider.

### `bulk`

A provider supports bulk requests. In our async mode we have sophisticatd workaround for providers with `bulk: false`, so when using Intentio in async mode (in Web Tools for example) there is no need to worry about this flag.

### `custom_glossary`

Machine translation intent only.

A provider supports passing custom glossaries.

### `custom_model`

A provider supports training models (only on their side for now) and passing them as a `custom_model` parameter when requesting a job.

### `delegated_credentials`

Intento supports connecting accounts to this provider. For example to Google AutoML. See more details in the [delegated credentials](./delegated_credentials.md)

### `integrated`

Intento can route requests to a provider.

### `lang_detect` - an intent-specific flag

Machine translation, text classify and sentiment intents only.

A provider supports language detection.

### `own_auth`

A provider supports requests with own keys.

### `production`

A provider advertises their service as beta or preview.

### `stock_model`

A provider can perform requests without uploading training data.

### Filtering providers by capabilities

One can get a list of providers, supporting these or that features (filtering not only by flags).
Some of these features are intent-specific, so it is useful to check the "Filtering providers by capabilities" section in the certan intent doc. For example [Filtering providers by capabilities for Sentiment analysis](./ai.text.sentiment.md#filtering-providers-by-capabilities).

One of the examples of how to filter by a feature, that is supported by all intents is filtering providers by capabilities in the async mode.
The Intento async mode in general allows to use more providers. For example one can translate batches with Amazon Translate, while Amazon itself does not support such feature. Also translating html format is available for all providers while requesting Intento in the async mode.

Compare responses on these two requests:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' https://api.inten.to/ai.text.translate/?mode=async&format=html
curl -H 'apikey: YOUR_INTENTO_KEY' https://api.inten.to/ai.text.translate/format=html
```

The first response will contain descriptions of 25 providers and the second one -- of 13 providers only.
