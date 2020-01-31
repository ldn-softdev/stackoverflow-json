### Query: [How to group by in jq?](https://stackoverflow.com/questions/60011507/how-to-group-by-in-jq)
([jump to the answer]())

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
the first solution that imediately springs onto mind, do it in 3 steps:
1. transform each records into required by the output
2. insert `"name"` records from each duplicae `cluster` into a first unique
3. walk all unique `cluster`s wrapping those into a JSON array.

let's see it in steps:
```bash
# 1. transform each records into required by the output:
bash $ <document.json jtc -w'<clusterName>l:[-1]' -u'<clusterName>l:[-1]' -T'{"clustername":{{$a}}, "buckets":[{"name":{{$b}}}]}' -tc
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

# 2. insert `"name"` records from each duplicae `cluster` into a first unique:
bash $ <document.json jtc -w'<clusterName>l:[-1]' -u'<clusterName>l:[-1]' -T'{"clustername":{{$a}}, "buckets":[{"name":{{$b}}}]}' /\
                          -w'[clustername]:<C>Q:[^0]<C>s[-1][buckets]' -i'[clustername]:<C>Q:[-1][buckets][0]' -tc
[
   {
      "buckets": [
         { "name": "bucket1" },
         { "name": "bucket2" }
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
         { "name": "bucket3" },
         { "name": "bucket4" }
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

# final version: 3. walk all unique `cluster`s wrapping those into a JSON array:
bash $ <document.json jtc -w'<clusterName>l:[-1]' -u'<clusterName>l:[-1]' -T'{"clustername":{{$a}}, "buckets":[{"name":{{$b}}}]}' /\
                          -w'[clustername]:<C>Q:[^0]<C>s[-1][buckets]' -i'[clustername]:<C>Q:[-1][buckets][0]' /\
                          -jw'[clustername]:<>q:[-1]' -tc
[
   {
      "buckets": [
         { "name": "bucket1" },
         { "name": "bucket2" }
      ],
      "clustername": "cluster1"
   },
   {
      "buckets": [
         { "name": "bucket3" },
         { "name": "bucket4" }
      ],
      "clustername": "cluster2"
   }
]
bash $ 
```
