### Query: [/usr/bin/jq: Argument list too long error bash](https://stackoverflow.com/questions/59854249/usr-bin-jq-argument-list-too-long-error-bash)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/jq%20-%20Argument%20list%20too%20long%20error%20bash.md#a))

I want to replace the value in sample json from larger swagger.json file content and it is too large.

    Error:
    /usr/bin/jq: Argument list too long error bash 

worked to solve this issue for a few days and cannot identify the issue here.
this is the sample json file:



    
    {
       "name": "",
       "description": "",
       "context": "",
       "version": "",
       "provider": "cbs",
       "apiDefinition": "",
       "wsdlUri": null,
       "responseCaching": "Disabled",
       "cacheTimeout": 300,
       "destinationStatsEnabled": false,
       "isDefaultVersion": true,
       "transport":    [
          "http",
          "https"
       ],
       "tags": ["PROVIDER_","MIFE"],
       "tiers": ["Unlimited","Default","Silver","Subscription","Gold","Premium","Bronze"],
       "maxTps":    {
          "sandbox": 5000,
          "production": 1000
       },
       "visibility": "PUBLIC",
       "visibleRoles": [],
       "endpointConfig": "",
       "endpointSecurity":    {
          "username": "user",
          "type": "basic",
          "password": "pass"
       },
       "gatewayEnvironments": "Production and Sandbox",
       "sequences": [],
       "subscriptionAvailability": null,
       "subscriptionAvailableTenants": [],
       "businessInformation":    {
          "businessOwnerEmail": "BUSINESSOWNEREMAIL_",
          "technicalOwnerEmail": "TECHNICALOWNEREMAIL_",
          "technicalOwner": "TECHNICALOWNER_",
          "businessOwner": "BUSINESSOWNER_"
       },
       "corsConfiguration":    {
          "accessControlAllowOrigins": ["*"],
          "accessControlAllowHeaders":       [
             "authorization",
             "Access-Control-Allow-Origin",
             "Content-Type",
             "SOAPAction"
          ],
          "accessControlAllowMethods":       [
             "GET",
             "PUT",
             "POST",
             "DELETE",
             "PATCH",
             "OPTIONS"
          ],
          "accessControlAllowCredentials": false,
          "corsConfigurationEnabled": false
       }
    }


<!-- end snippet -->

 - swagger.json file - [Click here to download swagger.json file][1]

this is the command i using and it give me a error which i as arguments too large. 

    swagger = $(cat swagger.json)

    jq -r --arg swagger "$swagger" '.apiDefinition = $swagger' <<<"$json"

Can anyone please help!


  [1]: https://drive.google.com/file/d/18ol5s7zjtviK73z6-CzRLCROOo_y_ivg/view?usp=sharing

swagger = $(cat swagger.json)


### A:
the request here: insert a rather a very large JSON into a specific location of a sample JSON. With
[`jtc`](https://github.com/ldn-softdev/jtc) there's no issue, as it handles any size JSON of argument (as long memory permits):
```bash
bash $ <sample.json jtc -w'<apiDefinition>l' -u swagger.json 
...
```
Of course, if in-place update is required into the source file use `-f` option:
```bash
bash $ jtc -w'<apiDefinition>l' -u swagger.json -f sample.json 
```




