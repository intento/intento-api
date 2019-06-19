# Using delegated credentials

There are two ways how you can set up delegated credentials in Intento:
1) In a convenient [user interface](https://console.inten.to/credentials) 
2) Programatically via API - find instructions below in this document.

The fastest way to set up Google AutoML with Intento is to use our [Stored Credentials Manager](https://console.inten.to/credentials).

<!-- TOC depthFrom:2 depthTo:3 -->

- [Overview](#overview)
- [Supported providers](#supported-providers)
    - [Google AutoML](#google-automl)

<!-- /TOC -->

## Overview

Delegated credentials allow you to store provider authentication information at the Intento platform to avoid passing such information with each request.

Additionally, it provides a hassle-free way to work with temporary credentials (like tokens with limited lifetime).

## Supported providers

Currently, we support only Google Service Account type credentials.

### Google AutoML

Recent Google AutoML APIs do not support API key authentication but require you to use temporary tokens instead.

Here is a manual on how to use your custom models with delegated credentials.

The example uses Google AutoML Translate instance `ai.text.translate.google.automl_api.v1beta1`.

#### 1. [Train your custom model in Google AutoML Translate](https://cloud.google.com/translate/automl/docs/quickstart)

#### 2. Get the model id

You can get it from the example on the ‘Predict’ page in the AutoML Translation UI.

Model name looks like this `projects/automl-196010/locations/us-central1/models/8254482168020221643` and does not include “:” suffix or “/v1beta1/” prefix.

#### 3. Create a service account for accessing Google AutoML API

For this, you have to set up a Service Account as described [here](https://cloud.google.com/iam/docs/creating-managing-service-accounts#creating_a_service_account)

You can limit access rights for the service account to only perform predictions on the model. Full access (creating and deleting models) is not required.

#### 4. Download a service account key file (in JSON)

Save the JSON file with a service account credentials.

#### 5. Send your service account credentials to Intento

Right now it is possible to add delegated credentials using Intento API and curl commands, later we’ll add it into the user interface. Or you can send us the service account key file, and we’ll do it for you.

Choose a name for the credentials (you will use this name in the requests to let the system know which credentials to use) and put it into `credential_id` field.

Set `credential_type` field to `"google_service_account"`

Put the JSON structure you obtained from Google into the `secret_credentials` field.

The request looks like this:

```sh
curl -XPOST -H "apikey: $YOUR_API_KEY" "https://api.inten.to/delegated_credentials" -d '
{
    "credential_id": "credentials_for_project_x",
    "credential_type": "google_service_account",
    "secret_credentials": {
        "type": "service_account",
        "project_id": "automl-196010",
        "private_key_id": "xxxxxxx",
        "private_key": "-----BEGIN PRIVATE KEY-----\nAUNKNKDND......4F3==\n-----END PRIVATE KEY-----\n",
        "client_email": "automl-translation@automl-196010.iam.gserviceaccount.com",
        "client_id": "xxxxxxx",
        "auth_uri": "https://accounts.google.com/o/oauth2/auth",
        "token_uri": "https://accounts.google.com/o/oauth2/token",
        "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
        "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/automl-translation%40automl-196010.iam.gserviceaccount.com"
    }
}'
```

Intento API just returns the copy of your request if there were no errors.

#### 6. Check the credential is added

Get a list of available delegated credentials.

```sh
curl -XGET -H "apikey: $YOUR_API_KEY" "https://api.inten.to/delegated_credentials"

[
    {
        "credential_id": "credentials_for_project_x",
        "credential_type": "google_service_account",
        "created_at": "2018-07-09 11:15:29.959298+00:00",
        "temporary_credentials": null,
        "temporary_credentials_created_at": null,
        "temporary_credentials_expiry_at": null,
        "error_message": null
    }
]
```

If you didn’t add any delegated credentials, the result is an empty list:

`[]`

#### 7. Wait 1 minute for our system to take the credential and run a regular token generation process for it

The fields `temporary_credentials`, `temporary_credentials_created_at`, `temporary_credentials_expiry_at` will become non-empty:

```sh
curl -XGET -H "apikey: $YOUR_API_KEY" "https://api.inten.to/delegated_credentials"

[
    {
        "credential_id": "credentials_for_project_x",
        "credential_type": "google_service_account",
        "created_at": "2018-07-09 11:15:29.959298+00:00",
        "temporary_credentials": {
            "access_token": "ya29.c.ElrzBU_TmUy47vsnNLXlV1bCZZZZqqidANzT-vEt_BZFFmN1gKj75sJVzLoYTKeHKBNfm7ff7nlNvKMjjD3TwZiUh6sSoZZOX1pqq_G6NllWDazz9fmmLl8W0"
        },
        "temporary_credentials_created_at": "2018-07-09 15:54:58.311881+00:00",
        "temporary_credentials_expiry_at": "2018-07-09 16:54:58.310336+00:00",
        "error_message": null
    }
]
```

If any error happens during generating a token (it can happen if you passed invalid secret credentials, using the wrong format or if your service account is disabled), the message will appear in the `error_message` field.

#### 8. Now you can call Google passing it the credential_id

Intento automatically uses the most recent token to call the Google API:

```sh
curl -XPOST -H "apikey: $YOUR_API_KEY" "$HOST/ai/text/translate" -d '
{
    "context": {
        "text": "epigenetics markers for treatment in a hospital setting ...",
        "from": "en",
        "to": "pt",
        "category": "projects/automl-196010/locations/us-central1/models/8254482168020221643"
    },
    "service": {
        "provider": "ai.text.translate.google.automl_api.v1alpha1b",
        "auth": {
            "ai.text.translate.google.automl_api.v1alpha1b": [
                {
                    "credential_id": "credentials_for_project_x"
                }
            ]
        }
    }
}'
```

That’s all. Everything will work.

#### 9. You can delete your credentials by their id if you do not longer need them

```sh
curl -XDELETE -H "apikey: $YOUR_API_KEY" "https://api.inten.to/delegated_credentials/credentials_for_project_x"

{"status": "OK"}
```
