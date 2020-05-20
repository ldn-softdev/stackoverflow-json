### Query: [jq - duplicate object in list when merging arbitrary number of json arrays from files](https://stackoverflow.com/questions/59884137/jq-duplicate-object-in-list-when-merging-arbitrary-number-of-json-arrays-from)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/jq%20-%20duplicate%20object%20in%20list%20when%20merging%20arbitrary%20number%20of%20json%20arrays%20from%20files.md#a))

I'm configuring cloudwatch agent logs, using saltstack (which is why there some odd syntax).  I am trying to merge an arbitrary number of files, containing identical schema's, but different data into a single file.

**File 1**
```
{
  "logs": {
    "logs_collected": {
      "files":{
        "collect_list": [
          {
            "file_name": "/var/log/suricata/eve-ips.json",
            "log_group_name": "{{grains.environment_full}}SuricataIPS",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S.%f+0000"
          }
        ]
      }
    }
  }
}
```
**File 2**
```
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_name": "/var/log/company/company-json.log",
            "log_group_name": "{{grains.environment_full}}Play",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ"
          },
          {
            "file_name": "/var/log/company/company-notifications.log",
            "log_group_name": "{{grains.environment_full}}Notifications",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ"
          }
        ]
      }
    }
  }
}
```
**File 3**
```
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_name": "/var/ossec/logs/alerts/alerts.json",
            "log_group_name": "{{grains.environment_full}}OSSEC",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%d %H:%M:%S"
          }
        ]
      }
    }
  }
}
```
**jq query** (based on some SO help)
```
jq  -s '.[0].logs.logs_collected.files.collect_list += [.[].logs.logs_collected.files.collect_list | add] | unique| .[0]' web.json suricata.json wazuh-agent.json
```
**Output**
```
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_name": "/var/log/company/company-json.log",
            "log_group_name": "{{grains.environment_full}}Play",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ"
          },
          {
            "file_name": "/var/log/company/company-notifications.log",
            "log_group_name": "{{grains.environment_full}}Notifications",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ"
          },
          {
            "file_name": "/var/log/company/company-notifications.log",
            "log_group_name": "{{grains.environment_full}}Notifications",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ"
          },
          {
            "file_name": "/var/log/suricata/eve-ips.json",
            "log_group_name": "{{grains.environment_full}}SuricataIPS",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S.%f+0000"
          },
          {
            "file_name": "/var/ossec/logs/alerts/alerts.json",
            "log_group_name": "{{grains['environment_full']}}OSSEC",
            "log_stream_name": "{{grains.id}}",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%d %H:%M:%S"
          }
        ]
      }
    }
  }
}
```
if you've gotten this far, thank you.  One additional point to note, is that if i change the order of the files the first index of `collect_list` is always duplicated and if `web.json` is last (the only one with a length of 2) the second log file is not in the group.

### A:
\- the ask to achive with [`jtc`](https://github.com/ldn-softdev/jtc) is so boring, so obvious:
```bash
bash $ <web.json jtc -mi suricata.json -i wazuh-agent.json -tc
{
   "logs": {
      "logs_collected": {
         "files": {
            "collect_list": [
               { "file_name": "/var/log/suricata/eve-ips.json", "log_group_name": "{{grains.environment_full}}SuricataIPS", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%f+0000", "timezone": "UTC" },
               { "file_name": "/var/log/company/company-json.log", "log_group_name": "{{grains.environment_full}}Play", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ", "timezone": "UTC" },
               { "file_name": "/var/log/company/company-notifications.log", "log_group_name": "{{grains.environment_full}}Notifications", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ", "timezone": "UTC" },
               { "file_name": "/var/ossec/logs/alerts/alerts.json", "log_group_name": "{{grains.environment_full}}OSSEC", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%d %H:%M:%S", "timezone": "UTC" }
            ]
         }
      }
   }
}
bash $ 
```
A lot more interesting ask would be this: say, the number of json files in the folder is too big to enlist each file over `-i` option, 
it would be a lot handier solution to include `*.json` files as an argument. The only requirement here would be to know just one
file name, e.g.: `web.json`  
Then the solution is this:
1. read all records from `collect_list` from every JSON file (and build a JSON containing the entire collection)
2. then swap out resulting collection with the known file name
3. and replace in the file its `collect_list` with the built one.
```bash
bash $ jtc -Jw'<collect_list>l[:]' / -w'<J>v' -u web.json / -w'<collect_list>l' -u{{J}} -tc *.json 
{
   "logs": {
      "logs_collected": {
         "files": {
            "collect_list": [
               { "file_name": "/var/log/company/company-json.log", "log_group_name": "{{grains.environment_full}}Play", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ", "timezone": "UTC" },
               { "file_name": "/var/log/company/company-notifications.log", "log_group_name": "{{grains.environment_full}}Notifications", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ", "timezone": "UTC" },
               { "file_name": "/var/ossec/logs/alerts/alerts.json", "log_group_name": "{{grains.environment_full}}OSSEC", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%d %H:%M:%S", "timezone": "UTC" },
               { "file_name": "/var/log/suricata/eve-ips.json", "log_group_name": "{{grains.environment_full}}SuricataIPS", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%f+0000", "timezone": "UTC" }
            ]
         }
      }
   }
}
bash $ 
```
The advantage of such solution that it let handling any number of files as long they share the same schema:

**Explanation for the extended solution:**
- in the first option-set: `-Jw'<collect_list>l[:]'`: read each items from `collect_list` from every json (argument `*.json`) and
put them all in the array, so JSON output looks like this after the 1st set:
```bash
[
   { "file_name": "/var/log/company/company-json.log", "log_group_name": "{{grains.environment_full}}Play", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ", "timezone": "UTC" },
   { "file_name": "/var/log/company/company-notifications.log", "log_group_name": "{{grains.environment_full}}Notifications", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%fZ", "timezone": "UTC" },
   { "file_name": "/var/ossec/logs/alerts/alerts.json", "log_group_name": "{{grains.environment_full}}OSSEC", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%d %H:%M:%S", "timezone": "UTC" },
   { "file_name": "/var/log/suricata/eve-ips.json", "log_group_name": "{{grains.environment_full}}SuricataIPS", "log_stream_name": "{{grains.id}}", "timestamp_format": "%Y-%m-%dT%H:%M:%S.%f+0000", "timezone": "UTC" }
]
```
- in the 2nd options set `-w'<J>v' -u web.json`: first the entire prior JSON output is getting memorized in the _namespase_ `J`
(`-w'<J>v'`) and then replaced with the JSON from file `-u web.json`
- the third option-set `-w'<collect_list>l' -u{{J}}`: updates (replaces) memorized array in the namespace `J` (`-u{{J}}`) 
into the `collect_list` of the current input (which is the JSON from `web.json` as per the 2nd option-set)





