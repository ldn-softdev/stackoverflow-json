### Query: [Retrieve value of odata-id with jq in Redfish output](https://stackoverflow.com/questions/59959364/retrieve-value-of-odata-id-with-jq-in-redfish-output)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Retrieve%20value%20of%20odata-id%20with%20jq%20in%20Redfish%20output.md#a))

I'm parsing some json Redfish data using jq. 

Trying to pull value for @odata.id from redfish.txt file below.

Invoking with recommended jq .["@odata.id"] doesn't seem to quite work to pull just value itself which is: /redfish/v1/Systems

Any suggestions welcomed. Output below... :)

Thanks,
Nick

    root@ubuntu-xenial:/var/opt# cat redfish.txt
    {"@odata.context":"/redfish/v1/$metadata#ServiceRoot.ServiceRoot","@odata.id":"/redfish/v1","@odata.type":"#ServiceRoot.v1_2_0.ServiceRoot","AccountService":{"@odata.id":"/redfish/v1/Managers/iDRAC.Embedded.1/AccountService"},"Chassis":{"@odata.id":"/redfish/v1/Chassis"},"Description":"Root Service","EventService":{"@odata.id":"/redfish/v1/EventService"},"Id":"RootService","JsonSchemas":{"@odata.id":"/redfish/v1/JSONSchemas"},"Links":{"Sessions":{"@odata.id":"/redfish/v1/Sessions"}},"Managers":{"@odata.id":"/redfish/v1/Managers"},"Name":"Root Service","Oem":{"Dell":{"@odata.type":"#DellServiceRoot.v1_0_0.ServiceRootSummary","IsBranded":0,"ManagerMACAddress":"d0:96:69:51:d4:70","ServiceTag":"XXXX"}},"RedfishVersion":"1.2.0","Registries":{"@odata.id":"/redfish/v1/Registries"},"SessionService":{"@odata.id":"/redfish/v1/SessionService"},"Systems":{"@odata.id":"/redfish/v1/Systems"},"Tasks":{"@odata.id":"/redfish/v1/TaskService"},"UpdateService":{"@odata.id":"/redfish/v1/UpdateService"}}
    root@ubuntu-xenial:/var/opt# cat redfish.txt | jq .Systems
    {
      "@odata.id": "/redfish/v1/Systems"
    }
    root@ubuntu-xenial:/var/opt# cat redfish.txt | jq .Systems | jq .@odata.id
    jq: error: syntax error, unexpected FIELD, expecting QQSTRING_START (Unix shell quoting issues?) at <top-level>, line 1:
    .@odata.id
    jq: error: try .["field"] instead of .field for unusually named fields at <top-level>, line 1:
    .@odata.id
    jq: 2 compile errors
    root@ubuntu-xenial:/var/opt# cat redfish.txt | jq .Systems | jq .["@odata.id"]
    {
      "@odata.id": "/redfish/v1/Systems"
    }
    "/redfish/v1/Systems"
    root@ubuntu-xenial:/var/opt# cat redfish.txt | jq .Systems | jq .["odata.id"]
    {
      "@odata.id": "/redfish/v1/Systems"
    }
    "/redfish/v1/Systems"
    root@ubuntu-xenial:/var/opt#

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc) finding/selecting the required JSON entry is super easy. 
In this case `"@odata.id"` label is required, it could be found via recursive search `<@odata.id>l`, but since there are many, first
we need to find a branch holding the required label:
```bash
bash $ <redfish.txt jtc -w'<Systems>l<@odata.id>l'
"/redfish/v1/Systems"
bash $ 
```
