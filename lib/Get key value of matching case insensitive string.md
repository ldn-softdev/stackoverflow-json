### Query: [Get key value of matching case insensitive string](https://stackoverflow.com/questions/59969841/jq-get-key-value-of-matching-case-insensitive-string)
([jump to the answer](https://github.com/ldn-softdev/stackoverflow-json/blob/master/lib/Get%20key%20value%20of%20matching%20case%20insensitive%20string.md#a))

I'm attempting to pull some specific data from the "id" field but `jq` matches are being case sensitive and creating an issue with the searches (not matching, basically, so returning 0 results).

JSON Example:

```
{
  "created_at": "2020-01-17T12:54:02Z",
  "primary": true,
  "verified": true,
  "deliverable_state": "deliverable",
  "undeliverable_count": 0,
  "updated_at": "2020-01-17T12:54:03Z",
  "url": "http://www.website.com",
  "id": 376062709553,
  "user_id": 374002305374,
  "type": "email",
  "value": "test-1234567@domain.com"
}
{
  "created_at": "2019-02-07T20:49:41Z",
  "primary": false,
  "verified": true,
  "deliverable_state": "deliverable",
  "undeliverable_count": 0,
  "updated_at": "2019-02-07T20:49:41Z",
  "url": "http://www.website.com",
  "id": 366235941554,
  "user_id": 374002305374,
  "type": "email",
  "value": "MyEmail@domain.com"
}
```

When running `jq` against the following, I get the correct return:

```
$ jq -r '. | select(.value=="MyEmail@domain.com") .id' sample.json
366235941554
```

But if I run with the incorrect case, eg.

```
$ jq -r '. | select(.value=="Myemail@domain.com") .id' sample.json
```

...I do not get a result.  I've read through some of the documentation but unfortunately, I do not understand the test/match/capture flags for insensitive search (https://stedolan.github.io/jq/manual/#RegularexpressionsPCRE) or how to get it to operate with this "locate and give me the value" request.

I've searched seemingly everywhere but I'm unable to find any examples of this being used.

### A:

[`jtc`](https://github.com/ldn-softdev/jtc) based solution: since a stream of JSON is given to process all of them `-a` option is
required:
```bash
bash $ jtc -aw'[value]:<myemail@domain.com\I>R[-1][id]' sample.json
366235941554
bash $ 
```
> Note the trailing `\I` in the REGEX match lexeme - that's the way to specify
[REGEX options](https://github.com/ldn-softdev/jtc/blob/master/User%20Guide.md#searching-json-with-re).

