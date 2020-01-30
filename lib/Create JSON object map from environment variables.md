### Query: [Create JSON object map from environment variables](https://stackoverflow.com/questions/59985760/jq-create-json-object-map-from-environment-variables)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Create%20JSON%20object%20map%20from%20environment%20variables.md#a))

I am using jq 1.5 and trying to pass two environment variables via jq to create a json object:

```
export REGIONS="region1,region2"
export KMS_KEYS="key1,key2"
```

test.json
```
{
    "builders": [
    {
      "name": "aws"
    }
  ]
}
```


using the following command:
```
jq --arg regions $REGIONS --arg kmskeys $KMS_KEYS '.builders[].region_kms_key_ids={$regions}' test.json
```

current outcome:
```
{
  "builders": [
    {
      "name": "aws",
      "region_kms_key_ids": {
        "regions": "region1,region2"
      }
    }
  ]
}
```


desired outcome:
```
{
    "builders": [
    {
      "name": "aws",
      "region_kms_key_ids": {
        "region1": "key1",
        "region2": "key2"
      }
    }
  ]
}
```

I'm stuck on how to use my REGIONS variable content as the keys and KMS_KEYS variable as the values. Any advice would be appreciated

### A:
[`jtc`](https://github.com/ldn-softdev/jtc) accepts as input only JSON values, so, once the env variables declared as valid JSONs,
the solution becomes feasible:
```
bash $ export REGIONS='["region1","region2"]'
bash $ export KMS_KEYS='["key1","key2"]'
bash $ 
bash $ <<<$REGIONS jtc -w'[:]<R>v' -mu"$KMS_KEYS" -u[:] -T'{"{R}":{{}}}' /\
                       -llw[:] /\
                       -w'<J>v' -u test.json /\
                       -w[builders][0] -i0 -T'{"region_kms_key_ids":{{J}}}'
{
   "builders": [
      {
         "name": "aws",
         "region_kms_key_ids": {
            "region1": "key1",
            "region2": "key2"
         }
      }
   ]
}
bash $ 






