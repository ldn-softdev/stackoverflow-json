### Q: [Concatenating one attribute of JSON from multiple into one file](https://stackoverflow.com/questions/59975714/concatenating-one-attribute-of-json-from-multiple-into-one-file-using-jq)

I have multiple JSON files that are of similar form, here are two examples:

message_1.json
```
{
  "participants": [
    {
      "name": "Person One"
    },
    {
      "name": "Person Two"
    }
  ],

  "messages": [
    {
      "sender_name": "Person One",
      "timestamp_ms": 0002,
      "content": "Text2.",
      "type": "Generic"
    },
    {
      "sender_name": "Person Two",
      "timestamp_ms": 0001,
      "content": "Text1.",
      "type": "Generic"
    }
  ],
  "title": "Person One",
  "is_still_participant": true,
  "thread_type": "Regular",
  "thread_path": "inbox/SomeString"
}
```
message_2.json
```
{
  "participants": [
    {
      "name": "Person One"
    },
    {
      "name": "Person Two"
    }
  ],

  "messages": [
    {
      "sender_name": "Person Two",
      "timestamp_ms": 0004,
      "content": "Text4.",
      "type": "Generic"
    },
    {
      "sender_name": "Person One",
      "timestamp_ms": 0003,
      "content": "Text3.",
      "type": "Generic"
    }
  ],
  "title": "Person One",
  "is_still_participant": true,
  "thread_type": "Regular",
  "thread_path": "inbox/SomeString"
}
```


Is there a way I can use ```jq``` to merge the JSON files so that the ```messages``` attribute is concatenated (order doesn't matter) and the others are left alone?

The result from merging message_1.json and message_2.json would look like this:

messages.json
```
{
  "participants": [
    {
      "name": "Person One"
    },
    {
      "name": "Person Two"
    }
  ],

  "messages": [
    {
      "sender_name": "Person One",
      "timestamp_ms": 0002,
      "content": "Text2.",
      "type": "Generic"
    },
    {
      "sender_name": "Person Two",
      "timestamp_ms": 0001,
      "content": "Text1.",
      "type": "Generic"
    },
    {
      "sender_name": "Person Two",
      "timestamp_ms": 0004,
      "content": "Text4.",
      "type": "Generic"
    },
    {
      "sender_name": "Person One",
      "timestamp_ms": 0003,
      "content": "Text3.",
      "type": "Generic"
    }
  ],
  "title": "Person One",
  "is_still_participant": true,
  "thread_type": "Regular",
  "thread_path": "inbox/SomeString"
}
```

I have 11 JSON files, message_1.json, ..., message_11.json. I would like to merge them all into one ```messages.json``` file of this form containing **all of the messages** across the JSON files. How can I do this using ```jq``` via bash?


