### Query: [One-liner way to invert boolean output in bash?](https://stackoverflow.com/questions/59913511/one-liner-way-to-invert-boolean-output-in-bash)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/One-liner%20way%20to%20invert%20boolean%20output%20in%20bash.md#a))

Is there a short and clean one-liner way to invert a boolean in bash ?

I have a piped command as follows that looks at the sealed status of Hashicorp Vault:

`vault status -format json -address="https://127.0.0.1:8200" -tls-skip-verify | jq -r '.sealed'`

The problem is that if Vault is in a unsealed state, the value of `sealed` will be *false*, and hence the output from `jq -r '.sealed'` will be false and this will be interpreted as a boolean false in my config management.

So therefore I need to invert the output from `jq`, i.e `sealed=false => true and sealed=true=>false`.

I would like to do this in a clean manner at the end of the pipe becaue the piped command is being used by a config management system, not by a script.

### A:
the way to achieve the ask in [`jtc`](https://github.com/ldn-softdev/jtc) is to use branching:
```bash
bash $ <<<'{"sealed": true}' jtc -w'<s:true>f[sealed]<true>b<s:false>v' -T{s}
false
bash $ <<<'{"sealed": false}' jtc -w'<s:true>f[sealed]<true>b<s:false>v' -T{s}
true
bash $ 
```
the advantage of the above solution is that even if `sealed` is not present, it would still work (assuming _unsealed_ state):
```bash
bash $ <<<'{}' jtc -w'<s:true>f[sealed]<true>b<s:false>v' -T{s}
true
bash $ 
```
If such behavior is undesirable, then move `<>f` after `[sealed]`

