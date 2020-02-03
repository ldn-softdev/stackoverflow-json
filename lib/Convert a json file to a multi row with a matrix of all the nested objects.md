### Query: [Convert a json file to a multi row with a matrix of all the nested objects](https://stackoverflow.com/questions/59818977/convert-a-json-file-to-a-multi-row-with-a-matrix-of-all-the-nested-objects)
([jump to the answer]())

In order to use the Extended Choice Parameter Jenkins plugin, I need to create a file containing a matrix with several options, such as:

        Country     State   City
        USA         FL      Miami
        USA         FL      Tampa
        USA         FL      Jacksonville
        USA         NY      NYC
        USA         NY      Rochester
        USA         NY      Syracuse

Given this list could be quite a challenge to maintain, I thought of creating a json file, for example (:

    ["USA":       
     [{        
          "NY": [
                "NYC",
                "Rochester",
                "Syracuse"
        ],
        "FL": [
                "Miami",
                "Tampa",
                "Jacksonville",
       etc...

The question is how to convert a JSON file with many nested objects to a matrix in which the last column is always the deepest nested object? 

Alternatively, is there another way to keep the parameters' file maintainable?

I can use bash, python etc...

Thanks!        

### A:
this how it looks with [`jtc`](https://github.com/ldn-softdev/jtc): 
```bash
bash $ <file.json jtc -w' ' -T'"Country\tState\tCity"'  -w'[:]<C>k[0][:]<S>k[:]' -qqT'"{C}\t{S}\t{}"'
Country	State	City
USA	FL	Miami
USA	FL	Tampa
USA	FL	Jacksonville
USA	NY	NYC
USA	NY	Rochester
USA	NY	Syracuse
bash $ 
```

