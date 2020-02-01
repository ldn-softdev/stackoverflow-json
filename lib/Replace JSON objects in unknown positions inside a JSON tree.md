### Query: [Replace JSON objects in unknown positions inside a JSON tree](https://stackoverflow.com/questions/59913277/replace-json-objects-in-unknown-positions-inside-a-json-tree)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Replace%20JSON%20objects%20in%20unknown%20positions%20inside%20a%20JSON%20tree.md#a))

In a JSON file I need to replace encrypted values with their clear text values as an initialization process using the command line tool `jq`. An application will then re-encrypt the values with its own keys, overwriting the clear text values. Encrypted values are represented as "$crypto" objects, containing information about the encryption method and which keys were used, looking like this:

    {
        "$crypto" : {
            "type" : "x-simple-encryption",
            "value" : {
                "cipher" : "AES/CBC/PKCS5Padding",
                "stableId" : "someId",
                "salt" : "4J5ckE6+JaS8TLqAN4073g==",
                "data" : "vBeHAPJXLl+X/8Enp9vxMA==",
                "keySize" : 16,
                "purpose" : "someDescription",
                "iv" : "N2xCe5RiJibHv9hLY+OduA==",
                "mac" : "VoOo1BKptwfqIJeSOb/qGA=="
            }
        }
    }

These "$crypto" objects can be anywhere in the JSON structure. A sample input document looks like this:

    {
        "unknownKey1" : {
            "unknownKey2" : {
                "name" : "JWT_SESSION",
                "properties" : {
                    "maxTokenLifeMinutes" : 120,
                    "tokenIdleTimeMinutes" : 30
                }
            },
            "unknownKey3" : [
                {
                    "unknownKey4" : "STATIC_USER",
                    "unknownKey5" : {
                        "unknownKey6" : "internal/user",
                        "unknownKey7" : "anonymous",
                        "unknownKey8" : {
                            "$crypto" : {
                                "type" : "x-simple-encryption",
                                "value" : {
                                    "cipher" : "AES/CBC/PKCS5Padding",
                                    "stableId" : "someId",
                                    "salt" : "4J5ckE6+JaS8TLqAN4073g==",
                                    "data" : "vBeHAPJXLl+X/8Enp9vxMA==",
                                    "keySize" : 16,
                                    "purpose" : "someDescription",
                                    "iv" : "N2xCe5RiJibHv9hLY+OduA==",
                                    "mac" : "VoOo1BKptwfqIJeSOb/qGA=="
                                }
                            }
                        }
                    },
                    "enabled" : true
                }
            ]
        }
    }

So the value of "unknownKey8" was encrypted. I need that document to look like this:

    {
        "unknownKey1" : {
            "unknownKey2" : {
                "name" : "JWT_SESSION",
                "properties" : {
                    "maxTokenLifeMinutes" : 120,
                    "tokenIdleTimeMinutes" : 30
                }
            },
            "unknownKey3" : [
                {
                    "unknownKey4" : "STATIC_USER",
                    "unknownKey5" : {
                        "unknownKey6" : "internal/user",
                        "unknownKey7" : "anonymous",
                        "unknownKey8" : "clearTextValue"
                    },
                    "enabled" : true
                }
            ]
        }
    }

I have been able to find crypto objects in the input file using the following command:

    cat input.json | jq 'paths | select(.[-1] == "$crypto")'
    [
      "unknownKey1",
      "unknownKey3",
      0,
      "unknownKey5",
      "unknownKey8",
      "$crypto"
    ]

But I have not been able to make meaningful progress on performing the replacement.

### A:
the query using [`jtc`](https://github.com/ldn-softdev/jtc) is super easy:
```bash
bash $ ev='"clearTextValue"'
bash $ 
bash $ <input.json jtc -w'<$crypto>l:[-1]' -u"$ev"
{
   "unknownKey1": {
      "unknownKey2": {
         "name": "JWT_SESSION",
         "properties": {
            "maxTokenLifeMinutes": 120,
            "tokenIdleTimeMinutes": 30
         }
      },
      "unknownKey3": [
         {
            "enabled": true,
            "unknownKey4": "STATIC_USER",
            "unknownKey5": {
               "unknownKey6": "internal/user",
               "unknownKey7": "anonymous",
               "unknownKey8": "clearTextValue"
            }
         }
      ]
   }
}
bash $ 
```

