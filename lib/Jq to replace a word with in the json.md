### Query: [Jq to replace a word with in the json](https://stackoverflow.com/questions/59952122/jq-to-replace-a-word-with-in-the-json)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Jq%20to%20replace%20a%20word%20with%20in%20the%20json.md#a))

I have a json below and i need to fetch the value of the key "toModify" in data section. And also wanted to modify the key(toModify) value with another value Ex: hello1234567 to xyz. How can we do that using jq.

    {
      "items": [
        {
          "source": {
            "id": "12334"        
          },
          "data": {
            "name": "test",
            "value": "test",        
            "cData": [
              {
                "key": "keOne",
                "value": "hello"
              },
              {
                "key": "toModify",
                "value": "hello1234567"
              }
            ]
          } 
        }
      ]
    }

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc) the ask is trivial:
```bash
bash $ <file.json jtc -w'[key]:<toModify>[-1][value]' -u'"xyz"'
{
   "items": [
      {
         "data": {
            "cData": [
               {
                  "key": "keOne",
                  "value": "hello"
               },
               {
                  "key": "toModify",
                  "value": "xyz"
               }
            ],
            "name": "test",
            "value": "test"
         },
         "source": {
            "id": "12334"
         }
      }
   ]
}
bash $ 
```
And if in-place file modification is required, then use:
```bash
bash $ jtc -w'[key]:<toModify>[-1][value]' -u'"xyz"' -f file.json
```
