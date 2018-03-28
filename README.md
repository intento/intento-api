# API User Manual

## Overview

This is a human-friendly manual to the Intento API. The interactive API Reference is available at https://intento.github.io.

### Intents

Intento API provides a uniform intent-based access to cognitive services from multiple providers, available at the Intento Service Platform. The intent is a basic action (like `translate`). 

Currently, the following intents are available:
* [Dictionary][ai.text.dictionary] `ai.text.dictionary`
* [Text translation][ai.text.translate] `ai.text.translate`
* [Sentiment analysis][ai.text.sentiment] `ai.text.sentiment`
 
Intent APIs are accessed via the base path. That said, API for the Machine Translation intent (ai.text.translate) is available at `/ai/text/translate`.

### Using the API

In order to get a list of providers available for some intent, use `GET` requests. Some providers support a subset of all available inputs for an intent. For example, not every Machine Translation service supports Korean to Russian language pair. To get a list of providers that support particular inputs, specify the input data constraints in GET parameters (see below).
 
In order to fulfill an intent (e.g. translate some text), use `POST` requests. The request is routed to a provider that supports the input data. In order to control the provider selection process, you may specify the bidding strategy or identifier of the particular provider to use.
 
All methods of the Intento API return JSON responses.

Some of the features described below are supported in the testing branch of our API. They are marked by :lock:. Please contact us to enable these features for your API key.
 
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

When the limits as exceeded, Intento API will return a HTTP error 429 (see below).

## Errors

Error responses usually include a JSON document in the response body, which contains information about the error.
 
Error codes:

* `400` -- Provider-related errors. 
* `401` -- Intento: Auth key is missing
* `403` -- Intento: Auth key is invalid 
* `404` -- Intento: Intent/Provider not found
* `413` -- Intento: Capabilities mismatch for the chosen provider (too long text, unsupported languages, etc)
* `429` -- Intento: API rate limit exceeded
* `500` -- Intento: Internal error
* `502` -- Intento: Gateway timeout

## Basic usage

A simple example of how to use Intento API.

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

Proceed to the following links for more details and examples on how to use a particular intent.

* [Text translation (ai.text.translate)][ai.text.translate]
* [Dictionary (ai.text.dictionary)][ai.text.dictionary]
* [Sentiment analysis (ai.text.sentiment)][ai.text.sentiment]

## Advanced usage

### :lock: Multi mode
In the multi mode, the request is performed using a list of providers. The mode is activated by passing an array of provider identificators.

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

### :lock: Using a service provider with your own keys

Intento supports two modes of using 3rd party services: 

* full proxy: 3rd party service used via Intento and paid to the Intento (available for some of the services).
* tech proxy: 3rd party service used via our user's own credentials and billed towards the user's account at the third-party service (available for all services).

In the tech proxy mode, the custom credentials are passed in the `auth` service field. `auth` field is a dictionary with keys equal to provider IDs you want to use your own key(s) with, and values set to a list of keys for the specified provider. There could be more than one key for the provider in the case you want to work with a pool of keys (more on this advanced feature later).

Auth object structure is different for different providers and may be obtained together with other provider details by sending GET request for this provider:

```sh
curl -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate?provider=ai.text.translate.google.translate_api.2-0'
[{
    "id": "ai.text.translate.google.translate_api.2-0",
    "name": "Google Cloud Translation API",
    "auth": {key: ""},
    "bulk_mode": true,
    "score": 0,
    "price": 0
  }]
```

```sh
curl -XPOST -H 'apikey: YOUR_INTENTO_KEY' 'https://api.inten.to/ai/text/translate' -d '{
 "context": {
 "text": "A sample text",
  "to": "es"
 },
 "service": {
  "provider": ["ai.text.translate.google.translate_api.2-0", "ai.text.translate.yandex.translate_api.1-5"]
  "auth": {
   "ai.text.translate.google.translate_api.2-0": [
     { "key": YOUR_GOOGLE_KEY }
   ],
   "ai.text.translate.yandex.translate_api.1-5": [
     { "key": YOUR_YANDEX_KEY1 },
     { "key": YOUR_YANDEX_KEY2 }
   ]
  }
 }
}'
```

### :lock: Smart routing

Intento provides the smart routing feature, so that the translation request is automatically routed to the best provider. The best provider is determined based on the following information:
- apriori benchmark on the standard dataset
- provider usage statistics, collected by Intento, including user feedback signals (the post-editing complexity for Machine Translation).

#### :lock: Basic smart routing
To use the smart routing, just omit the `service.provider` parameter:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
 "context": {
 "text": "A sample text",
  "to": "es"
 }
}'

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

#### :lock: Specifying the bidding strategy
By default, when the provider is missing, requests are routed to a provider with the best expected price/performance ratio. This behavior may be controlled by specifying the desired bidding strategy in the `service.bidding` parameter. Supported values are:
* `best` (default)
* `best_quality` - the best expected quality regardless of the price
* `best_price` - the cheapest provider
* `random` - a randomly chosen provider

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
 "context": {
 "text": "A sample text",
  "to": "es"
 },
 "service": {
  "bidding": "best_quality"
 }
}'
```

### :lock: Failover mode
Both for smart routing mode and basic mode, a failover is supported. By default, the failover is off, thus when the selected provider fails, an error is returned. To enable the failover mode, set the `service.failover` to `true`. By default, failover is governed by the default bidding strategy (`best`). To control this behavior, another bidding strategy may be specified via `service.bidding` parameter. Alternatively, you may specify a list of providers to consider for the failover (`service.failover_list`). This option overrides the bidding strategy for the failover procedure.

In the following example we set the provider, but specify the list of alternatives to control the failover:

```sh
curl -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
 "context": {
 "text": "A sample text",
  "to": "es"
 },
 "service": {
  "provider": "ai.text.translate.microsoft.translator_text_api.2-0",
  "failover": true,
  "failover_list": ["ai.text.translate.google.translate_api.2-0", "ai.text.translate.yandex.translate_api.1-5"]
 }
}'
```

### :lock: Sending feedback signals
In order to fine tune the smart routing algorithm, users may send the translation quality feedback. 

For each translation, Intento returns its unique id in `service.id` field:

```json
{
 "results": ["Un texto de ejemplo"],
 "service": {
  "id": "AVsAnRaMhIu3DcCXPJYY",
  "provider": {
   "id": "ai.text.translate.microsoft.translator_text_api.2-0",
   "name": "Microsoft Translator API"
  }
 }
}
```

This id may be used to send back the translation quality feedback:

```sh
curl -XPUT -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/ai/text/translate' -d '{
 "id": "AVsAnRaMhIu3DcCXPJYY",
 "feedback": {
  "type": "raw",
  "value": "A better text"
 }
}'
```

We support several types of the feedback:
* `raw` - raw post-edited text
* `fuzzy_word` - word-based inverted Levenstein distance
* `fuzzy_char` - character-based inverted Levenstein distance
* `time` - time spent on editing in seconds
* `effort` - an amount of mouse clicks and keyboard strokes spent on editing

### :lock: Session-based routing
The feedback signals sent back to Intento are used to fine-tune smart routing for all translations within language pair and domain for the user who sent the feedback.

To control this process, a user may specify the session as a `service.session` object with arbitrary string fields. We assume that sessions form some sort of hierarchy and do our best to infer it from the session data sent to Intento. Possible sessions may include: 

```json
{ 
  "translator": "John Reed",
  "author": "Leo Tolstoy",
  "project": "War and Peace",
  "document": "Chapter one" 
}
```

or 

```json
{ 
  "author": "Leo Tolstoy",
  "project": "Anna Karenina" 
}
```

[ai.text.translate]: <ai.text.translate.md>
[ai.text.dictionary]: <ai.text.dictionary.md>
[ai.text.sentiment]: <ai.text.sentiment.md>
