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
                      -w' ' -qqT"$hdr" -w'[:]<L>k' -T'"{L},{$c},{$e},{$d},{$a},{$h},{$b},{$g},{$f}"'
number,firstName,lastName,gender,age,streetAddress,city,state,postalCode
one,John,Smith,man,32,21 2nd Street,New York,NY,10021
two,Johnny,Smithy,man,33,22 2nd Street,New York,NY,10021
bash $ 
```

**Explanation:**
- the firs option set linearizes each of the top objects by _moving_ into the destination points `address` objects:
```bash
bash $ <file.json jtc -w[:] -pi[:][address] -T'{{}}'
{
   "one": {
      "age": 32,
      "city": "New York",
      "firstName": "John",
      "gender": "man",
      "lastName": "Smith",
      "postalCode": "10021",
      "state": "NY",
      "streetAddress": "21 2nd Street"
   },
   "two": {
      "age": 33,
      "city": "New York",
      "firstName": "Johnny",
      "gender": "man",
      "lastName": "Smithy",
      "postalCode": "10021",
      "state": "NY",
      "streetAddress": "22 2nd Street"
   }
}
```
  - `-w[:]` these insertion (destination) points
  - `-pi[:][address] -T'{{}}'`: moves (insert and then purge) each `address` object into _destination points_. Though it's a bit tricky: 
  it requires a template `{{}}` - why? If there wasn't such template (which represents the entire `address` object) then 
  `address` object gets inserted into the iterable where `address` object already exists, so _no actual insertion_ occurs. If such
  template applied - then insertion occurs of the standalone "_lableless_" object (thus, such template anonymizes the object) - and
  in such case the standalone object gets fully inserted.
- the second option set:
  - `-w' ' -T"$hdr" `: prints the header (walk `-w' '` is dummy) via template
  - `-w'[:]<L>k' -T'"{L},{$c},{$e},{$d},{$a},{$h},{$b},{$g},{$f}"'`: walks each object (`[:]`) and memorizes its label (`<L>k`) into the
  _namespace_ `L`, then template-interpolates each record using auto-tokens and the namespace `L` in the required order
  - `-qq` drops the outer quotation marks of the results







