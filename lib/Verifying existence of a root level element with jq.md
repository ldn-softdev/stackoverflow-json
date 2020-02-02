### Query: [Verifying existence of a root level element with jq](https://stackoverflow.com/questions/59861501/verifying-existence-of-a-root-level-element-with-jq)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Verifying%20existence%20of%20a%20root%20level%20element%20with%20jq.md#a))

Given a JSON structure such as below, I want to test for the existence of the `sys/` top-level key.


    {                                                                                                                                                             
      "abc/": {                                                                                                                                             
        "uuid": "fd"
       },
     "sys/": {                                                                                                                                                   
        "uuid": "id2"
      }
    }

I have tried various constructs such as `jq "sys/"` or `jq "sys/.uuid"` but I get compile errors and/or errors such as "jq: error: sys/0 is not defined at <top-level>, line 1:"

Note that it is important that the result code (`$?`) from `jq` returns correctly to reflect actual result (becuse this is being used in a config management test).

### A:
[`jtc`](https://github.com/ldn-softdev/jtc) does not let setting an arbitrary exit code (it uses own codes to indicate successful or
unsuccessful ending result of operation), however, it's possible to do relying on standard unix/linux tools:
```bash
bash $ if [ -z "$(jtc -w'[sys/]' file.json)" ]; then (exit 1); else (exit 0); fi
bash $ echo $?
0
bash $ if [ -z "$(jtc -w'[blah]' file.json)" ]; then (exit 1); else (exit 0); fi
bash $ echo $?
1
bash $ 
```
