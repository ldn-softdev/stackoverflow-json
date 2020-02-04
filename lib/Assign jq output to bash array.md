### Query: [Assign jq output to bash array](https://stackoverflow.com/questions/60052195/assign-jq-output-to-bash-array)
([jump to the answer]())

I have a problem to assign value to bash array correctly which was parsed by `jq`. I have a JSON output from `curl`: 

    {
      "id": 6442,
      "name": "Execute Workflow",
      "description": "Plan: data",
      "status": "In Queue",
      "start_date": 0,
      "end_date": 0,
      "job_type": "Execute Workflow",
      "created_by_name": null,
      "creation_date": 1580762385615,
      "creation_date_str": "02/03/2020 09:39:45 PM",
      "last_updated_date": 1580762385615,
      "last_updated_date_str": "02/03/2020 09:39:45 PM",
      "last_updated_by_name": null,
      "schedule_on": 0,
      "paused_at_step": 0,
      "percent_complete": 0,
      "job_action_type": null,
      "child_job_id": -1
    }

I want to save two key values `.id` and `.status` into bash array.

I am doing it this way:

    array=( $(echo '{ "id": 6442, "name": "Execute Workflow", "description": "Plan: data", "status": "In Queue", "start_date": 0, "end_date": 0, "job_type": "Execute Workflow", "created_by_name": null, "creation_date": 1580762385615, "creation_date_str": "02/03/2020 09:39:45 PM", "last_updated_date": 1580762385615, "last_updated_date_str": "02/03/2020 09:39:45 PM", "last_updated_by_name": null, "schedule_on": 0, "paused_at_step": 0, "percent_complete": 0, "job_action_type": null, "child_job_id": -1}' | jq '.id, .status') ) 

All seems OK until I try to get second element of that array: `echo ${array[1]}` and I get
`"In` not `"In Queue"`. 

My array is 3 elements long `echo ${#array[@]}` returns `3` but I want it to be 2 elements long.
Can someone help me please?

My next steps in bash script is to assign `job_status="=${array[1]}"` and I want to get variable `job_status="In Queue"`.

### A:
basically, the question is not about JSON convertsion, but rather about bash array parsing. To ensure that JSON strings containing spaces
will be setup in the array properly, the bash delimeter `IFS` needs to be set to the unique value:
```bash
bash $ ifs="$IFS"             # preserve IFS
bash $ IFS='|'                # | will be used as a unique separator

bash $ array=($(<curl.json jtc -qqT'"{$g}|\"{$r}\""'))
bash $ echo ${#array[@]}
2
bash $ echo ${array[1]}
"In Queue"
bash $ echo ${array[0]}
6442
bash $ 
```




