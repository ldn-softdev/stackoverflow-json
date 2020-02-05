### Query: [Use jq to replace many values with variable values](https://stackoverflow.com/questions/60061905/use-jq-to-replace-many-values-with-variable-values)
([jump to the answer]())

Using `jq`, is it possible to replace the value of each parameter in the sample JSON with the value of the variable that is the initial value?

In my scenario, Azure DevOps does not carryout any kind of variable substitution on the JSON file, so I need to do it manually. So for example, say `$SUBSCRIPTION_ID` is set to `abc-123`, I'd like to use `jq` to update the JSON file.

I can pull out the values using `.parameters[].value`, but I can't seem to find a way of setting each individual value.

The main challenge here is that the solution should be reusable, and different JSON files will have different `parameters`, so I don't think I can use `--argjson`.

# Example

### Original JSON

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/parametersTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "subscriptionId": {
                "value": "$SUBSCRIPTION_ID"
            },
            "topicName": {
                "value": "$TOPIC_NAME"
            }
        }
    }

### Variables

    SUBSCRIPTION_ID="abc-123"
    TOPIC_NAME="SomeTopic"

### Desired JSON

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/parametersTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "subscriptionId": {
                "value": "abc-123"
            },
            "topicName": {
                "value": "SomeTopic"
            }
        }
    }

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc): iterate recursively over each `value` holding a substitution token (starting with `$`)
and then replace it through _shell evaluation_:
```bash
bash $ <schema.json jtc -w'[value]:<^\$(.*)>R:' -eu echo '"${$1}"' \;
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/parametersTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "subscriptionId": {
         "value": "abc-123"
      },
      "topicName": {
         "value": "SomeTopic"
      }
   }
}
bash $ 
```



