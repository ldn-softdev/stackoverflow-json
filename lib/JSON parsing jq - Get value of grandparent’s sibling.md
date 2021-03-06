### Query: [JSON parsing [`jq`]: Get value of grandparent’s sibling](https://stackoverflow.com/questions/59813234/json-parsing-jq-get-value-of-grandparent-s-sibling)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/JSON%20parsing%20jq%20-%20Get%20value%20of%20grandparent%E2%80%99s%20sibling.md#a))

### Example

```json
{
  "gp_sibling": "desired_value",
  "grand-parent": {
    "parent": [
      {
        "key": "known_value",
        "sibling": "asdf"
      }
    ]
  }
}
```

### Problem definition

I have a large JSON. I know that `key` has a unique value. I need to get it’s grand-parent’s sibling key value (the value of `gp_sibling` key).

Now, I could `grep` the JSON, but I’d like to use `jq`. But I don’t know to achieve this (I use `jq` for simple queries only).

### Notes

Although I know that `jq` can be used on Windows too, I use it on Linux only.

### Post updates

1. Removed commas from the example. Some formatting fixes.

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc) the query is trivial:
```bash
bash $ <example.json jtc -w'[key]:<known_value>[-4][gp_sibling]'
"desired_value"
bash $ 
```

walk path explained:
- `[key]:<known_value>`: find `known_value` attached to the label `"key"`
- `[-4]`: go 4 parents up (the json tree from the found value)
- `[gp_sibling]` - address by `"gp_sibling"` label

