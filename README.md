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
- Requests per second: 10
- Requests per month: 1,000,000
- Data per request: 4MB
 
When the limits as exceeded, Intento API returns an error (see below).

## Errors

### Application specific errors

Error responses usually include a JSON document in the response body, which contains information about the error.
 
Structure returned:

```javascript
‘error’ : {
‘code’: error code
‘message’ : error text
}
```
 
Example:

```javascript
{
 “code” : 401,
 “message” : “Required parameter ‘text’ is missing”
}
```
 
Codes:

* `400` -- Error returned by a service provider. 
* `401` -- Intento: Required parameter is missing
* `403` -- Intento: Authentication error
* `404` -- Intento: Intent/Provider not found
* `500` -- Intento: Internal error

*TBD Exceeding usage limits*

## Basic usage

To translate a text, send a POST request to Intento API at https://api.inten.to/ai/text/translate. Specify the source text, the target language and the desired translation provider in JSON body of the request as in the following example:

```sh
curl -XPOST -H ‘apikey: YOUR_API_KEY’ ‘https://api.inten.to/ai/text/translate’ -d ‘{
 “context”: {
 “text”: “A sample text”,
  “to”: “es”
 },
 “service”: {
  “provider”: “ai.text.translate.microsoft.translator_text_api.2-0”
 }
}’
```
 
The response contains the translated text and a service information:

```sh
{
 "results": ["Un texto de ejemplo"],
 "service": {
  "provider": {
   "id": "ai.text.translate.microsoft.translator_text_api.2-0",
   "name": "Microsoft"
  }
 }
}
```

## Getting available providers

To get a list of available Machine Translation providers, send a GET request to https://api.inten.to/ai/text/translate. Additional parameters, such as the language pair, are specified in GET query:

```sh
curl -H ‘apikey: key’ ‘https://api.inten.to/ai/text/translate’
```
 
The response contains a list of the providers available for given constraints with an information on pricing etc:

```sh
Intents loaded:
	ai.text.detect-intent
	ai.text.translate
Instances loaded:
	ai.text.detect-intent.amazon.lex
	ai.text.detect-intent.api.ai.1-0.20150910
	ai.text.detect-intent.ibm.watson.conversation.1-0.20170421
	...
 ```
 
## Using a provider with your own keys

*TBD*

## Smart routing

*TBD*
 

