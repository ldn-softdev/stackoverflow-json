### Query: [How to jq to copy array inside the dictionary?](https://stackoverflow.com/questions/60045218/how-to-jq-to-copy-array-inside-the-dictionary)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/How%20to%20jq%20to%20copy%20array%20inside%20the%20dictionary.md#a))

I have the following json

       [ 
           { 
              "name":"bucket1"
           },
           { 
              "name":"bucket1"
           }
        ]
I want to convert it to

    { 
       "buckets":[ 
          { 
             "name":"bucket1"
          },
          { 
             "name":"bucket1"
          }
       ]
    }

How do I do this with jq?

### A:
couple ways to do it with [`jtc`](https://github.com/ldn-softdev/jtc):
```bash
# 1. update your JSON into a template read from <stdin>:
bash $ <<<'{"buckets":0}' jtc -w[0] -u file.json
{
   "buckets": [
      {
         "name": "bucket1"
      },
      {
         "name": "bucket1"
      }
   ]
}
bash $ 

#2. wrap it using a template:
bash $ jtc -T'{"buckets":{{}}}' file.json
{
   "buckets": [
      {
         "name": "bucket1"
      },
      {
         "name": "bucket1"
      }
   ]
}
bash $ 

# and if you like to make the changes right into ths file:
bash $ jtc -T'{"buckets":{{}}}' -f file.json 
bash $ jtc file.json
{
   "buckets": [
      {
         "name": "bucket1"
      },
      {
         "name": "bucket1"
      }
   ]
}
bash $ 
```

