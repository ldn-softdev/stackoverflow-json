### Query: [Converting a nested JSON into csv with jq](https://stackoverflow.com/questions/59820910/converting-a-nested-json-into-csv-with-jq)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Converting%20a%20nested%20JSON%20into%20csv%20with%20jq.md#a))

I would like to convert the following JSON into a csv format using jq. I know there are tons of similar questions but I could not figure it out based on them.
```json
{
	"one": {
		"firstName": "John",
		"lastName": "Smith",
		"gender": "man",
		"age": 32,
		"address": {
			"streetAddress": "21 2nd Street",
			"city": "New York",
			"state": "NY",
			"postalCode": "10021"
		}
	},
	"two": {
		"firstName": "Johnny",
		"lastName": "Smithy",
		"gender": "man",
		"age": 33,
		"address": {
			"streetAddress": "22 2nd Street",
			"city": "New York",
			"state": "NY",
			"postalCode": "10021"
		}
	}
}
```
The output should look like the following. I'm struggeling with the nested object as value of the address key. 

```none
number,firstName,lastName,gender,age,streetAddress,city,state,postalCode
one,John, Smith,man, 32, 21 2nd Street, New York, NY, 10021
two, Johnny, Smith,man, 33, 22 2nd Street, New York, NY, 10021
```


The Best I could do is the foillowing but it does not come close...
Your help is much apreciated

```sh
jq --raw-output 'to_entries | map_values({ job: .key } + .value )'
```

### A:
in this qeury, the best approach using [`jtc`](https://github.com/ldn-softdev/jtc)
would be to linearize each record and then template-interpolate it:
```bash
bash $ hdr='"number,firstName,lastName,gender,age,streetAddress,city,state,postalCode"'
bash $ 
bash $ <file.json jtc -w[:] -pi[:][address] -T'{{}}' /\
                      -nw' ' -qqT"$hdr" -w'[:]<L>k' -T'"{L},{$c},{$e},{$d},{$a},{$h},{$b},{$g},{$f}"'
number,firstName,lastName,gender,age,streetAddress,city,state,postalCode
one,John,Smith,man,32,21 2nd Street,New York,NY,10021
two,Johnny,Smithy,man,33,22 2nd Street,New York,NY,10021
bash $ 
```









