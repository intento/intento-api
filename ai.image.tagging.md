# ai.image.tagging

Image tagging generates a list of words or tags that are relevant to the content of the supplied image.

<!-- TOC depthFrom:2 -->

- [Basic usage](#basic-usage)
- [Getting available providers](#getting-available-providers)
    - [Filtering providers by capabilities](#filtering-providers-by-capabilities)
- [Getting information about a provider](#getting-information-about-a-provider)

<!-- /TOC -->

## Basic usage

To tag an image, send a POST request to Intento API at [https://api.inten.to/ai/image/tagging](https://api.inten.to/ai/image/tagging). Specify the source and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/image/tagging' -d '{
    "context": {
        "source": "..."
    },
    "service": {
        "provider": "ai.image.tagging.amazon.recognition_detect_labels_api"
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
            "id": "ai.image.tagging.amazon.recognition_detect_labels_api"
        }
    }
}
```

If the provider doesn't have capabilities (e.g. language) to process request, 413 error will be returned:

```json
{
  "error": {
    "code": 413,
    "message": "Provider ai.image.tagging.amazon.recognition_detect_labels_api constraint(s) violated."
  },
  "request_id": "..."
}
```

## Getting available providers

To get a list of available providers, send a GET request to [https://api.inten.to/ai/image/tagging](https://api.inten.to/ai/image/tagging).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/image/tagging'
```

The response contains a list of the providers available for given constraints:

```json
[
    {
        "id": "ai.image.tagging.baidu.image_classify_api",
        "api_id": "baiduimageclassify",
        "name": "Baidu",
        "description": "Image Classify",
        "own_auth": true
    }
]
```

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/image/tagging?language=ru'
```

Response:

```json
[
    {
        "id": "ai.image.tagging.amazon.recognition_detect_labels_api",
        "description": "API Detect Labels",
        "name": "Amazon Rekognition",
        "integrated": false,
        "own_auth": true,
        "stock_model": false,
        "custom_model": false
    },
    ...
]
```

More on [provider flags and capabilities](providers.md).

## Getting information about a provider

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/image/tagging/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/image/tagging/ai.image.tagging.google.cloud_vision_annotate_api.1-0'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.image.tagging.google.cloud_vision_annotate_api.1-0",
    "name": "Google Cloud",
    "description": "Vision API",
    "own_auth": true,
    "stock_model": true,
    "custom_model": false,
}
```
