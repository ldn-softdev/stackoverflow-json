### Query: [adding json objects from one file to another under single array using jq](https://stackoverflow.com/questions/59801518/adding-json-objects-from-one-file-to-another-under-single-array-using-jq)
([jump to the answer]())

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
Because the input here is a stream of JSONs, [`jtc`](https://github.com/ldn-softdev/jtc) cannot insert it using `-i` or `-u`, it then
must be processed as input, but it's not a probelm. Then the solution is: take the input JSON stream, JSONize it into an array,
insert into the resultng json the output file and preserve the final result in the output file:
```bash
# for the sake of example, let's assume that output.json looks like this:
bash $ jtc output.json 
[
   {
      "Age": "55",
      "Id": "3",
      "Name": "Blah"
   },
   {
      "Age": "66",
      "Id": "4",
      "Name": "Blah-Blah"
   }
]

# the solution:
bash $ <File_1.json jtc - -J / -mi output.json -f output.json
bash $ jtc output.json
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
      "Age": "55",
      "Id": "3",
      "Name": "Blah"
   },
   {
      "Age": "66",
      "Id": "4",
      "Name": "Blah-Blah"
   }
]
bash $ 
```


