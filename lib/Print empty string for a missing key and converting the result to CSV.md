### Query: [Print empty string for a missing key and converting the result to CSV](https://stackoverflow.com/questions/59960742/print-empty-string-for-a-missing-key-and-converting-the-result-to-csv)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Print%20empty%20string%20for%20a%20missing%20key%20and%20converting%20the%20result%20to%20CSV.md#a))

I am trying to convert belowJSON to CSV using jq command but the final CSV is unable to place deviceName field properly as it's missing in some JSON lines.

    {
      "id": "ABC",
      "deviceName": "",
      "total": 100,
      "master": 20
    }
    {
      "id": "ABC",
      "total": 100,
      "master": 20
    }


How can i make sure empty value gets when Key is missing ?.

I Tried below command to generate CSV

    ./jq -r '[.[]] | @csv' > final.csv

But it gives CSV like below as you can see when deviceName key is missing in JSON it's cell shifting left side.

    "ABC","",100,20
    "ABC",100,20

I want output something like below which adds empty value if deviceName is missing.

    "ABC","",100,20
    "ABC","",100,20
    
### A:
To achieve the ask here the best way: would be ensure that all the entries have all the default entries inserted (if any is missed) and 
then use template to interpolate the result.

I took a liberty of extending the example with a few additional entries missing some values and reshuffling the object entries:
```bash
bash $ cat file.json
{
  "id": "ABC",
  "deviceName": "",
  "total": 100,
  "master": 20
}
{
  "total": 100,
  "id": "ABC",
  "master": 20
}
{
  "id": "ABC",
  "deviceName": "",
  "master": 20
}
{
  "id": ""
}
bash $ 

# jtc solution:
bash $ <file.json jtc -ai'{"deviceName":"","id":"","master":0,"total":0}' /\
                      -qqT'"\"{$b}\", \"{$a}\", {$d}, {$c}"'
"ABC", "", 100, 20
"ABC", "", 100, 20
"ABC", "", 0, 20
"", "", 0, 0
bash $ 
```

1. the first option set will ensure that default values will be set (if any missed)
2. the second option set will interpolate each entry

> Note a benign side-effect: because `jtc` sorts the entries within _JSON objects_ alphabetically, it becomes possible to operate on
the entries deterministically, even if those are shuffled in the input source (like in the shown example).


