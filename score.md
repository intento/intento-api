# Score API (beta)

Score API is a collection of endpoints for calculating standard AI performance metrics on user-provided inputs.

### Machine Translation
To compare a machine translation with a reference translation, send a POST request to `https://api.inten.to/evaluate/score`. Specify data context keys: `items` - results of the machine translation,  `reference` -  ground truth and language.  List one or more currently implemented score functions, using `ignore_errors` flag to return result no matter of previous errors. Note that evaluation endpoints are supported only in `async` mode.

**Scores:**

- [hlepor](https://github.com/aaronlifenghan/aaron-project-hlepor)
- [bleu-corpus](https://github.com/moses-smt/mosesdecoder/blob/master/scripts/generic/multi-bleu-detok.perl)
- [bleu-sentence](https://github.com/odashi/mteval)
- [ribes](http://www.kecl.ntt.co.jp/icl/lirg/ribes/)
- [ter](https://github.com/jhclark/tercom)
- [sacrebleu](https://github.com/awslabs/sockeye/tree/3cca0c3ec397fbcb4c0ff0f51487e29338f53614/sockeye_contrib/sacrebleu)

#### Basic usage:


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
            "name": "bleu-corpus",
            "ignore_errors": true
        }
    ],
    "itemize": false,
    "async": true
}'
```

The response contains id of the operation:
```
{"id": "ea1684f1-4ec7-431d-9b7e-bfbe98cf0bda"}
 ```
 
Wait for processing to complete. To retrieve the result of the operation, make a GET request to the `https://api.inten.to/operations/YOUR_OPERATION_ID`. TTL of the resource is 30 days.

```json
{
    "id": "ea1684f1-4ec7-431d-9b7e-bfbe98cf0bda",
    "response": {
        "results": {
            "scores": [
                {
                    "name": "hlepor",
                    "value": 0.91
                },
                {
                    "name": "bleu-corpus",
                    "value": 0.95
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
                [
                    {
                        "name": "hlepor",
                        "value": 0.85
                    },
                    {
                        "name": "hlepor",
                        "value": 0.95
                    }
                ],
                [
                    {
                        "name": "bleu-corpus",
                        "value": 0.89
                    },
                    {
                        "name": "bleu-corpus",
                        "value": 0.99
                    }
                ]
            ]
        }
    },
    "error": null,
    "done": true
}
```