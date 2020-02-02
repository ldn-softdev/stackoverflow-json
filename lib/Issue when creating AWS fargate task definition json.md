### Query: [Issue when creating AWS fargate task definition json](https://stackoverflow.com/questions/60025451/issue-when-creating-aws-fargate-task-definition-json)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Issue%20when%20creating%20AWS%20fargate%20task%20definition%20json.md#a))

I'm trying to create a json file (task definiton for AWS fargate) using jq from my gitlab pipeline.

I want to achieve this configuration building a block with "logConfiguration" and "logDriver", see right below:
```
"logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "awslogs-wordpress",
                    "awslogs-region": "us-west-2",
                    "awslogs-stream-prefix": "awslogs-example"
                }
            },
```
1) I have my initial json file below, where I introduce some values with commands on point #2:


```
{
  "family": "",
  "networkMode": "awsvpc",
  "executionRoleArn": "arn:aws:iam::XXXXXXXXX:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "",
      "image": ":",
      "portMappings": [
        {
          "containerPort": 3091,
          "hostPort": 3091,
          "protocol": "tcp"
        }
      ],
      "essential": true
    }
  ],
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "cpu": "256",
  "memory": "512"
}
```

2) When I do these commands with jq on my gitlab pipeline I achieve part of what I want, it seems fine, and I get the json on point #2.A but **I realized it output 3 times "LogDriver"** which is not right:



```
jq '.containerDefinitions[0].logConfiguration.options."awslogs-group"="'my_grup'"' tmp_task > ejm.json &&
jq '.containerDefinitions[0].logConfiguration.options."awslogs-region"="'eu-west-2'"' ejm.json > tmp_task &&
jq '.containerDefinitions[0].logConfiguration.options."awslogs-stream-prefix"="'ecsx'"' tmp_task > ejm.json
```

2.A)
```
{
  "family": "my_branch",
  "networkMode": "awsvpc",
  "executionRoleArn": "arn:aws:iam::235907124541:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "",
      "image": ":",
      "portMappings": [
        {
          "containerPort": 3091,
          "hostPort": 3091,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "options": {
          "awslogs-group": "my_grup",
          "awslogs-region": "eu-west-2",
          "awslogs-stream-prefix": "ecsx"
        },
        "logDriver": "awslogs"
      },
      "logDriver": "awslogs"
    },
    {
      "logConfiguration": {
        "logDriver": "awslogs"
      }
    }
  ],
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "cpu": "256",
  "memory": "512"
}
```

As you can see on point #2.A above, the config "logDriver" is written few times, and when the task definition is created within AWS fargate no logs are available in CloudWatch, because it is not picking the "logDriver" configuration, yes there is a log group in CloudWatch, but neither stdout nor stderr is captured from the container because "logDriver" is not being correctly introduced in the json task.

THE RIGHT JSON TASK DEFINITION SHOULD BE AS THE ONE IN THE LINK BELOW.

https://docs.aws.amazon.com/AmazonECS/latest/userguide/using_awslogs.html

Potential solution is to understand how to correctly write into the json file, or if someone has a better idea of how to put this json task in a pipeline.

Looking forward to get some ideas from you, and thanks a lot in advanced.

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc), it's a matter of inserting a template into a right place:
```bash
bash $ group='"awslogs-group"'
bash $ region='"eu-west-2"'
bash $ prefix='"awslogs-stream-prefix"'
bash $ 
bash $ <input.json jtc -w'[containerDefinitions][0]' -i'{"logConfiguration":{"logDriver":"awslogs","options":{"awslogs-group":'$group',"awslogs-region":'$region',"awslogs-stream-prefix":'$prefix'}}}'
{
   "containerDefinitions": [
      {
         "essential": true,
         "image": ":",
         "logConfiguration": {
            "logDriver": "awslogs",
            "options": {
               "awslogs-group": "awslogs-group",
               "awslogs-region": "eu-west-2",
               "awslogs-stream-prefix": "awslogs-stream-prefix"
            }
         },
         "name": "",
         "portMappings": [
            {
               "containerPort": 3091,
               "hostPort": 3091,
               "protocol": "tcp"
            }
         ]
      }
   ],
   "cpu": "256",
   "executionRoleArn": "arn:aws:iam::XXXXXXXXX:role/ecsTaskExecutionRole",
   "family": "",
   "memory": "512",
   "networkMode": "awsvpc",
   "requiresCompatibilities": [
      "FARGATE"
   ]
}
bash $ 
```




