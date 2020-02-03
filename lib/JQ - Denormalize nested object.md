### Query: [JQ - Denormalize nested object](https://stackoverflow.com/questions/59824452/jq-denormalize-nested-object)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/JQ%20-%20Denormalize%20nested%20object.md#a))

I've been trying to convert some JSON to csv and I have the following problem:

I have the following input json:

    {"id": 100, "a": [{"t" : 1,"c" : 2 }, {"t": 2, "c" : 3 }] }
    {"id": 200, "a": [{"t": 2, "c" : 3 }] }
    {"id": 300, "a": [{"t": 1, "c" : 3 }] }

And I expect the following CSV output:

    id,t1,t2
    100,2,3
    200,,3
    300,3,


Unfortunately JQ doesn't output if one of select has no match.
Example:

    echo '{ "id": 100,  "a": [{"t" : 1,"c" : 2 }, {"t": 2, "c" : 3 }] }' | jq '{t1: (.a[] | select(.t==1)).c , t2: (.a[] | select(.t==2)).c }'

output:

    { "t1": 2,  "t2": 3   }

but if one of the objects select returns no match it doesn't return at all.
Example:

    echo '{ "id": 100,  "a": [{"t" : 1,"c" : 2 }] }' | jq '{t1: (.a[] | select(.t==1)).c , t2: (.a[] | select(.t==2)).c }' 


Expected output:

    { "t1": 2,  "t2": null   }


Does anyone know how to achieve this with JQ?

EDIT:

Based on a comment made by @peak I found the solution that I was looking for.

    jq -r '["id","t1","t2"],[.id, (.a[] | select(.t==1)).c//null, (.a[] | select(.t==2)).c//null ]|@csv'

The alternative operator does exactly what I was looking for.
[Alternative Operator][1]


  [1]: https://stedolan.github.io/jq/manual/#Alternativeoperator://
  
### A:
\- to accomplish the ask with [`jtc`](https://github.com/ldn-softdev/jtc): make individual walks for each entry 
which might go missing, ensuring that in such case an empty string (`""`) is the result, and then template-interpolate it:
```bash
bash $ <input.json jtc -jaw'<id>l<V>v' -T{{V}} -w'<V:>f[t]:<1>d<V>v' -w'<V:>f[t]:<2>d<V>v' / -qqT'"{}"'
100, 1, 2
200, , 2
300, 1, 
bash $ 
```

if the default spacer between commas is undesirable, then alter it by updating `$#` namespace:
```bash
bash $ <input.json jtc -jaw'<id>l<V>v' -T{{V}} -w'<V:>f[t]:<1>d<V>v' -w'<V:>f[t]:<2>d<V>v' / -qqT'"{}"' -w'<$#:,>v'
100,1,2
200,,2
300,1,
bash $ 
```

and with the header:
```bash
bash $ hdr='"id,t1,t2"'
bash $ <input.json jtc -Jjw'<id>l<V>v' -T{{V}} -w'<V:>f[t]:<1>d<V>v' -w'<V:>f[t]:<2>d<V>v' /\
                      -nw'<$#:,>v' -T$hdr -w[:] -qqT'"{}"' 
id,t1,t2
100,1,2
200,,2
300,1,
bash $ 
```


