### Query: [jq filter expression in string interpolation](https://stackoverflow.com/questions/59936373/jq-filter-expression-in-string-interpolation)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/jq%20filter%20expression%20in%20string%20interpolation.md#a))

I have been trying to reduce the array to a string to be used in string interpolation.

For example. 

```
input = ["123", "456"]
expected output = array=123,456
```

Here is my try

```
$ echo '["123", "456"]' | jq 'array=\(.|join(","))'
jq: error: syntax error, unexpected INVALID_CHARACTER (Unix shell quoting issues?) at <top-level>, line 1:
array=\(.|join(","))
jq: 1 compile error
```

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc):
```bash
bash $ echo '["123", "456"]' | jtc -qqT'"array={}"'
array=123, 456
bash $ 
```
If the spacer after comma (`, `) is redundant, then alter the _namespace_ `$#` holding a default separator for array stringification:
```bash
bash $ echo '["123", "456"]' | jtc -w'<$#:,>v' -qqT'"array={}"'
array=123,456
bash $ 
```
