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
bash $ <input.json jtc -jaw'<id>l<V>v' -T{{V}} -w'<V:>f[t]:<1>d[-1][c]<V>v' -w'<V:>f[t]:<2>d[-1][c]<V>v' / -qqT'"{}"'
100, 2, 3
200, , 3
300, 3, 
bash $ 
```

if the default spacer between commas is undesirable, then alter it by updating `$#` namespace:
```bash
bash $ <input.json jtc -jaw'<id>l<V>v' -T{{V}} -w'<V:>f[t]:<1>d[-1][c]<V>v' -w'<V:>f[t]:<2>d[-1][c]<V>v' / -qqT'"{}"' -w'<$#:,>v'
100,2,3
200,,3
300,3,
bash $ 
```

and with the header:
```bash
bash $ hdr='"id,t1,t2"'
bash $ <input.json jtc -Jjw'<id>l<V>v' -T{{V}} -w'<V:>f[t]:<1>d[-1][c]<V>v' -w'<V:>f[t]:<2>d[-1][c]<V>v' /\
>                        -w'<$#:,>v' -T$hdr -w[:] -qqT'"{}"' 
id,t1,t2
100,2,3
200,,3
300,3,

bash $ 
```

**Explanation:**
1. the first option-set is made of 3 walks and one (common) template (`-T{{V}}`) - the template just iterpolates an entry
from the namespace `V`. Each walk sets the namespave `V` differently:
    - `-w'<id>l<V>v'`: finds an entry by the label (`<id>l`) and sets the namespace `V` to the found entry
    - `-w'<V:>f[t]:<1>d[-1][c]<V>v'`: sets the branching point and also sets the namespace `V` to an empty value (`<V:>f`), then:
      - if value `"t":1` found (`[t]:<1>d`) then select its sibling `c` (`[-1][c]`) and record the value into the namespace `V` (`<V>v`)
      - else (the value is not found) just breaks out of the walk, effecively leaving the namespace `V` empty (`""`)
    - `-w'<V:>f[t]:<2>d[-1][c]<V>v'`: does the same as the prior walk just for the entry `"t":2`
    - `-Jj` ensures that each walking each entry in the input JSON stream is jsonized int array and then all entries also jsonized,
    so the output of the 1st option-set is this:
    ```bash
    [
       [ 100, 2, 3 ]
       [ 200, "", 3 ]
       [ 300, 3, "" ]
    ]
    ```
2. the second option-set:
    - `-w'<$#:,>v' -T$hdr`: the walk sets the internal namespace constrollig the separator for the array interpolations (`$#`)
    to the custom value `","` (instead of default `", "`) and the template for this walk prints the header (`-T$hdr`)
    - `-w[:] -T'"{}"'`: goes over each array entry (in the output form the 1st option-set) and interpolates the array into the string,
    producting the output:
    ```bash
    "id,t1,t2"
    "100,2,3"
    "200,,3"
    "300,3,"
    ```
    - `-qq` drops the outer quotation marks      
  


