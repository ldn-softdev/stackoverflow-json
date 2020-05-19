### Query: [How to parse json with jq where the structure is non-linear](https://stackoverflow.com/questions/59823154/how-to-parse-json-with-jq-where-the-structure-is-non-linear)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/How%20to%20parse%20json%20with%20jq%20where%20the%20structure%20is%20non-linear.md#a))

I hope I have represented my problem clearly.
Need help querying and them parsing multiple json files using JQ where the structure is non-linear within each file. The application produces config data that can look like this example. There can be zero or many of the DualEndPoint or Local objects per file.  I need to be able to query for a specific user in the "User" attribute and insert a new password for resubmission back to the api.  For DualEndPoints, the nested object names are variable so one cannot code those values in looking for the "User" attribute.

Where a match for a specific user is found, return the entire structure with only that user's new password inserted. In the example, querying for user1 would return the entire PROFILE1 and PROFILE2, but not PROFILE3 as it doesn't contain user1 credentials.

    {
      "PROFILE1": {
        "Type": "ConnectionProfile:FileTransfer:DualEndPoint",
        "WorkloadAutomationUsers": [
          "*"
        ],
        "VerifyBytes": true,
        "TargetAgent": "sqlrptvmjhbpr01",
        "TargetCTM": "Production",
        "Endpoint:Src:Local_0": {
          "Type": "Endpoint:Src:Local",
          "User": "user1",
          "Port": "0",
          "OsType": "Windows",
          "HostName": "Local",
          "Password": "*****",
          "HomeDirectory": "/user1homedir"
        },
        "Endpoint:Dest:SFTP_1": {
          "Type": "Endpoint:Dest:SFTP",
          "User": "user2",
          "HostName": "server2",
          "Password": "*****",
          "HomeDirectory": "/user2homedir"
        }
      },
      "PROFILE2": {
        "Type": "ConnectionProfile:FileTransfer:Local",
        "WorkloadAutomationUsers": [
          "*"
        ],
        "VerifyBytes": true,
        "User": "user1",
        "VerifyDestination": true,
        "OsType": "Windows",
        "HostName": "Local",
        "Password": "*****",
        "TargetAgent": "server1",
        "TargetCTM": "Production"
      },
      "PROFILE3": {
        "Type": "ConnectionProfile:FileTransfer:Local",
        "WorkloadAutomationUsers": [
          "*"
        ],
        "VerifyBytes": true,
        "User": "user3",
        "OsType": "Windows",
        "HostName": "Local",
        "Password": "*****",
        "HomeDirectory": "/user3hoemdir",
        "TargetAgent": "server2",
        "TargetCTM": "Production"
      }
    }

### A:
Assuming, I understand the query right - with [`jtc`](https://github.com/ldn-softdev/jtc) it takes 2 steps to accomplish the query:
1. purge all profiles which do not hold required user
2. for those which do (those survived step 1) update the password.

for clarity of the answer, the username and password will be handled though shell variables:
```bash
bash $ user='user1'
bash $ passwd='new_passwd'
bash $ 
bash $ <file.json jtc -pw'<PROFILE\d+>L:<>f[User]:<'$user'><>F' /\
                      -w'[User]:<'$user'>:[-1][Password]' -u'"'$passwd'"'
{
   "PROFILE1": {
      "Endpoint:Dest:SFTP_1": {
         "HomeDirectory": "/user2homedir",
         "HostName": "server2",
         "Password": "*****",
         "Type": "Endpoint:Dest:SFTP",
         "User": "user2"
      },
      "Endpoint:Src:Local_0": {
         "HomeDirectory": "/user1homedir",
         "HostName": "Local",
         "OsType": "Windows",
         "Password": "new_passwd",
         "Port": "0",
         "Type": "Endpoint:Src:Local",
         "User": "user1"
      },
      "TargetAgent": "sqlrptvmjhbpr01",
      "TargetCTM": "Production",
      "Type": "ConnectionProfile:FileTransfer:DualEndPoint",
      "VerifyBytes": true,
      "WorkloadAutomationUsers": [
         "*"
      ]
   },
   "PROFILE2": {
      "HostName": "Local",
      "OsType": "Windows",
      "Password": "new_passwd",
      "TargetAgent": "server1",
      "TargetCTM": "Production",
      "Type": "ConnectionProfile:FileTransfer:Local",
      "User": "user1",
      "VerifyBytes": true,
      "VerifyDestination": true,
      "WorkloadAutomationUsers": [
         "*"
      ]
   }
}
bash $ 
```

**Explanation:**
1. first option-set:
  - walk path `-w'<PROFILE\d+>L:<>f[User]:<'$user'><>F'`: finds by the label each entry matching RE `PROFILE\d+`, then sets a branching
  point (`<>f`):
    - if the found entry contains the record `"User":"user1" `(`[User]:<'$user'>` matched), then such entry is skipped (`<>F`) 
    and the walk continues to the next iteration
    - if the found entry does not contain the user record - then the entire entry is matched (`<>f` triggered)
  - so, all matched entries (i.e., those where no user record found) are getting purged (`-p`) effectively leaving only those where
  user record is found
2. second option-set:
  - `-w '[User]:<'$user'>:[-1][Password]'`: finds each record `"User":"user1"` (lexeme `[User]:<'$user'>:`) then selects its
  immediate parent (`[-1]`) and then select entry by the label `[Password]` and 
  - updates it with the password `-u'"'$passwd'"'`










