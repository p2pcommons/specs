# Interoperability specification v0.3.0

This document outlines specifications for module storage and indexing to achieve an interoperable application layer. This standardizes the way
information from the peer-to-peer commons is cached on a device, such that applications may be used interchangeably. For instance, a command line interface and a desktop
application may be available; this specification makes changing between the two (or more) applications seamless and efficient.

This specification is versioned using [Semantic Versioning
2.0.0](https://semver.org/); `{MAJOR}.{MINOR}.{PATCH}`. This specification may change fundamentally over time, but always with due notice.

This document is available under the [CC0 Public Domain
Dedication](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are used as described in [RFC
2119](https://www.ietf.org/rfc/rfc2119.txt).

Whenever modules are referred to in this specification, this only refers to valid modules in accordance with the [Module specification](./module.md).

## Settings

Settings MUST be stored in `~/.p2pcommons/settings.json`, in a valid JSON object.

The JSON object MAY contain the following type/name value pairs:

| Type            | Value   | Default  | Note                                                                                                                            |
|-----------------|---------|----------|---------------------------------------------------------------------------------------------------------------------------------|
| network-depth   | integer | 2        | -                                                                                                                               |
| default-profile | string  | -        | 64 character hash referring to writable module of `type: profile`                                                               |
| keys            | string  | `~/.hyperdrive` | Location of **private** Hypercore keys                                                                                                            |
| sparse          | boolean | true     |  [Hyperdrive option for sparse replication](https://github.com/mafintosh/hyperdrive#var-archive--hyperdrivestorage-key-options) |
| sparseMetadata  | boolean | true     | Hyperdrive option for sparse metadata replication                                                                               |

Default values are RECOMMENDED values to harmonize operations between applications. Application specific settings MUST be stored in their own files.

## Module replication

Modules MUST be replicated and managed locally using the [`hyperdrive-daemon`](https://github.com/hypercore-protocol/hyperdrive-daemon).

A local LevelDB database of cached modules MUST be kept in `~/.p2pcommons/`, in accordance with the following [avro schemas](https://avro.apache.org/docs/1.8.1/spec.html) for validation. These schemas are in accordance with the modules specification, [values and names section](modules.md#namevalues).

#### Content schema

```json
{
  "name": "Content",
  "namespace": "P2PCommons",
  "type": "record",
  "fields": [
    {
      "name": "title",
      "type": {
        "type": "string",
        "logicalType": "title"
      }
    },
    {"name": "description", "type": "string"},
    {
      "name": "url",
      "type": {
        "type": "string",
        "logicalType": "dat-url"
      }
    },
    {
      "name": "links",
      "type": {
        "namespace": "P2PCommons",
        "type": "record",
        "fields": [
          {
            "name": "license",
            "type": {
              "type": "array",
              "items" : {
                "type": "record",
                "fields": [
                  {
                    "type": "string",
                    "name": "href"
                  }
                ]
              }
            }
          },
          {
            "name": "spec",
            "type": {
              "type": "array",
              "items" : {
                "type": "record",
                "fields": [
                  {
                    "type": "string",
                    "name": "href"
                  }
                ]
              }
            }
          }
        ]
      }
    },
    {
      "name": "p2pcommons",
      "type": {
        "type": "record",
        "fields": [
          {
            "name": "type",
            "type": {
              "type": "enum",
              "symbols": ["content", "profile"]
            }
          },
          {"name": "subtype", "type": "string"},
          {
            "name": "main",
            "type": {
              "type": "string",
              "logicalType": "path"
            }
          },
          {
            "name": "authors",
            "type": {
              "type": "array",
              "items": {
                "type": "string",
                "logicalType": "dat-url"
              }
            },
            "default": []
          },
          {
            "name": "parents",
            "type": {
              "type": "array",
              "items": {
                "type": "string",
                "strict": true,
                "logicalType": "dat-versioned-url"
              }
            },
            "default": []
          }
        ]
      }
    }
  ]
}

```

#### Profile schema

```json
{
  "name": "Profile",
  "namespace": "P2PCommons",
  "type": "record",
  "fields": [
    {
      "name": "title",
      "type": {
        "type": "string",
        "logicalType": "title"
      }
    },
    {"name": "description", "type": "string"},
    {
      "name": "url",
      "type": {
        "type": "string",
        "logicalType": "dat-url"
      }
    },
    {
      "name": "links",
      "type": {
        "namespace": "P2PCommons",
        "type": "record",
        "fields": [
          {
            "name": "license",
            "type": {
              "type": "array",
              "items" : {
                "type": "record",
                "fields": [
                  {
                    "type": "string",
                    "name": "href"
                  }
                ]
              }
            }
          },
          {
            "name": "spec",
            "type": {
              "type": "array",
              "items" : {
                "type": "record",
                "fields": [
                  {
                    "type": "string",
                    "name": "href"
                  }
                ]
              }
            }
          }
        ]
      }
    },
    {
      "name": "p2pcommons",
      "type": {
        "type": "record",
        "fields": [
          {
            "name": "type",
            "type": {
              "type": "enum",
              "symbols": ["content", "profile"]
            }
          },
          {"name": "subtype", "type": "string"},
          {
            "name": "main",
            "type": {
              "type": "string",
              "logicalType": "path"
            }
          },
          {
            "name": "avatar",
            "type": {
              "type": "string",
              "logicalType": "path"
            }
          },
          {
            "name": "follows",
            "type": {
              "type": "array",
              "items": {
                "type": "string",
                "logicalType": "dat-versioned-url"
              }
            },
            "default": []
          },
          {
            "name": "contents",
            "type": {
              "type": "array",
              "items": {
                "type": "string",
                "logicalType": "dat-versioned-url"
              }
            },
            "default": []
          }
        ]
      }
    }
  ]
}
```
