### Query: [reformat objects to be tsv-formatted making table from json data](https://stackoverflow.com/questions/59973594/jq-object-cannot-be-tsv-formatted-only-array-error-when-making-table-from-jso)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/reformat%20objects%20to%20be%20tsv-formatted%20making%20table%20from%20json%20data.md#a))

My script creates a json data set and then tries to present it as a table. 

In the output for the first line of code, you can see an example data set:

> echo ${conn_list[@]} | jq '.'

```
{
  "host": {
    "name": "mike1",
    "node": "c04",
    "s_ip": "10.244.7.235",
    "s_port": "38558",
    "d_ip": "129.12.34.567",
    "d_port": "22",
    "pif_ip": "129.23.45.678",
    "pif_port": "11019"
  }
}
{
  "host": {
    "name": "fhlb-test",
    "node": "c04",
    "s_ip": "10.244.7.20",
    "s_port": "49846",
    "d_ip": "129.98.76.543",
    "d_port": "22",
    "pif_ip": "129.87.65.432",
    "pif_port": "23698"
  }
}
```

I use the following jq command with @tsv to try to build and fill the table but run into this error:

> echo ${conn_list[@]} | jq -r '["NAME","NODE","SOURCE IP","SOURCE PORT","DESTINATION IP","DESTINATION PORT","GATEWAY IP","GATEWAY PORT"], (.[], map(length*"-")), (.[] | [.name, .node, .s_ip, .s_port, .d_ip, .d_port, .pif_ip, .pif_port]) | @tsv'

```
NAME    NODE    SOURCE IP       SOURCE PORT     DESTINATION IP  DESTINATION PORT        GATEWAY IP      GATEWAY PORT
jq: error (at <stdin>:1): object ({"name":"mi...) cannot be tsv-formatted, only array
NAME    NODE    SOURCE IP       SOURCE PORT     DESTINATION IP  DESTINATION PORT        GATEWAY IP      GATEWAY PORT
jq: error (at <stdin>:1): object ({"name":"fh...) cannot be tsv-formatted, only array
```

My goal is to have only one column title row in the table instead of one for each entry, and of course to display the data instead of the error. The '(.[], map(length*"-"))' bit is meant to automatically generate right size dashes to separate the column titles from the data. Anyone see what I'm doing wrong? :) 

### A:
even though the question was specific for jq, the type of operation is typical for JSON.

with [`jtc`](https://github.com/ldn-softdev/jtc), the sulution looks like this:
```bash
bash $ echo ${conn_list[@]} | jtc -J / -w'<host>l:' -qqT'"{$c}\t{$d}\t{$g}\t{$h}\t{$a}\t{$b}\t{$e}\t{$f}"'
mike1	c04	10.244.7.235	38558	129.12.34.567	22	129.23.45.678	11019
fhlb-test	c04	10.244.7.20	49846	129.98.76.543	22	129.87.65.432	23698
bash $ 
```

If there's a requirement to add a header, it could be done using an additional template:
```bash
bash $ <file.json jtc -J / -nw' ' -T'"NAME\tNODE\tSOURCE IP\tSOURCE PORT\tDESTINATION IP\tDESTINATION PORT\t GATEWAY IP\tGATEWAY PORT"' -w'<host>l:' -qqT'"{$c}\t{$d}\t{$g}\t{$h}\t{$a}\t{$b}\t{$e}\t{$f}"'
```
