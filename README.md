# API User Manual

## Overview

This is a human-friendly manual to the Intento API. The interactive API Reference is available at https://intento.github.io.

### Intents

Intento API provides a uniform intent-based access to cognitive services from multiple providers, available at the Intento Service Platform. The intent is a basic action (like `translate`). Currently, only Machine Translation services are available (intent `ai.text.translate`).
 
Intent APIs are accessed via the base path. That said, API for the Machine Translation intent (ai.text.translate) is available at `/ai/text/translate`.

### Using the API

In order to get a list of providers available for some intent, use `GET` requests. Some providers support a subset of all available inputs for an intent. For example, not every Machine Translation service supports Korean to Russian language pair. To get a list of providers that support particular inputs, specify the input data constraints in GET parameters (see below).
 
In order to fulfill an intent (e.g. translate some text), use `POST` requests. The request is routed to a provider that supports the input data. In order to control the provider selection process, you may specify the bidding strategy or identifier of the particular provider to use.
 
All methods of the Intento API return JSON responses.

Some of the features described below are supported in the testing branch of our API, please contact us to enable these features for your API key.
 
### Error handling

Intento uses the standard HTTP error reporting format for the JSON API. Successful requests return HTTP status codes in the 2xx range. Failed requests return status codes in the 4xx and 5xx ranges. Requests that require a redirect returns status codes in the 3xx range. 

### Authentication and keys

Currently, we use the token-based authentication via headers (more on that below). If this is inconvenient for you, let us know. We always strive to support all use-cases our clients have in mind.
 
We enable cross-origin resource sharing support for our API so that you can use it right from the web application. However, be careful with your access keys. If you think your access credentials are compromised, let us know. We will give you new credentials and consult on how to take care of them.

### Paying for services

Third-party services may be paid via Intento (the default scenario) or via your own account at the third party service. In order to use your own account at the third party service, specify the account credentials as described below. NOTE: some of the services are available only with your own accounts (i.e. we don’t have a billing integration with them).

## Authentication

We use the key-based authentication. Each request to intento API should pass an access key in header “apikey” as demonstrated in the examples below.
 
For each account, we provide two keys, a real key and a sandbox one. Requests performed with the real key are actually fulfilled via third-party services and billed towards your account. Usage limits for real keys are governed by the subscription tier you have. Requests performed with the sandbox key are intended for testing purposes and return some sample responses. Usage limits for sandbox keys are quite low, let us know if anything is wrong with that.

## Usage limits

The following limits apply to the production keys:
- Requests per second: 100
- Requests per month: 1,000,000
- Data per request: 4MB
 
Remaining limits are returned with a response in headers: `X-RateLimit-Remaining-second`, `X-RateLimit-Remaining-month`

When the limits as exceeded, Intento API will return a HTTP error 429/413 (see below).

## Errors

Error responses usually include a JSON document in the response body, which contains information about the error.
 
Error codes:

* `400` -- Provider-related errors. 
* `401` -- Intento: Required parameter is missing
* `403` -- Intento: Auth key is invalid 
* `404` -- Intento: Intent/Provider not found
* `413` -- Intento: Request entity too large
* `429` -- Intento: API rate limit exceeded
* `500` -- Intento: Internal error
* `502` -- Intento: Gateway timeout

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
 "service": {
  "provider": {
   "id": "ai.text.translate.microsoft.translator_text_api.2-0",
   "name": "Microsoft Translator API"
  }
 }
}
```

## Getting available providers

To get a list of available Machine Translation providers, send a GET request to https://api.inten.to/ai/text/translate. 

```sh
curl -H 'apikey: key' 'https://api.inten.to/ai/text/translate'
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

### :lock: Advanced usage (beta)

The list of providers may be further constrained by adding desired parameter values to the GET request:

```sh
curl -H 'apikey: key' 'https://api.inten.to/ai/text/translate?from=en&to=es'
```

In case of multiple values passed per parameter, the conjunction of the constraints is used (i.e. the method returns all providers that meet all the constraints). The following request would return providers that support all four language pairs (en-es, ru-es, en-de, ru-de):

```sh
curl -H 'apikey: key' 'https://api.inten.to/ai/text/translate?from=en&from=ru&to=es&to=de'
```

Besides source and target language, service providers may be filtered by support of specific translation domains (`domain`), bulk translate option (`bulk`) and available billing via Intento (`billing`). Note that usually specific domains are supported for a small number of language pairs, thus it's better to supply the language pair altogether:

```sh
curl -H 'apikey: key' 'https://api.inten.to/ai/text/translate?from=en&to=es&domain=patent&bulk=true'
```

## Using a service provider with your own keys

*TBD*

## Smart routing

*TBD*
 

