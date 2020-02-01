### Query: [JQ Transverse List as Map](https://stackoverflow.com/questions/59915712/jq-transverse-list-as-map)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/JQ%20Transverse%20List%20as%20Map.md#a))

I want to convert the Below JSON to Map. but its coming as a list.


    {
      "_embedded": {
        "items": [
          {
             "order": 1,
            "technical": {
              "id": 1,
              "_links": {
                "self": {
                  "href": "/profile/api/v3/service/40/technical"
                }
              }
            }
          },
           {
            "order": 2,
            "technical": {
              "id": 1,
              "_links": {
                "self": {
                  "href": "/profile/api/v3/service/40/technical"
                }
              }
            }
          }
        ]
      }
    }

Expected format:

    {
      "1": "/profile/api/v3/service/40/technical",
      "2": "/profile/api/v3/service/40/technical"
    }

But i am getting :::

    [
      {
        "1": "/profile/api/v3/service/40/technical"
      },
      {
        "2": "/profile/api/v3/service/40/technical"
      }
    ]


JQ Query:::

    ._embedded.items | map({(.order| tostring ) : .technical._links.self.href} )

Please help, Thanks in Advance.. 

Code Snippet - https://jqplay.org/s/cEzh5_LimP 

### A:
with [`jtc`](https://github.com/ldn-softdev/jtc): walk each `order` and memorize its value then find respective `href` and 
transform into the required format:
```bash
bash $ <sample.json jtc -w'<order>l:<O>v[-1]<href>l' -T'{"{O}":{{}}}' -lljj
{
   "1": "/profile/api/v3/service/40/technical",
   "2": "/profile/api/v3/service/40/technical"
}
bash $ 
```






