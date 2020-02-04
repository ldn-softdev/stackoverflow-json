### Query: [Trying to find date based on jq](https://stackoverflow.com/questions/60045283/trying-to-find-date-based-on-jq)
([jump to the answer]())

In Home Assistant CLI, running `hassio snapshots list`, the output is as below, where I'm trying to locate the last date to keep in this output looking back 3 days (in the example list below this should be the date of 2020-01-24):

    - date: "2019-12-10T03:00:01.313293+00:00"
      name: Automated backup 2019-12-10 04:00
      protected: false
      slug: a0d3f958
      type: full
    - date: "2020-02-03T16:25:55.265219+00:00"
      name: Automated backup 2020-02-03 17:25
      protected: false
      slug: acb7907b
      type: full
    - date: "2020-02-03T15:00:11.584836+00:00"
      name: Automated backup 2020-02-03 16:00
      protected: false
      slug: 6284d707
      type: full
    - date: "2020-01-24T03:00:01.169351+00:00"
      name: Automated backup 2020-01-24 04:00
      protected: false
      slug: 53d10566
      type: full



Earlier this worked, but there has been a change and I can't resolve what is wrong now:

    last_date_to_keep=$(hassio snapshots list | jq .data.snapshots[].date | sort -r | head -n "3" | tail -n 1 | xargs date -D "%Y-%m-%dT%T" +%s --date )

The output is:

    zsh: no matches found: .data.snapshots[].date
    date: option requires an argument: date

### A:
Ideologically it would be wrong to process _yaml_ data with non-yaml tools. However, just as a point of an exercise, let's do it with `sed`
and [`jtc`](https://github.com/ldn-softdev/jtc).

Given `jtc` works only with JSONs, we need to use `sed` to convert the above _yaml_ into a _stream of JSONs_:
```bash
bash $ <file.txt sed -E 's/(^[^:]+): (.*)/{"\1": "\2"}/; s/""/"/g'
{"- date": "2019-12-10T03:00:01.313293+00:00"}
{"  name": "Automated backup 2019-12-10 04:00"}
{"  protected": "false"}
{"  slug": "a0d3f958"}
{"  type": "full"}
{"- date": "2020-02-03T16:25:55.265219+00:00"}
{"  name": "Automated backup 2020-02-03 17:25"}
{"  protected": "false"}
{"  slug": "acb7907b"}
{"  type": "full"}
{"- date": "2020-02-03T15:00:11.584836+00:00"}
{"  name": "Automated backup 2020-02-03 16:00"}
{"  protected": "false"}
{"  slug": "6284d707"}
{"  type": "full"}
{"- date": "2020-01-24T03:00:01.169351+00:00"}
{"  name": "Automated backup 2020-01-24 04:00"}
{"  protected": "false"}
{"  slug": "53d10566"}
{"  type": "full"}
bash $ 
```

Now, we can use `jtc` to sort the date and select the 3rd date from the 3 end of sorted:
```bash
bash $ <file.txt sed -E 's/(^[^:]+): (.*)/{"\1": "\2"}/; s/""/"/g' |\
                 jtc -J / -jw'[- date]:<>g:' / -w[-3:-2]
"2020-01-24T03:00:01.169351+00:00"
bash $ 
```






