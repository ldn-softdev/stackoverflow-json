### Query: [Generating a JSON map containing shell variables named in a list](https://stackoverflow.com/questions/59778578/generating-a-json-map-containing-shell-variables-named-in-a-list)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Generating%20a%20JSON%20map%20containing%20shell%20variables%20named%20in%20a%20list.md#a))

My shell-fu is at a below-beginner level. I have a file that contains some lines that happen to be the names of environment variables.


e.g.
```
ENV_VAR_A
ENV_VAR_B
...
```


What I want to do is use this file to generate a JSON string containing the names and current values of the named variables using jq like this:

```
jq -n --arg arg1 "$ENV_VAR_A" --arg arg2 "$ENV_VAR_B" '{ENV_VAR_A:$arg1,ENV_VAR_B:$arg2}'

# if ENV_VAR_A=one and ENV_VAR_B=two then the preceding command would output 
# {"ENV_VAR_A":"one","ENV_VAR_B":"two"}
```

I'm trying to create the jq command through a shell script and I have no idea what I'm doing :(

### A:
[`jtc`](https://github.com/ldn-softdev/jtc) being a JSON utility, accepts as input only in `JSON` form, while the requestor's input file is clearly not the one.
With `jtc` then it's required first to transform input into a stream of JSONs, which is easy:
```bash
bash $ <tmp.txt sed -E 's/(.*)/{"\1": 0}/' 
{"ENV_VAR_A": 0}
{"ENV_VAR_B": 0}
{"ENV_VAR_C": 0}
bash $ 
```
and  once converted, `jtc` solution to transform it into the desired format:
```bash
bash $ export ENV_VAR_A='first token'
bash $ export ENV_VAR_B='2nd token'
bash $ 
bash $ <tmp.txt sed -E 's/(.*)/{"\1": 0}/' | jtc -Jw'[0]<L>k' -eu echo '\""${L}"\"' \; / -w[:][0] -ljj
{
   "ENV_VAR_A": "first token",
   "ENV_VAR_B": "2nd token",
   "ENV_VAR_C": ""
}
bash $ 
```

**Explanation**:
- the first option-set `jtc -Jw'[0]<L>k' -eu echo '\""${L}"\"' \;`: memorizes the label of the input JSON (which is env. var) and
updates the value with the shell-evaluated output for the given variable (combining all the processed outputs in one JSON array): 
```bash
bash $ <tmp.txt sed -E 's/(.*)/{"\1": 0}/' | jtc -Jw'[0]<L>k' -eu echo '\""${L}"\"' \; 
[
   {
      "ENV_VAR_A": "first token"
   },
   {
      "ENV_VAR_B": "2nd token"
   },
   {
      "ENV_VAR_C": ""
   }
]
bash $ 
```
- the second one `-w[:][0] -ljj`: walks each object's value (`[:][0]`) and join them together by labels (`-l`) into a JSON object (`-jj`)





