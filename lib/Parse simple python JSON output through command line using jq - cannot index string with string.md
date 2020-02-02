### Query: [Parse simple python JSON output through command line using jq - “cannot index string with string”](https://stackoverflow.com/questions/60022403/parse-simple-python-json-output-through-command-line-using-jq-cannot-index-st)
([jump to the answer]())

I've been dancing around with this problem for a few hours and I haven't had any luck resolving it by looking up similar posts. Hopefully it's simple enough for a quick fix.

Essentially, I have a python script that retrieves a secret, dumps it as a json object to stdout.
There is a bash script that calls this python script, retrieves the output from the script and is intended to parse the username and password from the json output for use.

Simplified python code:
```
# test.py
payload = response.payload.data.decode('UTF-8').replace('\n', '')
# ^ equates to:
# payload = "{\"username: \"test\", \"password\": \"test\"}"
sys.stdout.write(json.dumps(payload))
```

Simplified bash code:
```
#!/bin/bash
credentials=$(python test.py)
echo $(credentials)
echo $(credentials) | jq .
echo $(credentials) | jq .username
```

Output:
```
"{\"username: \"test\", \"password\": \"test\"}"                         # for echo #1
"{\"username: \"test\", \"password\": \"test\"}"                         # for echo #2 with | jq .
jq: error (at <stdin>:1): Cannot index string with string "username"     # for echo #3
```

Instead of the error, I am expecting the third output to be:
```
test
```

#UPDATE
I was able to finally resolve this by doing something that feels a little hacky.
The modified bash code which fixes this issue is:
```
#!/bin/bash
credentials=$(python test.py | jq -r .)
echo $(credentials)
echo $(credentials) | jq .
echo $(credentials) | jq .username
```

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc) it's required first to convert a stringified JSON into a JSON and then
select the `username`:
```bash
bash $ echo '"{\"username\": \"test\", \"password\": \"test\"}"' | jtc -T'<<{{}}>>' / -w[username]
"test"
bash $ 
```






