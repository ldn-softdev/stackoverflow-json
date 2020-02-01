### Query: [jq: change multiple values within a select object](https://stackoverflow.com/questions/59904094/jq-change-multiple-values-within-a-select-object)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/jq%20-%20change%20multiple%20values%20within%20a%20select%20object.md#a))

I have an array of JSON objects, and I am trying change the `name` and `version` on the object of a given `@type`, with the following input
```
[
 {
    "name": "oldname",
    "version": "oldversion",
    "@type": "Project"
 },
 {
    "name": "bomname",
    "version": "bomversion",
    "@type": "BOM"
 },
 {
    "name": "componentname",
    "version": "componentversion",
    "@type": "Component"
 }
]
```

I found many examples for changing a single value, and I can successfully do this by chaining multiple `select` statements together. 

```
$ cat original.json | jq '[ .[] | (select(.["@type"] == "Project") | .name ) = "newname" | (select(.["@type"] == "Project") | .version ) = "newversion ] ' > renamed.json

```

But I was hoping I could condense this so I only have perform the `select` once to change both values.

### A:
Using [`jtc`](https://github.com/ldn-softdev/jtc) the query is easy to accomplish - find the record and update with the required
entries:
```bash
bash $ <original.json jtc -w'[@type]:<Project>[-1]' -mu'{"name":"newname","version":"newversion"}'
[
   {
      "@type": "Project",
      "name": "newname",
      "version": "newversion"
   },
   {
      "@type": "BOM",
      "name": "bomname",
      "version": "bomversion"
   },
   {
      "@type": "Component",
      "name": "componentname",
      "version": "componentversion"
   }
]
bash $  
```
