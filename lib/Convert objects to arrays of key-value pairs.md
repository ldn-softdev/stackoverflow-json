### Query: [Convert objects to arrays of key-value pairs](https://stackoverflow.com/questions/59974811/convert-objects-to-arrays-of-key-value-pairs)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Convert%20objects%20to%20arrays%20of%20key-value%20pairs.md#a))

I have a file where every line is a json (not formatted) as follows:
```
{
  "callID": "xxxxxxxxxxxxxx",
  "authType": "xxxxxxxxxxxxxx",
  "timestamp": "xxxxxxxxxxxxxx",
  "errCode": "0",
  "errMessage": "xxxxxxxxxxxxxx",
  "endpoint": "xxxxxxxxxxxxxx",
  "userKey": "xxxxxxxxxxxxxx",
  "httpReq": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  },
  "ip": "xxxxxxxxxxxxxx",
  "params": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  },
  "uid": "xxxxxxxxxxxxxx",
  "apikey": "xxxxxxxxxxxxxx",
  "userAgent": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  },
  "userKeyDetails": {
    "name": "xxxxxxxxxxxxxx"
  }
}
```

I need to perform a conversion where every object (`httpReq`, `params`, `userAgent`, `userKeyDetails`) need to be converted as array of objects, with `key` and `value` properties. Every key is not mandatory, a single json may NOT have all given keys.

Here is a partial output of the structure:
```
{
  "httpReq": [
    {
      "key": "key1",
      "value": "value1"
    },
    {
      "key": "key2",
      "value": "value2"
    }
  ]
}
```

Using `jq` command line I understand that `to_entries` operator is the one I'm looking for, so I created this command
```
cat test.json | jq -c '.userAgent = (.userAgent | to_entries) | .userKeyDetails = (.userKeyDetails | to_entries) | .params = (.params | to_entries) | .httpReq= (.
httpReq | to_entries)' > out.json
```
It works, but it's failing on rows where one of the given key is missing, with following error:
```
jq: error (at <stdin>:2): null (null) has no keys
jq: error (at <stdin>:3): null (null) has no keys
jq: error (at <stdin>:4): null (null) has no keys
jq: error (at <stdin>:5): null (null) has no keys
```

So I need a selector which works handling the possibility of a key to be missing, can this be obtained directly with a jq selector?

### A:
[`jtc`](https://github.com/ldn-softdev/jtc)
does not have a predefined template of function to auto-map objects to arrays with well-known keys, so such mapping 
has to be provided by user via a template:
```bash
bash $ <test.json jtc -w'<httpReq|params|userAgent|userKeyDetails>L:' -i'{"":[]}' /\
                      -w'<>l:[-1][1:][-1][]' -pi'<>l:[-1][1:]<K>k' -T'{"key":{{K}},"value":{{}}}' /\
                      -sw'<>l:' -w'<>l:[-1]' -tc
{
   "apikey": "xxxxxxxxxxxxxx",
   "authType": "xxxxxxxxxxxxxx",
   "callID": "xxxxxxxxxxxxxx",
   "endpoint": "xxxxxxxxxxxxxx",
   "errCode": "0",
   "errMessage": "xxxxxxxxxxxxxx",
   "httpReq": [
      { "key": "key1", "value": "value1" },
      { "key": "key2", "value": "value2" },
      { "key": "key3", "value": "value3" }
   ],
   "ip": "xxxxxxxxxxxxxx",
   "params": [
      { "key": "key1", "value": "value1" },
      { "key": "key2", "value": "value2" },
      { "key": "key3", "value": "value3" }
   ],
   "timestamp": "xxxxxxxxxxxxxx",
   "uid": "xxxxxxxxxxxxxx",
   "userAgent": [
      { "key": "key1", "value": "value1" },
      { "key": "key2", "value": "value2" },
      { "key": "key3", "value": "value3" }
   ],
   "userKey": "xxxxxxxxxxxxxx",
   "userKeyDetails": [
      { "key": "name", "value": "xxxxxxxxxxxxxx" }
   ]
}
```
1. the first option set will prepare-insert into each required label an entry `"":[]` to collect all the converted objects
2. the second optiion set will move each entry into the prepared array for all due labels. at the same time trasnforming moved entires
into `key`/`value` pairs (using the template)
3. the third option set will move the array upwards effectively attaching those to the labels

Of course, if there're no other labels, or object in the source json, then it could be shortened to:
```bash
bash $ <test.json jtc -w'<>o1:' -i'{"":[]}' /\
                      -w'<>l:[-1][1:][-1][]' -pi'<>l:[-1][1:]<K>k' -T'{"key":{{K}},"value":{{}}}' /\
                      -sw'<>l:' -w'<>l:[-1]' -tc
```






