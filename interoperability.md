# Interoperability specification v0.1.0

This document outlines specifications for module storage and indexing to achieve an interoperable application layer. This standardizes the way 
information from the peer-to-peer commons is cached on a device, such that applications may be used interchangeably. For instance, a command line interface and a desktop
application may be available; this specification makes changing between the two (or more) applications seamless and efficient.

This specification is versioned using [Semantic Versioning
2.0.0](https://semver.org/); `{MAJOR}.{MINOR}.{PATCH}` and is now at
`v0.1.0`. This specification may change fundamentally over time, but always with due notice.

This document is available under the [CC0 Public Domain
Dedication](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are used as described in [RFC
2119](https://www.ietf.org/rfc/rfc2119.txt).

Whenever modules are referred to in this specification, this only refers to valid modules in accordance with the [Module specification](./module.md).

## Storage location

All information described in this specification MUST be stored in the folder `~/.p2pcommons` (i.e., in the .p2pcommons folder in the user's home directory).

## Caching modules

An active module MUST be cached in a folder that indicates its own Dat archive key (i.e., `~/.p2pcommons/<hash>`). 

An active versioned module MUST be cached in a folder that indicates its own Dat archive key and its version (i.e., `~/.p2pcommons/<hash>+<version>`). This folder tree MUST be read-only.

## Global settings

Settings MUST be stored in `~/.p2pcommons/settings.json`, in a valid JSON object. 

The JSON object may contain the following type/name value pairs:

| Type            | Value   | Default  | Note                                                                                                                            |
|-----------------|---------|----------|---------------------------------------------------------------------------------------------------------------------------------|
| network-depth   | integer | 2        | -                                                                                                                               |
| default-profile | string  | -        | 64 character hash referring to writable module of `type: profile`                                                               |
| keys            | string  | `~/.dat` | Location of Dat keys                                                                                                            |
| sparse          | boolean | true     |  [Hyperdrive option for sparse replication](https://github.com/mafintosh/hyperdrive#var-archive--hyperdrivestorage-key-options) |
| sparseMetadata  | boolean | true     | Hyperdrive option for sparse metadata replication                                                                               |

Default values are RECOMMENDED values to harmonize operations between applications.

## Indexing modules

Module metadata MUST be indexed into one valid JSON object in the file `~/.p2pcommons/index.json`.
