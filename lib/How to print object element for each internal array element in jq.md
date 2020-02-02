Query: [How to print object element for each internal array element in jq](https://stackoverflow.com/questions/59879489/how-to-print-object-element-for-each-internal-array-element-in-jq)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/How%20to%20print%20object%20element%20for%20each%20internal%20array%20element%20in%20jq.md#a))

I have JSON that looks like this:

**test.json**
```
[
  {
    "item1": {
      "item2": 123,
      "array1": [
        {
          "item3": 456,
          "item4": "teststring"
        },
        {
          "item3": 789,
          "item4": "teststring2"
        }
      ]
    }
  }
]
```

I am trying to print it out in the following format:

```
123, 456, teststring
123, 789, teststring2
```
I tried to do it like this:
```
cat test.json | jq -r '.[].item1.item2, (.[].item1.array1[] | .item3, .item4)' | xargs printf "%s, %s, %s\n"
```
But the result is that `item2` is printed, followed by `item3` and `item4` in the first line, and then only `item3` and `item4` are printed in the next line, like so:
```
123, 456, teststring
789, teststring2, 
```

How can I make the element on the outside of the array print for every set of elements that are printed from within the array?

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc) the ask is quite trivial:
```bash
bash $ <test.json jtc -w'<item2>l<I>v[-1][array1][:]' -qqT'"{I}, {}"'
123, 456, teststring
123, 789, teststring2
bash $ 
```

