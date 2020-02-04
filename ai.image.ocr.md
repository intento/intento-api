# ai.image.ocr

Optical Character Recognition (OCR) detects and extracts text within an image with support for a broad range of languages.

<!-- TOC depthFrom:2 -->

- [Basic usage](#basic-usage)
- [Getting available providers](#getting-available-providers)
    - [Filtering providers by capabilities](#filtering-providers-by-capabilities)
- [Getting information about a provider](#getting-information-about-a-provider)

<!-- /TOC -->

## Basic usage

To detect or extract text within an image, send a POST request to Intento API at [https://api.inten.to/ai/image/ocr](https://api.inten.to/ai/image/ocr). Specify the source and the desired provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/image/ocr' -d '{
    "context": {
        "image": "..."
    },
    "service": {
        "provider": "ai.image.ocr.abbyy.cloud_ocr"
    }
}'
```

The response contains the results:

```json
{
    "results": "...",
    "meta": {},
    "service": {
        "provider": {
            "id": "ai.image.ocr.abbyy.cloud_ocr"
        }
    }
}
```

If the provider doesn't have capabilities (e.g. language) to process request, 413 error will be returned:

```json
{
  "error": {
    "code": 413,
    "message": "Provider ai.image.ocr.abbyy.cloud_ocr constraint(s) violated."
  },
  "request_id": "..."
}
```

## Getting available providers

To get a list of available providers, send a GET request to [https://api.inten.to/ai/image/ocr](https://api.inten.to/ai/image/ocr).

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/image/ocr'
```

The response contains a list of the providers available for given constraints:

```json
[
    {
        "id": "ai.image.ocr.microsoft.vision_api.1-0",
        "name": "Microsoft",
        "description": "Computer Vision API",
        "own_auth": true
    }
]
```

### Filtering providers by capabilities

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/image/ocr?own_auth=true'
```

Response:

```json
[
    {
        "id": "ai.image.ocr.google.cloud_vision_api.1-1",
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

To get information about a provider with a given ID, send a GET request to `https://api.inten.to/ai/image/ocr/PROVIDER_ID`.

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/image/ocr/ai.image.ocr.google.cloud_vision_api.1-1'
```

The response contains a list of the metadata fields and values available for the provider:

```json
{
    "id": "ai.image.ocr.google.cloud_vision_api.1-1",
    "vendor": "Google Cloud",
    "logo": "https://inten.to/static/img/api/ggl_vision.png",
    "billing": true,
    "description": "Vision API",
    "production": true,
    "integrated": false,
    "billable": true,
    "own_auth": true,
    "stock_model": true,
    "custom_model": false,
    "auth": {
        "key": "YOUR_KEY"
    }
}
```
