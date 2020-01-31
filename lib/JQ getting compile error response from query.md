### Query: [JQ getting compile error response from query](https://stackoverflow.com/questions/59995900/jq-getting-compile-error-response-from-query)
([jumpt to the answer]())

I would like to ask for your guidance for newbie like me on testing query on a sample file. I'm getting  compile error while testing it. Please advice on what is needed to adjust.

JSON FILE --> sample.json

        {
        "lambda": {
            "apple": [
                {
                    "type": "fruit",
                    "color": "red",
                    "shape": "round",
                    "cron": "0/10 * * * ? *"
                }
            ],
            "orange": [
                {
                    "type": "fruit",
                    "color": "orange",
                    "shape": "round",
                    "cron": "0/15 * * * ? *"
                }
            ],
            "apple-cider": [
                {
                    "type": "juice",
                    "color": "pink",
                    "shape": "none",
                    "cron": "0/30 * * * ? *"
                }
            ]
        }
    }

I'm getting this *jq: 1 compile error* message when I tried to get data from **apple-cider**

    # jq -r ".lambda."apple-cider"[].shape" test.json
    jq: error: cider/0 is not defined at <top-level>, line 1:
    .lambda.apple-cider[].shape
    jq: 1 compile error
    
    
### A:
Even though question is jq specific, the JSON operation is generic. Namely, it's a matter of digging(addressing) through a JSON tree.
With [`jtc`](https://github.com/ldn-softdev/jtc), it's a trivial task:
```bash
bash $ <sample.json jtc -w'[lambda][apple-cider][0][shape]'
"none"
bash $ 
```


 
