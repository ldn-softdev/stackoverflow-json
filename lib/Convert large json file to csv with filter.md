### Query: [Convert large json file to csv with filter](https://stackoverflow.com/questions/tagged/jq%2bor%2bjson%2bbash?tab=Newest)
([jump to the answer]())

I need a script which helps me basically look for all json files in a directory and extract only one field from each and write them to a second csv file.

This is what I have:

```
#!/bin/bash
files=`find /data/json/ -maxdepth 1 -name "#*.json"`
for file_name in $files
do
        echo $file_name
        temp_file=${file_name:10}
        new_file=${temp_file/json/csv}
        if [ ! -f /data/csv/$new_file ]; then
                jq -r '.[] | [.text] | @csv' $file > /data/csv/$new_file
        else
                echo "File already exists"
        fi
done
```

So I essentially use temp_file to remove the name of the folder where a file is and new_file to change the extension.



However, this program ends almost instantaneously only printing out the file names and generating empty csv files.



Any idea how I can resolve this?



Thanks!

### A:
using [`jtc`](https://github.com/ldn-softdev/jtc) there's no need to utilize `for` loop in bash, as it's easy to check all the files
at once:
```bash
bash $ jtc -w'<text>l:' -qqT'"{}"' '#'*.json
abc, 123
def, 456
bash $ 
```

