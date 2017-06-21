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
