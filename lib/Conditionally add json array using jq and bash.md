### Query: [Conditionally add json array using jq and bash](https://stackoverflow.com/questions/59946450/conditionally-add-json-array-using-jq-and-bash)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Conditionally%20add%20json%20array%20using%20jq%20and%20bash.md#a))

Am trying to add a json array with just one array element to a json. And it has to be added only if it does not already exist. 
Example json is below.

    {
        "lorem": "2.0",
        "ipsum": {
            "key1": "value1",
            "key2": "value2"
        },
        "schemes": ["https"],
        "dorum" : "value3" 
    }
Above is the json, where `"schemes": ["https"],` exists. Am trying to add schemes only if it does not exist using the below code.

    scheme=$( cat rendertest.json | jq -r '. "schemes" ')
    echo $scheme
    schem='["https"]'
    echo "Scheme is"
    echo $schem
    if [ -z $scheme ]
    then
    echo "all good"
    else 
    jq --argjson argval  "$schem"  '. += { "schemes" : $schem }' rendertest.json > test.json
    fi

I get the below error in a file when the json array element 'schemes' does not exist. It returns a null and errors out. Any idea where am going wrong?

    null
    Scheme is
    ["https"]
    jq: error: schem/0 is not defined at <top-level>, line 1:
    . += { "schemes" : $schem }                   
    jq: 1 compile error


Edit: the question is not about how to pass on bash variables to jq.

### A:
using [`jtc`](https://github.com/ldn-softdev/jtc), it's quite simple:
```bash
bash $ schem='["https"]'
bash $ 
bash $ <example.json jtc -w'<>f[schemes][-1]' -i"{\"schemes\":$schem}"
{
   "dorum": "value3",
   "ipsum": {
      "key1": "value1",
      "key2": "value2"
   },
   "lorem": "2.0",
   "schemes": [
      "https"
   ]
}
bash $ 
```
\- insert would fail if `"schemes"` exists because inserting a clashing label with `-i` always fails - it always preserves
the existing one  
\- insert would work if `"schemes"` is missed because upon trying to walk `[schemes]` walking would fail and the _walk-path_ would
get reinstated at root (thanks to `<>f` lexeme) then the whole object under insert option `{\"schemes\":$schem}` would get inserted

Naturally, if in-place modification of the source file is required - use `-f` option:
```bash
bash $ jtc -w'<>f[schemes][-1]' -i"{\"schemes\":$schem}" -f example.json
```
