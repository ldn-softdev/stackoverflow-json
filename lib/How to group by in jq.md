### Query: [How to group by in jq?](https://stackoverflow.com/questions/60011507/how-to-group-by-in-jq)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/How%20to%20group%20by%20in%20jq.md#a))

Here's the json document

    [
        {"name": "bucket1","clusterName":"cluster1"},
        {"name": "bucket2","clusterName":"cluster1"},
        {"name": "bucket3","clusterName":"cluster2"},
        {"name": "bucket4","clusterName":"cluster2"}
    ]

And I want to convert it to

    [
    {"clusterName": "cluster1", buckets:[{"name": "bucket1"}, {"name": "bucket2"}]},
    {"clusterName": "cluster2", buckets:[{"name": "bucket3"}, {"name": "bucket4"}]},
    ]

How do I do that in jq?

### A:
the first solution using [`jtc`](https://github.com/ldn-softdev/jtc) that imediately springs onto mind, do it in 3 steps:
1. transform each record into the required by the output
2. insert `"name"` records from each duplicae `cluster` into a first unique one
3. walk all unique `cluster`s wrapping those into a JSON array

let's see it in steps:
```bash
# 1. transform each record into the required by the output:
bash $ <document.json jtc -w'<name>l:[-1]' -pi'<name>l:' -T'{"buckets":[{"name":{{}}}]}' -tc
[
   {
      "buckets": [
         { "name": "bucket1" }
      ],
      "clustername": "cluster1"
   },
   {
      "buckets": [
         { "name": "bucket2" }
      ],
      "clustername": "cluster1"
   },
   {
      "buckets": [
         { "name": "bucket3" }
      ],
      "clustername": "cluster2"
   },
   {
      "buckets": [
         { "name": "bucket4" }
      ],
      "clustername": "cluster2"
   }
]
bash $ 

# 2. insert `"name"` records from each duplicae `cluster` into a first unique one:
bash $ <document.json jtc -w'<name>l:[-1]' -pi'<name>l:' -T'{"buckets":[{"name":{{}}}]}' /\
                          -w'[clusterName]:<C>Q:[^0]<C>s[-1][buckets]' -mi'[clusterName]:<>Q:[-1][buckets]' -tc
[
   {
      "buckets": [
         { "name": "bucket1" },
         { "name": "bucket2" }
      ],
      "clusterName": "cluster1"
   },
   {
      "buckets": [
         { "name": "bucket2" }
      ],
      "clusterName": "cluster1"
   },
   {
      "buckets": [
         { "name": "bucket3" },
         { "name": "bucket4" }
      ],
      "clusterName": "cluster2"
   },
   {
      "buckets": [
         { "name": "bucket4" }
      ],
      "clusterName": "cluster2"
   }
]
bash $ 

# final version: 3. walk all unique `cluster`s wrapping those into a JSON array:
bash $ <document.json jtc -w'<name>l:[-1]' -pi'<name>l:' -T'{"buckets":[{"name":{{}}}]}' /\
                          -w'[clusterName]:<C>Q:[^0]<C>s[-1][buckets]' -mi'[clusterName]:<>Q:[-1][buckets]' /\
                          -jw'[clusterName]:<>q:[-1]' -tc
[
   {
      "buckets": [
         { "name": "bucket1" },
         { "name": "bucket2" }
      ],
      "clusterName": "cluster1"
   },
   {
      "buckets": [
         { "name": "bucket3" },
         { "name": "bucket4" }
      ],
      "clusterName": "cluster2"
   }
]
bash $ 
```

However, after a bit of thinking, it's possible to provide a much better/concise solution in 2 steps only:
1. transform each entry into object with the top label of the cluster
2. after merging objects by labels transform each record into a desired format
```bash
# 1. transform each entry into object with the top label of the cluster:
bash $ <document.json jtc -w'[:]' -T'{"{$a}":{"name":{{$b}}}}' -ll -tc
"cluster1": { "name": "bucket1" }
"cluster1": { "name": "bucket2" }
"cluster2": { "name": "bucket3" }
"cluster2": { "name": "bucket4" }
bash $ 

# 2: after merging objects by labels transform each record into a desired format:
bash $ <document.json jtc -w'[:]' -T'{"{$a}":{"name":{{$b}}}}' -ll /\
                          -jw'[:]<L>k' -T'{"clusterName":{{L}}, "buckets":{{}}}' -tc
[
   {
      "buckets": [
         { "name": "bucket1" },
         { "name": "bucket2" }
      ],
      "clusterName": "cluster1"
   },
   {
      "buckets": [
         { "name": "bucket3" },
         { "name": "bucket4" }
      ],
      "clusterName": "cluster2"
   }
]
bash $ 
```
