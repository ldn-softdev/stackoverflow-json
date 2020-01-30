### Query: [create array of object in json and insert the new object each time bash scripts executes](https://stackoverflow.com/questions/59986903/jq-create-array-of-object-in-json-and-insert-the-new-object-each-time-bash-scri)
(jump to the answer)

I want to create valid json using jq in bash.
each time when bash script will execute "Add new element to existing JSON array  "  and if file is empty create new file.

so currently I am using following jq command to create my json (which is incomplete , please help me to complete it )

```
$jq -n -s '{service: $ARGS.named}' \
         --arg transcationId $TRANSACTION_ID_METRIC '{"transcationId":"\($transcationId)"}' \
         --arg name $REALPBPODDEFNAME '{"name ":"\($name )"}'\
		 --arg lintruntime $Cloudlintruntime '{"lintruntime":"\($lintruntime)"}' \
         --arg status $EXITCODE '{"status":"\($status)"}' \
         --arg buildtime $totaltime '{"buildtime":"\($buildtime)"}' >> Test.json
```

which is producing output like 
```

{
  "service": {
    "transcationId": "12345",
    "name": "sdsjkdjsk",
    "lintruntime": "09",   
    "status": "0",
    "buildtime": "9876"
  }
}
{
  "service": {
    "transcationId": "123457",
    "servicename": "sdsjkdjsk",
    "lintruntime": "09",   
    "status": "0",
    "buildtime": "9877"
  }
}

```

but I don't want output in this format 

json should be created first time like 

what should be jq command for  creating below jason

```
{ 
   "ServiceData":{ 
      "date":"30/1/2020",
      "ServiceInfo":[ 
         { 
            "transcationId":"20200129T130718Z",
            "name":"MyService",
            "lintruntime":"178",
            "status":"0",
            "buildtime":"3298"
         }
      ]
   }
}
```


and when I next time execute the bash script element should be added into the array like 

what is the jq command for getting json in this format 
```
{ 
   "ServiceData":{ 
      "date":"30/1/2020",
      "ServiceInfo":[ 
         { 
            "transcationId":"20200129T130718Z",
            "name":"MyService",
            "lintruntime":"16",
            "status":"0",
            "buildtime":"3256"
         },
         { 
            "transcationId":"20200129T130717Z",
            "name":"MyService",
            "lintruntime":"16",
            "status":"0",
            "buildtime":"3256"
         }
      ]
   }
}
```

also I want "date " , "service data" , "service info"  
fields in my json which are missing in my current one ...


please do needful..

Thank You in advance ...

### A:
Given there's a starting template:
```bash
bash $ <Test.json jtc
{
   "ServiceData": {
      "ServiceInfo": [],
      "date": "30/1/2020"
   }
}
bash $ 
```
and all required env. or local variables are setup as proper JSON values:
```bash
bash $ TRANSACTION_ID_METRIC=12345
bash $ REALPBPODDEFNAME='"sdsjkdjsk"'
bash $ Cloudlintruntime='"09"'
bash $ EXITCODE='"0"'
bash $ totaltime='"9876"'
```
then solution is a matter of crafting a rigth JSON with all substitutions:
```bash
bash $ jtc -w'<ServiceInfo>l:' -i'{"transcationId":'$TRANSACTION_ID_METRIC',"name":'$REALPBPODDEFNAME', "lintruntime":'$Cloudlintruntime', "status":'$EXITCODE', "buildtime":'$totaltime' }' -f Test.json
bash $ jtc Test.json
{
   "ServiceData": {
      "ServiceInfo": [
         {
            "buildtime": "9876",
            "lintruntime": "09",
            "name": "sdsjkdjsk",
            "status": "0",
            "transcationId": 12345
         }
      ],
      "date": "30/1/2020"
   }
}
bash $ 
```
with each new insertion (invocation of the cli) the source `Test.json` file will be updated properly





