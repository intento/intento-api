# Score API (beta)

Score API is a collection of endpoints for calculating standard AI performance metrics on user-provided inputs.

## Machine Translation

To compare a machine translation with a reference translation, send a POST request to `https://api.inten.to/evaluate/score`. Specify data context keys: `items` - results of the machine translation,  `reference` -  ground truth and language.  List one or more currently implemented score functions, using `ignore_errors` flag to return result no matter of previous errors. Note that evaluation endpoints are supported only in `async` mode.

**Scores:**

- [hlepor](https://github.com/aaronlifenghan/aaron-project-hlepor)
- [ribes](http://www.kecl.ntt.co.jp/icl/lirg/ribes/)
- [ter](https://github.com/jhclark/tercom)
- [sacrebleu](https://github.com/awslabs/sockeye/tree/3cca0c3ec397fbcb4c0ff0f51487e29338f53614/sockeye_contrib/sacrebleu)
- [rouge](https://github.com/pltrdy/rouge)
- [chrf](https://github.com/mjpost/sacrebleu)
- [bert](https://github.com/Tiiiger/bert_score)

### Basic usage

```sh
curl  -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/evaluate/score'  -d '{
    "data": {
        "items": [
            "A sample text",
            "Some other text"
        ],
        "reference": [
            "Not a sample text",
            "Some other context"
        ],
        "lang": "en"
    },
    "scores": [
        {
            "name": "hlepor",
            "ignore_errors": true
        },
        {
            "name": "ter",
            "ignore_errors": true
        }
    ],
    "itemize": false,
    "async": true
}'
```

The response contains id of the operation:

```json
{ "id": "ea1684f1-4ec7-431d-9b7e-bfbe98cf0bda" }
```

Wait for processing to complete. To retrieve the result of the operation, make a GET request to the `https://api.inten.to/evaluate/score/YOUR_OPERATION_ID`. TTL of the resource is 30 days.

```json
{
    "id": "ea1684f1-4ec7-431d-9b7e-bfbe98cf0bda",
    "response": {
        "results": {
            "scores": [
                {
                    "name": "hlepor",
                    "value": 0.755
                },
                {
                    "name": "ter",
                    "value": 28.571
                }
            ]
        }
    },
    "error": null,
    "done": true
}
```

To get scores for each pair of items (machine translation and reference), set itemize flag to true.  In this case results would be:

```json
{
    "id": "ea1684f1-4ec7-431d-9b7e-bfbe98cf0bda",
    "response": {
        "results": {
            "scores": [
                {
                    "value": 0.770527098111673,
                    "name": "hlepor"
                },
                {
                    "value": 0.740740740740741,
                    "name": "hlepor"
                },
                {
                    "value": 25.0,
                    "name": "ter"
                },
                {
                    "value": 33.33333333333333,
                    "name": "ter"
                }
            ]
        }
    },
    "error": null,
    "done": true
}
```

## COMET

Crosslingual Optimized Metric for Evaluation of Translation ([COMET](https://github.com/Unbabel/COMET)) is metrics that achieve high levels of correlation with different types of human judgments. COMET takes 3 lists of strings as input: `sources` - a list of source sentences, `items` - results of the machine translation and `references` - ground truth and language. List one or more currently implemented score functions, using ignore_errors flag to return result no matter of previous errors. Note that evaluation endpoints are supported only in async mode. 

### Basic usage

```sh
curl  -XPOST -H 'apikey: YOUR_API_KEY' 'https://api.inten.to/evaluate/score'  -d '{
    "data": {
        "items": [
            "A sample text",
            "Some other text"
        ],
        "reference": [
            "Not a sample text",
            "Some other context"
        ],
        "source": [
            "Un texto de muestra",
            "Algún otro texto"
        ],
        "lang": "en"
    },
    "scores": [
        {
            "name": "comet",
            "ignore_errors": true
        }
    ],
    "itemize": false,
    "async": true
}'
```

The response contains id of the operation:

```json
{ "id": "c74934b3-89e9-463e-b358-335c7c717f02" }
```

Wait for processing to complete.

```json
{
    "id": "c74934b3-89e9-463e-b358-335c7c717f02",
    "done": true,
    "response": {
        "results": {
            "scores": [
                {
                    "value": {
                        "segment_scores": [
                            0.3271389603614807,
                            0.08198270201683044
                        ],
                        "corpus_scores": [
                            0.20456083118915558
                        ],
                        "return_hash": "wmt20-comet-da"
                    },
                    "name": "comet"
                }
            ]
        },
        "type": "scores"
    },
    "meta": {},
    "error": null
}
```

