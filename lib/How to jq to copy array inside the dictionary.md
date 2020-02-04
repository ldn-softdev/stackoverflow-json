### Query: [How to jq to copy array inside the dictionary?](https://stackoverflow.com/questions/60045218/how-to-jq-to-copy-array-inside-the-dictionary)
([jump to the answer]())

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
bash $ jtc -T'{"bukcets":{{}}}' file.json
{
   "bukcets": [
      {
         "name": "bucket1"
      },
      {
         "name": "bucket1"
      }
   ]
}
bash $ 

and if you like to make the changes right into ths file:
bash $ jtc -T'{"bukcets":{{}}}' -f file.json 
bash $ jtc file.json
{
   "bukcets": [
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

