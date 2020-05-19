### Query: [adding json objects from one file to another under single array using jq](https://stackoverflow.com/questions/59801518/adding-json-objects-from-one-file-to-another-under-single-array-using-jq)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/adding%20json%20objects%20from%20one%20file%20to%20another%20under%20single%20array%20using%20jq.md#a))

I am new here so sorry if I do any mistakes while asking the question.

I have a json file that keeps updating every minute(File_1.json) with json objects. All i want to do is copy these objects to another file under a single array using the jq command. 

Samples of files
File_1.json:

            {
              "Id":"1",
              "Name":"Kiran",
              "Age":"12"
            }
            {
              "Id":"2",
              "Name":"Dileep",
              "Age":"22"
            }

Expected Output

         [ 
           {
              "Id":"1",
              "Name":"Kiran",
              "Age":"12"
            }
            {
              "Id":"2",
              "Name":"Dileep",
              "Age":"22"
            }
           ]

I have tried using -s(slurp) but since the code will be running once for every minute its creating multiple arrays. 

### A:
As of _`v1.76`_ [`jtc`](https://github.com/ldn-softdev/jtc) is capable of processing a _stream of JSONs_ given as a file argument to either of options `-i` / `-u`. Given that
capability the query is super easy:  

let's start with an empty array in the `output.json` file:
```bash
bash $ <output.json jtc
[]
bash $ 
```

To add then to the file is trivial:
```bash
bash $ jtc -mi file_1.json -f output.json 
bash $ 
bash $ <output.json jtc
[
   {
      "Age": "12",
      "Id": "1",
      "Name": "Kiran"
   },
   {
      "Age": "22",
      "Id": "2",
      "Name": "Dileep"
   }
]
bash $ 
```

- `-m` ensures _merging_ of the input array (stream) of JSONs in `file_1.json` to the input file (`output.json`)
- `-f` forces / redirects the output into the input file `output.json` instead of <stdout>

If the same command executed once more - observe the extended array:
```bash
bash $ jtc -mi file_1.json -f output.json 
bash $ 
bash $ <output.json jtc
[
   {
      "Age": "12",
      "Id": "1",
      "Name": "Kiran"
   },
   {
      "Age": "22",
      "Id": "2",
      "Name": "Dileep"
   },
   {
      "Age": "12",
      "Id": "1",
      "Name": "Kiran"
   },
   {
      "Age": "22",
      "Id": "2",
      "Name": "Dileep"
   }
]
bash $ 
```


