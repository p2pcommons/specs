# Module specifications v0.9.3

This document outlines specifications for module [initialization](#initialization),
[metadata validation](#metadata), [registration](#registration), [verification](#verification), and for [module files](#files). It is a
derivative of previous peer-reviewed publications
[[1](https://doi.org/10.3390/publications6020021),
[2](https://doi.org/10.3390/publications7020040)] that may be
referenced for more conceptual information.

This specification assumes a JavaScript environment in describing
types.

This specification is versioned using [Semantic Versioning
2.0.0](https://semver.org/); `{MAJOR}.{MINOR}.{PATCH}`. This 
specification formulates bare minimum specifications to reduce the
risk of major, backwards incompatible changes. Please note
that this specification is downstream from the [Hypercore
protocol](https://hypercore-protocol.org/).

This document is available under the [CC0 Public Domain
Dedication](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are used as described in [RFC
2119](https://www.ietf.org/rfc/rfc2119.txt). Examples of
up-to-specification metadata are included in the [Examples](#examples)
section.

## Initialization

One module is defined as a [Hyperdrive](https://github.com/mafintosh/hyperdrive/) (a folder on a machine)
containing [module metadata](#module-metadata) in accordance with this
specification.

A Hyperdrive is initialized when it has emitted the "ready" event
and thus has a public key (64 character hash). We utilize Hyperdrive `v10`.

A versioned Hyperdrive key MUST be suffixed with `+{vers}` where `vers`
is an integer denoting the Hyperdrive version. A non-versioned Hyperdrive
key MUST NOT be suffixed with a version.

When a new module is initialized, a new folder MUST be created and
that folder SHOULD be initialized as a Hyperdrive. The folder MUST be
initialized as a Hyperdrive for any actions to be taken on the module
(e.g., [Registration](#registration), [Verification](#verification)).

A newly initialized module SHOULD NOT be a direct clone of another
Hyperdrive, but MAY be if and only if the old Hyperdrive feeds are
completely deleted and a new Hyperdrive is initialized. It is
RECOMMENDED to first initialize a new module and copy files into it.

## Metadata

Each module MUST contain a `index.json` file in its top directory
(`./index.json`). This file SHOULD be encoded UTF-8.

### Object structure

`index.json` MUST be valid JSON consisting of a singular object and MUST 
NOT be an array of objects. 

The `index.json` object MUST contain the key/value pairs `title`,
`description`, `url`, `links`, `p2pcommons`.

`links` MUST be an object with string keys and array values. It MUST
contain the key/value pairs `license`, and `spec`, where their arrays
MUST contain an object with the `href` key paired with a URL string
(other key/value pairs MAY be included from the [Weblink
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
| `title`               | string           | [`^(?!\s*$).{1,300}$`](https://regex101.com/r/q7SL6z/1)                                             |
| `description`         | string           |                                                                                                     |
| `url`                 | string           | [`^(hyper:\/\/)([a-fA-F0-9]{64})$`](https://regex101.com/r/naEFVg/8)                                          |
| `links.license`       | array of objects |                                                                                                     |
| `links.spec`          | array of objects |                                                                                                     |
| `p2pcommons.type`     | string           | [`(profile OR content)$`](https://regex101.com/r/RRKb5N/1)                                          |
| `p2pcommons.subtype`  | string           | [`^[A-Za-z0-9]*$`](https://regex101.com/r/hDRGfc/3)                                                            |
| `p2pcommons.main`     | string           |                                                                                                     |
| `p2pcommons.avatar`   | string           |                                                                                                     |
| `p2pcommons.authors`  | array of strings | [`^([a-fA-F0-9]{64})$`](https://regex101.com/r/GQOim5/11)                                          |
| `p2pcommons.parents`  | array of strings | [`^([a-fA-F0-9]{64})(\+\d+)$`](https://regex101.com/r/GQOim5/12)                                   |
| `p2pcommons.follows`  | array of strings | [`^([a-fA-F0-9]{64})(\+\d+)?$`](https://regex101.com/r/GQOim5/13)                                  |
| `p2pcommons.contents` | array of strings | [`^([a-fA-F0-9]{64})(\+\d+)?$`](https://regex101.com/r/GQOim5/14)                                  |

`title` and `description` MUST be strings. `title` MUST contain a
string between 1 and 300 characters long and MUST NOT consist only
of whitespace characters. `description` MAY be empty (`''`; see also
[Registration](#registration)).

`url` MUST be a string starting with the `hyper://` protocol prefix,
followed by the non-versioned Hyperdrive key equivalent to the Hyperdrive key
of the module itself.

`links.license` MUST contain an object in an array with key/value pair
`href` referring to the CC0 1.0 Public Domain Dedication [legal code
URL](https://creativecommons.org/publicdomain/zero/1.0/legalcode)
(`https://creativecommons.org/publicdomain/zero/1.0/legalcode`). `links.license`
mitigates legal uncertainty and encourages the behavior of
distributing and reusing content on a peer-to-peer basis.

`links.spec` MUST contain an object in an array with key/value pair
`href` referring to the active module specification URL (e.g.,
`https://p2pcommons.com/specs/module/x.x.x`).

`p2pcommons.type` MUST be a string containing either `content` or
`profile`.

`p2pcommons.subtype` MAY be an empty string, but MUST NOT be a string
that includes characters beyond the standard alphanumeric set (e.g.,
hyphens, colons, etc). These MAY be application specific, but it is
RECOMMENDED to use WikiData item identifiers for consistent
disambiguation.

If `p2pcommons.type` is `content`, `p2pcommons.main` MUST be a string
containing one relative path. If `p2pcommons.type` is `profile`,
`p2pcommons.main` MUST be an empty string or a string containing one
relative path. The path MUST refer to an existing file within the Hyperdrive,
MUST NOT refer to a relative home or relative parent directory and MUST NOT
refer to a dotfile (e.g., `./.example.json`). The `./` part of a relative path
MAY be included.

If included, `p2pcommons.avatar` MUST be a string containing one relative path
that MUST NOT refer to a relative home or relative parent directory. The
relative path SHOULD refer to a valid image file within the Hyperdrive
(see also [Registration](#registration)). The `./` part of a relative path
MAY be included.

`p2pcommons.authors` (specific to `p2pcommons.type: content`) MUST be
an array of strings containing unique non-versioned Hyperdrive keys. These
Hyperdrive keys SHOULD each refer to a valid module of
`p2pcommons.type: profile` (see also [Verification](#verification)).

`p2pcommons.parents` (specific to `p2pcommons.type: content`) MUST be
an array of strings containing unique versioned Hyperdrive keys. These
versioned Hyperdrive keys MUST each refer to a valid module of
`p2pcommons.type: content` and MUST be registered by at least one of 
its listed authors' profiles at the time this value is set. The network state
SHOULD be up-to-date before determining registration status.
If `p2pcommons.parents` contains this module's own Hyperdrive
key, it MUST refer to a preceding version.

`p2pcommons.follows` (specific to `p2pcommons.type: profile`) MUST be
an array of strings containing unique Hyperdrive keys that MAY be versioned
(freeze follow). These Hyperdrive keys SHOULD each refer to a valid
module of `type: profile`, MUST NOT refer to this module's own Hyperdrive key
at any version, and MAY refer to multiple versions of the same Hyperdrive key.

`p2pcommons.contents` (specific to `p2pcommons.type: profile`) MUST be
an array of strings containing unique Hyperdrive keys that SHOULD be
versioned but MAY be non-versioned (public drafts). These Hyperdrive
keys SHOULD each refer to a valid module of `p2pcommons.type:
content` and MAY refer to multiple versions of the same Hyperdrive key. 

## Registration

To be registered modules must be of `p2pcommons.type: content` (i.e., origin
module) and MUST be registered to modules of `p2pcommons.type:
profile` (i.e., destination module).

Registration MUST NOT occur if the origin or destination modules are invalid.
Registration SHOULD NOT occur if `p2pcommons.authors` is empty.
The destination module MUST be writable.

Registration MUST result in the addition of a valid Hyperdrive key to
the `p2pcommons.contents` property of the destination module. The
registered Hyperdrive key MUST be that of the specified origin
module.

It is RECOMMENDED to verify whether the origin module contains the
destination module's Hyperdrive key in the `authors` property (see
also [Verification](#verification)).

## Verification

The origin module to verify MUST be of `p2pcommons.type: content` and
verification MUST occur on a specific version (i.e., versioned Hyperdrive
key).<!-- Non-versioned origin modules MUST be deemed --> <!--
unverifiable. --> <!-- eg call them drafts -->

The destination module in verification MUST be of `p2pcommons.type:
profile` and SHOULD be the latest version (i.e., non-versioned Hyperdrive
key). Verification MUST NOT be based on the outcome of
verification across multiple versions (i.e., verification MUST be
version specific).

A version specific origin module MUST be deemed verified when all its
`p2pcommons.authors` (destination) modules include the versioned
origin module in their `p2pcommons.contents` property.

Due to the peer-to-peer nature of the Hypercore protocol and its
cryptographic signing, this two-way verification is the highest degree
of trust that is implemented in this specification.

## Files

All files in a module MUST be of an [open file format](https://en.wikipedia.org/wiki/List_of_open_formats); these file formats can be used and implemented by anyone. Wikipedia and Wikidata SHOULD be considered the authority in determining what file formats are considered open.

In case of text based files, we RECOMMEND Hypertext Markup Language (HTML) files. In case of multiple versions of the same text file, the HTML version SHOULD be preferred as the main file.

In order to remain self-contained, all references to local files SHOULD be valid relative file paths within the module itself. Files MAY reference URLs. It is RECOMMENDED to use references to module files instead of URLs as much as possible. Where URLs cannot be avoided, it is RECOMMENDED to use [persistent URLs](https://en.wikipedia.org/wiki/Persistent_uniform_resource_locator) as much as possible.

## Examples

The links used in the examples do not work and serve illustrative purposes only. Where version `x.x.x` is displayed, it should read the actual version number of the specification.

### `p2pcommons.type === 'content'`

```js
{
  "title": "Content example",
  "description": "",
  "url": "hyper://00a4f2f18bb6cb4e9ba7c2c047c8560d34047457500e415d535de0526c6b4f23",
  "links": {
     "license": [{"href": "https://creativecommons.org/publicdomain/zero/1.0/legalcode"}],
     "spec": [{"href": "https://p2pcommons.com/specs/module/x.x.x"}]
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
  "url": "hyper://cca6eb69a3ad6104ca31b9fee7832d74068db16ef2169eaaab5b48096e128342",
  "links": {
     "license": [{"href": "https://creativecommons.org/publicdomain/zero/1.0/legalcode"}],
     "spec": [{"href": "https://p2pcommons.com/specs/module/x.x.x"}]
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
