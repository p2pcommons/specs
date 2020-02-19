# Module specifications v0.2.3

This document outlines specifications for module initialization,
validation, registration, and verification on the Dat network. It is a
derivative of previous peer-reviewed publications
[[1](https://doi.org/10.3390/publications6020021),
[2](https://doi.org/10.3390/publications7020040)] that may be
referenced for more conceptual information.

This specification assumes a JavaScript environment in describing
types.

This specification is versioned using [Semantic Versioning
2.0.0](https://semver.org/); `{MAJOR}.{MINOR}.{PATCH}` and is now at
`v0.2.3`. This specification formulates bare minimum specifications to
reduce the risk of major, backwards incompatible changes. Please note
that this specification is downstream from the [Dat
protocol](https://www.datprotocol.com/).

This document is available under the [CC0 Public Domain
Dedication](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are used as described in [RFC
2119](https://www.ietf.org/rfc/rfc2119.txt). Examples of
up-to-specification metadata are included in the [Examples](#examples)
section.

## Initialization

One module is defined as a Dat archive (a folder on a machine)
containing [module metadata](#module-metadata) in accordance with this
specification.

A Dat archive is initialized when it contains a valid
[Hyperdrive](https://github.com/mafintosh/hyperdrive/) and has a valid
Dat archive key (64 character hash). We utilize Hyperdrive `v10`
cores.

In general, Dat archive keys in this specification MAY be prefixed by
the protocol indicator `dat://`, and MAY be suffixed by `+{vers}`
where `vers` is an integer denoting the Dat archive version. Trailing
slashes SHOULD be omitted.

When a new module is initialized, a new folder MUST be created and
that folder SHOULD be initialized as a Dat archive. The folder MUST be
initialized as a Dat archive for any actions to be taken on the module
(e.g., [Registration](#registration), [Verification](#verification)).

A newly initialized module SHOULD NOT be a direct clone of another Dat
archive, but MAY be if and only if the old Dat archive feeds are
completely deleted and a new Dat archive is initialized. It is
RECOMMENDED to first initialize a new module and copy files into it.

## Metadata

Each module MUST contain a `dat.json` file in its top directory
(`./dat.json`). This file SHOULD be encoded UTF-8.

### Object structure

`dat.json` MUST be valid JSON consisting of a singular object and MUST 
NOT be an array of objects. 

The `dat.json` object MUST contain the key/value pairs `title`,
`description`, `url`, `links`, `p2pcommons`.

`links` MUST be an object with string keys and array values. It MUST
contain the key/value pairs `license`, and `spec`, where their arrays
MUST contain an object with the `href` key paired with a URL string
(other key/value pairs may be included from the [Weblink
specification](https://html.spec.whatwg.org/multipage/semantics.html#the-link-element)).

`p2pcommons` MUST have an object value containing `type`, `subtype`,
`main`. If `p2pcommons.type: content`, the object MUST include the
key/value pairs `authors` and `parents`. If `p2pcommons.type:
profile`, the object MUST include the key/value pairs `follows` and
`contents`; it is RECOMMENDED to include the `p2pcommons.avatar` key/value pair for profiles.

For legibility and consistency across implementations, the ordering of
the key/value is RECOMMENDED to be in the order presented in the
specification.

Other key/value pairs MAY be added, but MUST NOT conflict with those
outlined in this specification.

### Key/values

The table declares the value types, adds recommended regular
expressions to validate some of the values. Below the table, the value
conditions are specified per name.

| name                  | value            | RECOMMENDED regex                                                                                   |
| -------------         | ---------------- | --------------------------------------------------------------------------------------------------- |
| `title`               | string           |                                                                                                     |
| `description`         | string           |                                                                                                     |
| `url`                 | string           | [`^(dat:\/\/)?(\w{64})$`](https://regex101.com/r/naEFVg/2)                                          |
| `links.license`       | array of objects |                                                                                                     |
| `links.spec`          | array of objects |                                                                                                     |
| `p2pcommons.type`     | string           | [`(profile OR content)$`](https://regex101.com/r/RRKb5N/1)                                          |
| `p2pcommons.subtype`  | string           | [`^\w+$`](https://regex101.com/r/hDRGfc/1)                                                            |
| `p2pcommons.main`     | string           | [`^((?!\/) OR (\.\/))(?!~ OR \.).*(?<!\/)$`](https://regex101.com/r/MZXJnK/1)                       |
| `p2pcommons.avatar`     | string           | [`^((?!\/) OR (\.\/))(?!~ OR \.).*(?<!\/)$`](https://regex101.com/r/MZXJnK/1)                       |
| `p2pcommons.authors`  | array of strings | [`^(dat:\/\/)?(\w{64})$`](https://regex101.com/r/naEFVg/2)                                          |
| `p2pcommons.parents`  | array of strings | [`^(dat:\/\/)?(\w{64})(\+\d+)$`](https://regex101.com/r/naEFVg/3)                                   |
| `p2pcommons.follows`  | array of strings | [`^(dat:\/\/)?(\w{64})(\+\d+)?$`](https://regex101.com/r/naEFVg/4)                                  |
| `p2pcommons.contents` | array of strings | [`^(dat:\/\/)?(\w{64})(\+\d+)?$`](https://regex101.com/r/naEFVg/4)                                  |

`title` and `description` MUST be strings. `title` MUST contain a
string (max character length 300) and `description` MAY be empty (`''`; see also
[Registration](#registration)).

`url` MUST be a string containing the non-versioned Dat archive key
equivalent to the Dat archive key of the module itself.

`links.license` MUST contain an object in an array with key/value pair
`href` referring to the CC0 1.0 Public Domain Dedication [legal code
URL](https://creativecommons.org/publicdomain/zero/1.0/legalcode)
(`https://creativecommons.org/publicdomain/zero/1.0/legalcode`). `links.license`
mitigates legal uncertainty and encourages the behavior of
distributing and reusing content on a peer-to-peer basis.  <!--
nesting makes it compatible with Beaker woo
https://beakerbrowser.com/docs/apis/manifest.html -->

`links.spec` MUST contain an object in an array with key/value pair
`href` referring to the active module specification URL (e.g.,
`https://p2pcommons.com/specs/module/0.2.3`).

`p2pcommons.type` MUST be a string containing either `content` or
`profile`.

`p2pcommons.subtype` MAY be an empty string, but MUST NOT be a string
that includes characters beyond the standard alphanumeric set (e.g.,
hyphens, colons, etc). These MAY be application specific, but it is
RECOMMENDED to use WikiData item identifiers for consistent
disambiguation. <!-- maybe start compiling an easy to use database for
this -->

`p2pcommons.main` MUST be a string containing one relative path but MUST
NOT refer to a relative home or relative parent directory. The
relative path SHOULD refer to a valid relative file within the Dat
archive (see also [Registration](#registration)). The `./` part of a
relative path MAY be included.

If included, `p2pcommons.avatar` MUST be a string containing one relative path
that MUST NOT refer to a relative home or relative parent directory. The
relative path SHOULD refer to a valid image file within the Dat
archive (see also [Registration](#registration)). The `./` part of a
relative path MAY be included.

`p2pcommons.authors` (specific to `p2pcommons.type: content`) MUST be
a unique array of strings containing non-versioned Dat archive keys. These
Dat archive keys SHOULD each refer to a valid module of
`p2pcommons.type: profile` (see also [Verification](#verification)).

`p2pcommons.parents` (specific to `p2pcommons.type: content`) MUST be
a unique array of strings containing versioned Dat archive keys. These
versioned Dat archive keys SHOULD each refer to a valid module of
`p2pcommons.type: content` and MUST NOT contain this module's
own Dat archive key.
<!-- it is RECOMMENDED to only allow verified parents? -->

`p2pcommons.follows` (specific to `p2pcommons.type: profile`) MUST be
a unique array of strings containing Dat archive keys and MAY be versioned
(freeze follow). These Dat archive keys SHOULD each refer to a valid
module of `type: profile` and MUST NOT contain this module's
own Dat archive key.

`p2pcommons.contents` (specific to `p2pcommons.type: profile`) MUST be
a unique array of strings containing Dat archive keys that SHOULD be
versioned but MAY be non-versioned (public drafts). These Dat archive
keys SHOULD each refer to a valid module of `p2pcommons.type:
content`.

## Registration

Registered modules must be of `p2pcommons.type: content` (i.e., origin
module) and MUST be registered to modules of `p2pcommons.type:
profile` (i.e., destination module).

The destination module MUST be valid and writable. The `p2pcommons.main` MUST refer to an existing path. The metadata of the
destination module SHOULD be valid prior to registration.

Registration MUST result in the addition of a valid Dat archive key to
the `p2pcommons.contents` property of the destination module. The
registered Dat archive key MUST be that of the specified origin
module.

Registration MUST NOT occur when the origin module contains invalid
metadata. Registration SHOULD NOT occur if the `title` is empty or
when `p2pcommons.authors` is empty.

It is RECOMMENDED to verify whether the origin module contains the
destination module's Dat archive key in the `authors` property (see
also [Verification](#verification)).

## Verification

The origin module to verify MUST be of `p2pcommons.type: content` and
verification MUST occur on a specific version (i.e., versioned Dat
archive key).<!-- Non-versioned origin modules MUST be deemed --> <!--
unverifiable. --> <!-- eg call them drafts -->

The destination module in verification MUST be of `p2pcommons.type:
profile` and SHOULD be the latest version (i.e., non-versioned Dat
archive key). Verification MUST NOT be based on the outcome of
verification across multiple versions (i.e., verification MUST be
version specific).

A version specific origin module MUST be deemed verified when all its
`p2pcommons.authors` (destination) modules include the versioned
origin module in their `p2pcommons.contents` property.

<!-- When not all destination modules include the versioned origin
module --> <!-- in their `contents` property, -->

Due to the peer-to-peer nature of the Dat protocol and its
cryptographic signing, this two-way verification is the highest degree
of trust that is implemented in this specification.

<!-- + It is non-breaking to loosen conditions or to add properties
later on, so I erred on the side of strictness and parsimony -->
<!-- + I am being non-specific about Dat protocol requirements to
allow for flexibility down the line? I mean, I -->

## Examples

The links used in the examples do not work and serve illustrative purposes only.

### `p2pcommons.type === 'content'`

```js
{
  "title": "Content example",
  "description": "",
  "url": "00a4f2f18bb6cb4e9ba7c2c047c8560d34047457500e415d535de0526c6b4f23",
  "links": {
     "license": [{"href": "https://creativecommons.org/publicdomain/zero/1.0/legalcode"}],
     "spec": [{"href": "https://p2pcommons/specs/module/0.2.2"}]
  },
  "p2pcommons": {
    "type": "content",
    "subtype": "",
    "main": "test-content.html",
    "authors": [
      "cca6eb69a3ad6104ca31b9fee7832d74068db16ef2169eaaab5b48096e128342",
      "f7daadc2d624df738abbccc9955714d94cef656406f2a850bfc499c2080627d4"
    ],
    "parents": [
      "f0abcd6b1c4fc524e2d48da043b3d8399b96d9374d6606fca51182ee230b6b59+12",
      "527f404aa77756b91cba4e3ba9fe30f72ee3eb5eef0f4da87172745f9389d1e5+4032"
    ]
  }
}
```

### `p2pcommons.type === 'profile'`

```js
{
  "title": "Profile example",
  "description": "",
  "url": "cca6eb69a3ad6104ca31b9fee7832d74068db16ef2169eaaab5b48096e128342",
  "links": {
     "license": [{"href": "https://creativecommons.org/publicdomain/zero/1.0/legalcode"}],
     "spec": [{"href": "https://p2pcommons/specs/module/0.2.2"}]
  },
  "p2pcommons": {
    "type": "profile",
    "subtype": "",
    "main": "test-profile.html",
    "avatar": "./test.png",
    "follows": [
      "f7daadc2d624df738abbccc9955714d94cef656406f2a850bfc499c2080627d4"
    ],
    "contents": [
      "00a4f2f18bb6cb4e9ba7c2c047c8560d34047457500e415d535de0526c6b4f23+12"
    ]
  }
}
```
