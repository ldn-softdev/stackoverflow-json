### Query: [jq - iterate through dictionaries](https://stackoverflow.com/questions/60060802/jq-iterate-through-dictionaries)
([jump to the answer]())

My json knowledge is shaky, so pardon me if I use the wrong terminology.

I have `input.txt` which can be simplified down to this:

    [
      {
        "foo1": "bar1",
        "baz1": "fizz1"
      },
      {
        "foo2": "bar2",
        "baz2": "fizz2"
      }
    ]

I want to iterate through each object via a loop, so I'm essentially hoping to tackle just the 1's first, then loop through the 2's, etc. 

I thought it was something like:

    jq 'keys[]' input.json | while read key ; do
        echo "loop --$(jq "[$key]" input.json)"
    done

but that's giving me

    loop 0
    loop 1

where I would expect to see (spacing here is optional, not sure how jq would parse it):

    loop { "foo1": "bar1", "baz1": "fizz1" }
    loop { "foo2": "bar2", "baz2": "fizz2" }

What am I missing?

### A:
solution with [`jtc`](https://github.com/ldn-softdev/jtc):
```bash
bash $ <input.json jtc -w[:] -qqT'"loop >{{}}<"' 
loop { "baz1": "fizz1", "foo1": "bar1" }
loop { "baz2": "fizz2", "foo2": "bar2" }
bash $ 
```
\- the trick was to use JSON stringification `>..<` template-interpolation operation in order to fit walked JSON into a string.



