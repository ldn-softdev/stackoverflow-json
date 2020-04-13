### Query: [How to sort/unique output using jq](https://stackoverflow.com/questions/59849700/how-to-sort-unique-output-using-jq)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/How%20to%20sort%2Cunique%20output%20using%20jq.md#a))

I have json like below:

    % cat example.json
    {
        "values" : [
            {
                "title": "B",
                "url": "https://B"
            },
            {
                "title": "A",
                "url": "https://A"
            }
        ]
    }

I want to sort the values based on title. i.e. expected output 

    {
      "title": "A",
      "url": "https://A"
    }
    {
      "title": "B",
      "url": "https://B"
    }

Tried the blow. Does not work:

    % jq '.values[] | sort' example.json           
    jq: error (at example.json:12): object ({"title":"B...) cannot be sorted, as it is not an array

    % jq '.values[] | sort_by(.title)' example.json
    jq: error (at example.json:12): Cannot index string with string "title"
    
### A:
the query here is only about sorting records by the record's inner labels
1. to achieve the same output as in the question:
```bash
bash $ <example.json jtc -w'[title]:<>g:[-1]'
{
   "title": "A",
   "url": "https://A"
}
{
   "title": "B",
   "url": "https://B"
}
bash $ 
```

2. to sort and apply in-place modification into the source file:
```bash
bash $ jtc -w'<values>l' -pi'[title]:<>g:[-1]' -f example.json 
bash $ jtc example.json
{
   "values": [
      {
         "title": "A",
         "url": "https://A"
      },
      {
         "title": "B",
         "url": "https://B"
      }
   ]
}
bash $ 
```


