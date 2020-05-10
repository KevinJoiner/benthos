---
title: Bloblang Beta
author: Ashley Jeffs
author_url: https://github.com/Jeffail
author_image_url: /img/ash.jpg
description: Available in v3.13
keywords: [
    "benthos",
    "bloblang",
    "go",
    "golang",
    "stream processor",
    "mapping",
]
tags: [ "Bloblang" ]
---

As of this weekend (and Benthos v3.13) you can now use a [`bloblang` processor](/docs/components/processors/bloblang) and complementary [condition](/docs/components/conditions/bloblang). These components are in a beta phase, which means that based on feedback the mapping language might change in minor ways in upcoming minor releases.

<!--truncate-->

## The Motivation

[In the last post][post.sneak_peek] I outlined my motivations for experimenting with a mapping language. Words are stupid and boring and so to illustrate why a mapping language kicks ass here's a config example using the old processors compared to the new one. Keep in mind that the new version is simpler _and_ performs better.

Using old processors:

```yaml
pipeline:
  processors:
  - metadata:
      operator: set
      key: bar
      value: ${!json_field:foo.bar} 

  - json:
      operator: delete
      path: foo.bar

  - json:
      operator: set
      path: foo.topic
      value: ${!metadata:topic} 

  - metadata:
      operator: delete
      key: topic

  - conditional:
      condition:
        jmespath:
          query: "foo.baz == 'thing'"
      processors:
      - json:
          operator: set
          path: foo.thing_id
          value: ${!uuid_v4}
```

Using Bloblang:

```yaml
pipeline:
  processors:
  - bloblang: |
      root = this

      foo.topic = meta("topic")
      meta topic = deleted()

      meta bar = foo.bar
      foo.bar = deleted()

      foo.thing_id = match {
        foo.baz == "thing" => uuid_v4()
      }
```

My ultimate intention is to completely eradicate the need for a `json`, `metadata` and `text` processor, as well as a range of others. However, I'll need as much help as possible to get the language right, so please consider testing and feeding back ([Github issues][gh.issues], or [Gitter][gitter]) for the good of blobkind.

[processor.bloblang]: /docs/components/processors/bloblang
[condition.bloblang]: /docs/components/conditions/bloblang
[post.sneak_peek]: /blog/2020/04/18/sneak-peek-at-bloblang
[gh.issues]: https://github.com/Jeffail/benthos/issues
[gitter]: https://gitter.im/jeffail-benthos/community