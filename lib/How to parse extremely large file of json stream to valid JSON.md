### Query: [How to parse extremely large file of json stream to valid JSON](https://stackoverflow.com/questions/59962659/how-to-parse-extremely-large-file-of-json-stream-to-valid-json)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/How%20to%20parse%20extremely%20large%20file%20of%20json%20stream%20to%20valid%20JSON.md#a))

I'm working on a CS course project where I've to perform sentiment analysis of Twitter data on an Ubuntu VM. I was able to build a crawler to obtain the data but I have the output in the format of a file of JSON stream which is a very large file which is of the style:

    {
      "query": "#India_since_2019",
      "username": "user_1",
      "ID": "123455",
      "tweet": "This is the tweet",
      "datetime": "2019-04-05"
    }
    {
      "query": "#India_since_2019",
      "username": "user_1",
      "ID": "123455",
      "tweet": "This is the tweet",
      "datetime": "2019-04-05"
    }

and so on.

I essentially have to filter results based on year and store the resultant json file.

The output I'm looking to get is
```
[
{
      "query": "#India_since_2019",
      "username": "user_1",
      "ID": "123455",
      "tweet": "This is the tweet",
      "datetime": "2019-04-05"
    },
    {
      "query": "#India_since_2019",
      "username": "user_1",
      "ID": "123455",
      "tweet": "This is the tweet",
      "datetime": "2019-04-05"
    }
]
```

This prevents me from reading the file line by line as all the data is appended thus not creating any new lines. 

I tried using jq to parse the data but the file was too large and thus created errors.

Would you guys have any suggestions for how I can convert this easily to a valid JSON and write to another file?

I'd be open to solutions in any script as I'm flexible with those, although I'd prefer an idea I can work with in Python/Shell.

Thanks!

### A:
\- for [`jtc`](https://github.com/ldn-softdev/jtc) it does not matter how big input JSON file is - as long (virtual) memory allows,
it'll be processed (i.e. `jtc` won't fail because JSON is too big).

However, if it's required to process a stream of JSONs with the memory constrains, then it's best to make `jtc` to parse JSONs
in a _streamed_ way: `jtc` switches to a _streamed_ read once the input is <stdin> and option `-a` given, then the memory will be
consumed only for 1 JSON at most upon parsing:
```bash
bash $ cat stream.json | jtc -aw'[datetime]:<^2019>R[-1]' / -J
[
   {
      "ID": "123455",
      "datetime": "2019-04-05",
      "query": "#India_since_2019",
      "tweet": "This is the tweet",
      "username": "user_1"
   },
   {
      "ID": "123455",
      "datetime": "2019-04-05",
      "query": "#India_since_2019",
      "tweet": "This is the tweet",
      "username": "user_1"
   }
]
bash $ 
```

That way any number of JSON in the stream can be processed (does not matter how many JSONs are in the stream) and JSON matching 
the criteria will be retained for the output


