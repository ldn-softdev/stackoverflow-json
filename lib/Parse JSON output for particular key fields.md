### Query: [Parse JSON output for particular key fields](https://stackoverflow.com/questions/59791160/parse-json-output-for-particular-key-fields)
([jump to the answer]())

I have the following JSON content in a file.json. 

I need only a particular key field from all this overwhelming information. 

Let us assume that I need `web_url`,

The problem here is there are multiple key field with "web_url".

How do only get the `web_url` field I am after?

    [{"id":196,"iid":1,"project_id":233,"title":"DEV to Master","description":"","state":"merged","created_at":"2019-12-04T14:14:35.424-06:00","updated_at":"2019-12-04T14:14:47.310-06:00","merged_by":{"id":122,"name":"Sengoku","username":"sengk","state":"active","avatar_url":"https://secure.gravatar.com/avatar/7cvffgfgfgfgf9eb1348d0ba7795a076?s=80\u0026d=identicon","web_url":"https://gitlaboo.tests.com/sengk"},"merged_at":"2019-12-04T14:14:47.468-06:00","closed_by":null,"closed_at":null,"target_branch":"master","source_branch":"DEV","upvotes":0,"downvotes":0,"author":{"id":122,"name":"Sengoku","username":"sengk","state":"active","avatar_url":"https://secure.gravatar.com/avatar/7fgdfdgdfgdvfg9eb1348d0ba7795a076?s=80\u0026d=identicon","web_url":"https://gitlaboo.tests.com/sengk"},"assignee":{"id":122,"name":"Sengoku","username":"sengk","state":"active","avatar_url":"https://secure.gravatar.com/avatar/7afsdfdvdfvfde24f89eb1348d0ba7795a076?s=80\u0026d=identicon","web_url":"https://gitlaboo.tests.com/sengk"},"source_project_id":233,"target_project_id":233,"labels":[],"work_in_progress":false,"milestone":null,"merge_when_pipeline_succeeds":false,"merge_status":"can_be_merged","sha":"6318e51ea8czfdfsdvdfvdfbc02988ba62c71e5774107e","merge_commit_sha":"6dc5vdfvdfgdfg5bf14e97dea949b8584c0c68d6","user_notes_count":0,"discussion_locked":null,"should_remove_source_branch":null,"force_remove_source_branch":false,"web_url":"https://gitlaboo.tests.com/demo/frog/merge_requests/1","time_stats":{"time_estimate":0,"total_time_spent":0,"human_time_estimate":null,"human_total_time_spent":null},"squash":false}]
    
### A:
Unsure what's the ask here. It looks though like a banal addressing question:
```bash
# to address the first match:
bash $ <file.json jtc -w'<web_url>l'           # or equally <web_url>l0
"https://gitlaboo.tests.com/sengk"
bash $ 

# to address the second match:"
bash $ <file.json jtc -w'<web_url>l1' 
"https://gitlaboo.tests.com/sengk"
bash $ 

# and so on. To address all the matches:
bash $ <file.json jtc -w'<web_url>l:' 
"https://gitlaboo.tests.com/sengk"
"https://gitlaboo.tests.com/sengk"
"https://gitlaboo.tests.com/sengk"
"https://gitlaboo.tests.com/demo/frog/merge_requests/1"
bash $ 

# to address all the matches starting from 3rd one:
bash $ <file.json jtc -w'<web_url>l2:' 
"https://gitlaboo.tests.com/sengk"
"https://gitlaboo.tests.com/demo/frog/merge_requests/1"
bash $ 

# etc
```
