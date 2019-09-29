# Module specifications v0.1.1

This document outlines specifications for module initialization,
validation, registration, and verification on the Dat network. It is a
derivative of previous peer-reviewed publications
[[1](https://doi.org/10.3390/publications6020021),
[2](https://doi.org/10.3390/publications7020040)] that may be
referenced for more conceptual information.

This specification is versioned using [Semantic Versioning
2.0.0](https://semver.org/); `{MAJOR}.{MINOR}.{PATCH}` and is now at
`v0.1.0`. This specification formulates bare minimum specifications to
reduce the risk of major, backwards incompatible changes. Please note
that this specification is downstream from the [Dat
protocol](https://www.datprotocol.com/).

This document is available under the [CC0 Public Domain
Dedication](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are used as described in [RFC
2119](https://www.ietf.org/rfc/rfc2119.txt).

## Initialization

One module is defined as a Dat archive (a folder on a machine)
containing [module metadata](#module-metadata) in accordance with this
specification.

A Dat archive is initialized when it contains a valid
[Hyperdrive](https://github.com/mafintosh/hyperdrive/) and has a valid
Dat archive key (64 character hash). 

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

`dat.json` MUST be a valid singular JSON object and MUST NOT be an
array of JSON objects.

The JSON object in `dat.json` MUST contain the name/value pairs
`title`, `description`, `url`, `type`, `main`, and
`license`.

If `type: content`, the JSON object MUST include the name/value
pairs `authors` and `parents`.

If `type: profile`, the JSON object MUST include the name/value
pairs `follows` and `contents`. 

For legibility and consistency across implementations, the ordering of
the name/value is RECOMMENDED to be in the order presented in the
specification.

Other name/value pairs MAY be added, but MUST NOT conflict with those
outlined in this specification.

### Name/values

The table declares the value types, adds recommended regular
expressions to validate some of the values. Below the table, the value
conditions are specified per name.

| name          | value            | RECOMMENDED regex                                                                                   |
| ------------- | ---------------- | --------------------------------------------------------------------------------------------------- |
| `title`       | string           |                                                                                                     |
| `description` | string           |                                                                                                     |
| `url`         | string           | [`^(dat:\/\/)?(\w{64})$`](https://regex101.com/r/naEFVg/2)                                          |
| `type`        | string           | [`(profile OR content)$`](https://regex101.com/r/RRKb5N/1)                                              |
| `main`        | string           | [`^((?!\/) OR (\.\/))(?!~ OR \.).*(?<!\/)$`](https://regex101.com/r/MZXJnK/1)                             |
| `license`     | string, object   |                                                                                                     |
| `authors`     | array of strings | [`^(dat:\/\/)?(\w{64})$`](https://regex101.com/r/naEFVg/2)                                          |
| `parents`     | array of strings | [`^(dat:\/\/)?(\w{64})(\+\d+)$`](https://regex101.com/r/naEFVg/3)                                   |
| `follows`     | array of strings | [`^(dat:\/\/)?(\w{64})(\+\d+)?$`](https://regex101.com/r/naEFVg/4)                                  |
| `contents`     | array of strings | [`^(dat:\/\/)?(\w{64})(\+\d+)?$`](https://regex101.com/r/naEFVg/4)                                  |

`title` and `description` MUST be strings, although those strings MAY
be empty (`''`; see also [Registration](#registration)). 

`url` MUST be a string containing the non-versioned Dat archive key
equivalent to the Dat archive key of the module itself.

`type` MUST be a string ending in either `content` or `profile`. This
way, it MAY be prefixed by application or user-specific
strings. 

`main` MUST be a string containing a relative path but MUST NOT refer
to a relative home or relative parent directory. The relative path
SHOULD refer to a valid relative file within the Dat archive (see also
[Registration](#registration)).

`license` MUST be a string referring to the CC0 1.0 Public Domain
Dedication and MAY be nested in another name/value pair. This
`license` mitigates legal uncertainty and encourages the behavior of
distributing and reusing content on a peer-to-peer basis. The string
SHOULD contain the URL of the [legal
code](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
<!-- nesting makes it compatible with Beaker woo
https://beakerbrowser.com/docs/apis/manifest.html -->

`authors` (specific to `type: content`) MUST be an array of strings
containing non-versioned Dat archive keys. These Dat archive keys
SHOULD each refer to a valid module of `type: profile` (see also
[Verification](#verification)).

`parents` (specific to `type: content`) MUST be an array of strings
containing versioned Dat archive keys. These versioned Dat archive
keys SHOULD each refer to a valid module of `type: content`.
<!-- it is RECOMMENDED to only allow verified parents? -->

`follows` (specific to `type: profile`) MUST be an array of strings
containing Dat archive keys and MAY be versioned (freeze
follow). These Dat archive keys SHOULD each refer to a valid module of
`type: profile`.

`contents` (specific to `type: profile`) MUST be an array of strings
containing Dat archive keys that SHOULD be versioned but MAY be
non-versioned (public drafts). These Dat archive keys SHOULD each
refer to a valid module of `type: content`.

## Registration

Registered modules must be of `type: content` and MUST be registered
to modules of `type: profile`.

The destination module MUST valid and writable. The metadata of the
destination module SHOULD be valid prior to registration.

Registration MUST result in the addition of a valid Dat archive key to
the `modules` property of the destination module. The registered Dat
archive key MUST be that of the specified origin module.

Registration MUST NOT occur when the origin module contains invalid
metadata. Registration SHOULD NOT occur if the `title` is empty or
when `authors` is empty.

It is RECOMMENDED to verify whether the origin module contains the
destination module's Dat archive key in the `authors` property (see
also [Verification](#verification)).

## Verification

The origin module to verify MUST be of `type: content` and
verification MUST occur on a specific version (i.e., versioned Dat
archive key).<!--  Non-versioned origin modules MUST be deemed  -->
<!-- unverifiable. -->
<!-- eg call them drafts -->

The destination module in verification MUST be of `type: profile` and
SHOULD be the latest version (i.e., non-versioned Dat archive
key). Verification MUST NOT be based on the outcome of verification
across multiple versions (i.e., verification MUST be version
specific).

A version specific origin module MUST be deemed verified when all its
`authors` (destination) modules include the versioned origin module in
their `modules` property. 

<!-- When not all destination modules include the versioned origin module -->
<!-- in their `modules` property,  -->

Due to the peer-to-peer nature of the Dat protocol and its
cryptographic signing, this two-way verification is the highest degree
of trust that is implemented in this specification.

<!-- + It is non-breaking to loosen conditions or to add properties later on, so I erred on the side of strictness and parsimony -->
<!-- + I am being non-specific about Dat protocol requirements to allow for flexibility down the line? I mean, I -->

