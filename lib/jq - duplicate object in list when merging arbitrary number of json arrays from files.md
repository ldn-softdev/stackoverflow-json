### Query: [jq - duplicate object in list when merging arbitrary number of json arrays from files](https://stackoverflow.com/questions/59884137/jq-duplicate-object-in-list-when-merging-arbitrary-number-of-json-arrays-from)
([jump to the answer]())

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
\- the ask to achive with [`jtc`](https://github.com/ldn-softdev/jtc) is so obvious, so boring:
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

