### Query: [How to find and replace a value with jq in a nested json](https://stackoverflow.com/questions/59851383/how-to-find-and-replace-a-value-with-jq-in-a-nested-json)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/How%20to%20find%20and%20replace%20a%20value%20with%20jq%20in%20a%20nested%20json.md#a))

How to find and replace a value with jq in a nested json.

https://raw.githubusercontent.com/linuxmint/cinnamon/master/files/usr/share/cinnamon/applets/menu%40cinnamon.org/settings-schema.json


    {
        "layout": {
            "menu-layout": {
                "type": "section",
                "title": "Layout and content",
            "keys": [
                "show-category-icons",
                "favbox-show",
                "favbox-min-height",
                "show-places",
            ]
            },
            "menu-behave": {
                "type": "section",
                "keys": [
                    "enable-autoscroll",
                    "search-filesystem"
                ]
            }
        },
        "favbox-min-height": {
            "type": "spinbutton",
            "default": 300,
        }
    }


For example, in this file above teste.json:
I want to replace in the item "favbox-min-height", in the key: value
"default": 300
for
"default": 400

I'm not able to do this, could someone help me do this?

### A:
the input invariant solution using [`jtc`](https://github.com/ldn-softdev/jtc) is easy:
```bash
bash $ <teste.json jtc -w'<favbox-min-height>l[default]' -u400
{
   "favbox-min-height": {
      "default": 400,
      "type": "spinbutton"
   },
   "layout": {
      "menu-behave": {
         "keys": [
            "enable-autoscroll",
            "search-filesystem"
         ],
         "type": "section"
      },
      "menu-layout": {
         "keys": [
            "show-category-icons",
            "favbox-show",
            "favbox-min-height",
            "show-places"
         ],
         "title": "Layout and content",
         "type": "section"
      }
   }
}
bash $ 
```




