### Query: [I want to represent json data In Column format](https://stackoverflow.com/questions/59896979/i-want-to-represent-json-data-in-column-format)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/I%20want%20to%20represent%20json%20data%20In%20Column%20format.md#a))

```
[
  {
    "status": 200,
    "url": "https://url1",
    "length": 20429
  },
  {
    "status": 401,
    "url": "https://url2",
    "length": 30890
  }
]
```

I have data in this json format.I want to represent it in the following way :
```
200 20429 https://url1
401 30890 https://url2
```
I am not good with jq and formatting. But I want to represent the json data in the column format. So it will be very easy to read a long list or urls and their response length and status codes. Any help is appreciated.

### A:
\- with [`jtc`](https://github.com/ldn-softdev/jtc) it's a matter of dumping each record with the requested entries order:
```bash
bash $ <file.json jtc -w'<$#: >v[:]' -qqT'"{}"'
20429 200 https://url1
30890 401 https://url2
bash $ 
```
If the order to be respected:
```bash
bash $ <file.json jtc -w[:] -qqT'"{$b} {$a} {$c}"'
200 20429 https://url1
401 30890 https://url2
bash $ 
```

