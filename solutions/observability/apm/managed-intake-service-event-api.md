---
mapped_pages:
  - https://www.elastic.co/guide/en/serverless/current/observability-apm-server-api.html
applies_to:
  serverless:
products:
  - id: cloud-serverless
---

# Managed intake service event API [observability-apm-server-api]

::::{warning}
This API is exclusively for APM agent developers. The vast majority of users should have no reason to interact with this API.
::::

The managed intake service exposes endpoints for:

* [The managed intake service information API](#observability-apm-server-api-server-information-api)
* [Elastic APM events intake API](#observability-apm-server-api-events-intake-api)
* [OpenTelemetry intake API](#observability-apm-server-api-opentelemetry-api)

## Server information API [observability-apm-server-api-server-information-api]

The managed intake service exposes an API endpoint to query general server information. This lightweight endpoint is useful as a server up/down health check.

### Server Information endpoint [api-info-endpoint]

Send an `HTTP GET` request to the server information endpoint:

```bash
https://{hostname}:{port}/
```

This endpoint always returns an HTTP 200.

Requests to this endpoint must be authenticated.

#### Example [api-info-examples]

Example managed intake service information request:

```sh
curl -X POST http://127.0.0.1:8200/ \
  -H "Authorization: ApiKey api_key"

{
  "build_date": "2021-12-18T19:59:06Z",
  "build_sha": "24fe620eeff5a19e2133c940c7e5ce1ceddb1445",
  "publish_ready": true,
  "version": "8.16.1"
}
```

## Events intake API [observability-apm-server-api-events-intake-api]

::::{note}
Most users do not need to interact directly with the events intake API.

::::

The events intake API is what we call the internal protocol that APM agents use to talk to the managed intake service. Agents communicate with the Server by sending events — captured pieces of information — in an HTTP request. Events can be:

* Transactions
* Spans
* Errors
* Metrics

Each event is sent as its own line in the HTTP request body. This is known as [newline delimited JSON (NDJSON)](https://github.com/ndjson/ndjson-spec).

With NDJSON, agents can open an HTTP POST request and use chunked encoding to stream events to the managed intake service as soon as they are recorded in the agent. This makes it simple for agents to serialize each event to a stream of newline delimited JSON. The managed intake service also treats the HTTP body as a compressed stream and thus reads and handles each event independently.

Refer to [Learn about data types](/solutions/observability/apm/data-types.md) to learn more about the different types of events.

### Endpoints [api-events-endpoint]

The managed intake service exposes the following endpoints for Elastic APM agent data intake:

| Name | Endpoint |
| --- | --- |
| APM agent event intake | `/intake/v2/events` |

### Request [api-events-example]

Send an `HTTP POST` request to the managed intake service `intake/v2/events` endpoint:

```bash
https://{hostname}:{port}/intake/v2/events
```

The managed intake service supports asynchronous processing of batches. To request asynchronous processing the `async` query parameter can be set in the POST request to the `intake/v2/events` endpoint:

```bash
https://{hostname}:{port}/intake/v2/events?async=true
```

::::{note}
Since asynchronous processing defers some of the event processing to the background and takes place after the client has closed the request, some errors can’t be communicated back to the client and are logged by the managed intake service. Furthermore, asynchronous processing requests will only be scheduled if the managed intake service can service the incoming request, requests that cannot be serviced will receive an internal error `503` "queue is full" error.

::::

### Response [api-events-response]

On success, the server will respond with a 202 Accepted status code and no body.

Keep in mind that events can succeed and fail independently of each other. Only if all events succeed does the server respond with a 202.

### API Errors [api-events-errors]

There are two types of errors that the managed intake service may return to an agent:

* Event related errors (typically validation errors)
* Non-event related errors

The managed intake service processes events one after the other. If an error is encountered while processing an event, the error encountered as well as the document causing the error are added to an internal array. The managed intake service will only save 5 event related errors. If it encounters more than 5 event related errors, the additional errors will not be returned to agent. Once all events have been processed, the error response is sent.

Some errors, not relating to specific events, may terminate the request immediately. For example: IP rate limit reached, wrong metadata, etc. If at any point one of these errors is encountered, it is added to the internal array and immediately returned.

An example error response might look something like this:

```json
{
  "errors": [
    {
      "message": "<json-schema-err>",   <1>
      "document": "<ndjson-obj>"   <2>
    },{
      "message": "<json-schema-err>",
      "document": "<ndjson-obj>"
    },{
      "message": "<json-decoding-err>",
      "document": "<ndjson-obj>"
    },{
      "message": "too many requests"   <3>
    },
  ],
  "accepted": 2320   <4>
}
```

1. An event related error
2. The document causing the error
3. An immediately returning non-event related error
4. The number of accepted events

If you’re developing an agent, these errors can be useful for debugging.

### Event API Schemas [api-events-schema-definition]

The managed intake service uses a collection of JSON Schemas for validating requests to the intake API.

### Metadata [observability-apm-server-api-metadata]

Every new connection to the managed intake service starts with a `metadata` stanza. This provides general metadata concerning the other objects in the stream.

Rather than send this metadata information from the agent multiple times, the managed intake service hangs on to this information and applies it to other objects in the stream as necessary.

::::{tip}
Metadata is stored under `context` when viewing documents in {{es}}.

::::

#### Metadata Schema [metadata-schema]

The managed intake service uses JSON Schema to validate requests. The specification for metadata is defined on [GitHub](https://github.com/elastic/apm-server/blob/main/docs/spec/v2/metadata.json) and included below.

::::{dropdown} Click to expand the schema
```json
{
  "$id": "docs/spec/v2/metadata",
  "type": "object",
  "properties": {
    "cloud": {
      "description": "Cloud metadata about where the monitored service is running.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "account": {
          "description": "Account where the monitored service is running.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "id": {
              "description": "ID of the cloud account.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "name": {
              "description": "Name of the cloud account.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "availability_zone": {
          "description": "AvailabilityZone where the monitored service is running, e.g. us-east-1a",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "instance": {
          "description": "Instance on which the monitored service is running.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "id": {
              "description": "ID of the cloud instance.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "name": {
              "description": "Name of the cloud instance.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "machine": {
          "description": "Machine on which the monitored service is running.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "type": {
              "description": "ID of the cloud machine.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "project": {
          "description": "Project in which the monitored service is running.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "id": {
              "description": "ID of the cloud project.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "name": {
              "description": "Name of the cloud project.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "provider": {
          "description": "Provider that is used, e.g. aws, azure, gcp, digitalocean.",
          "type": "string",
          "maxLength": 1024
        },
        "region": {
          "description": "Region where the monitored service is running, e.g. us-east-1",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "service": {
          "description": "Service that is monitored on cloud",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "name": {
              "description": "Name of the cloud service, intended to distinguish services running on different platforms within a provider, eg AWS EC2 vs Lambda, GCP GCE vs App Engine, Azure VM vs App Server.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        }
      },
      "required": [
        "provider"
      ]
    },
    "labels": {
      "description": "Labels are a flat mapping of user-defined tags. Allowed value types are string, boolean and number values. Labels are indexed and searchable.",
      "type": [
        "null",
        "object"
      ],
      "additionalProperties": {
        "type": [
          "null",
          "string",
          "boolean",
          "number"
        ],
        "maxLength": 1024
      }
    },
    "network": {
      "description": "Network holds information about the network over which the monitored service is communicating.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "connection": {
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "type": {
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        }
      }
    },
    "process": {
      "description": "Process metadata about the monitored service.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "argv": {
          "description": "Argv holds the command line arguments used to start this process.",
          "type": [
            "null",
            "array"
          ],
          "items": {
            "type": "string"
          },
          "minItems": 0
        },
        "pid": {
          "description": "PID holds the process ID of the service.",
          "type": "integer"
        },
        "ppid": {
          "description": "Ppid holds the parent process ID of the service.",
          "type": [
            "null",
            "integer"
          ]
        },
        "title": {
          "description": "Title is the process title. It can be the same as process name.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      },
      "required": [
        "pid"
      ]
    },
    "service": {
      "description": "Service metadata about the monitored service.",
      "type": "object",
      "properties": {
        "agent": {
          "description": "Agent holds information about the APM agent capturing the event.",
          "type": "object",
          "properties": {
            "activation_method": {
              "description": "ActivationMethod of the APM agent capturing information.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "ephemeral_id": {
              "description": "EphemeralID is a free format ID used for metrics correlation by agents",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "name": {
              "description": "Name of the APM agent capturing information.",
              "type": "string",
              "maxLength": 1024,
              "minLength": 1
            },
            "version": {
              "description": "Version of the APM agent capturing information.",
              "type": "string",
              "maxLength": 1024
            }
          },
          "required": [
            "name",
            "version"
          ]
        },
        "environment": {
          "description": "Environment in which the monitored service is running, e.g. \`production\` or \`staging\`.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "framework": {
          "description": "Framework holds information about the framework used in the monitored service.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "name": {
              "description": "Name of the used framework",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "version": {
              "description": "Version of the used framework",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "id": {
          "description": "ID holds a unique identifier for the running service.",
          "type": [
            "null",
            "string"
          ]
        },
        "language": {
          "description": "Language holds information about the programming language of the monitored service.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "name": {
              "description": "Name of the used programming language",
              "type": "string",
              "maxLength": 1024
            },
            "version": {
              "description": "Version of the used programming language",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          },
          "required": [
            "name"
          ]
        },
        "name": {
          "description": "Name of the monitored service.",
          "type": "string",
          "maxLength": 1024,
          "minLength": 1,
          "pattern": "^[a-zA-Z0-9 _-]+$"
        },
        "node": {
          "description": "Node must be a unique meaningful name of the service node.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "configured_name": {
              "description": "Name of the service node",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "runtime": {
          "description": "Runtime holds information about the language runtime running the monitored service",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "name": {
              "description": "Name of the language runtime",
              "type": "string",
              "maxLength": 1024
            },
            "version": {
              "description": "Name of the language runtime",
              "type": "string",
              "maxLength": 1024
            }
          },
          "required": [
            "name",
            "version"
          ]
        },
        "version": {
          "description": "Version of the monitored service.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      },
      "required": [
        "agent",
        "name"
      ]
    },
    "system": {
      "description": "System metadata",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "architecture": {
          "description": "Architecture of the system the monitored service is running on.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "configured_hostname": {
          "description": "ConfiguredHostname is the configured name of the host the monitored service is running on. It should only be sent when configured by the user. If given, it is used as the event's hostname.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "container": {
          "description": "Container holds the system's container ID if available.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "id": {
              "description": "ID of the container the monitored service is running in.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "detected_hostname": {
          "description": "DetectedHostname is the hostname detected by the APM agent. It usually contains what the hostname command returns on the host machine. It will be used as the event's hostname if ConfiguredHostname is not present.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "hostname": {
          "description": "Deprecated: Use ConfiguredHostname and DetectedHostname instead. DeprecatedHostname is the host name of the system the service is running on. It does not distinguish between configured and detected hostname and therefore is deprecated and only used if no other hostname information is available.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "kubernetes": {
          "description": "Kubernetes system information if the monitored service runs on Kubernetes.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "namespace": {
              "description": "Namespace of the Kubernetes resource the monitored service is run on.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "node": {
              "description": "Node related information",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the Kubernetes Node",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "pod": {
              "description": "Pod related information",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the Kubernetes Pod",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "uid": {
                  "description": "UID is the system-generated string uniquely identifying the Pod.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            }
          }
        },
        "platform": {
          "description": "Platform name of the system platform the monitored service is running on.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      }
    },
    "user": {
      "description": "User metadata, which can be overwritten on a per event basis.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "domain": {
          "description": "Domain of the logged in user",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "email": {
          "description": "Email of the user.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "id": {
          "description": "ID identifies the logged in user, e.g. can be the primary key of the user",
          "type": [
            "null",
            "string",
            "integer"
          ],
          "maxLength": 1024
        },
        "username": {
          "description": "Name of the user.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      }
    }
  },
  "required": [
    "service"
  ]
}
```

::::

#### Kubernetes data [kubernetes-data]

APM agents automatically read Kubernetes data and send it to the managed intake service. In most instances, agents are able to read this data from inside the container. If this is not the case, or if you wish to override this data, you can set environment variables for the agents to read. These environment variable are set via the Kubernetes [Downward API](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#use-pod-fields-as-values-for-environment-variables). Here’s how you would add the environment variables to your Kubernetes pod spec:

```yaml
         - name: KUBERNETES_NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: KUBERNETES_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: KUBERNETES_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: KUBERNETES_POD_UID
            valueFrom:
              fieldRef:
                fieldPath: metadata.uid
```

The table below maps these environment variables to the APM metadata event field:

| Environment variable | Metadata field name |
| --- | --- |
| `KUBERNETES_NODE_NAME` | system.kubernetes.node.name |
| `KUBERNETES_POD_NAME` | system.kubernetes.pod.name |
| `KUBERNETES_NAMESPACE` | system.kubernetes.namespace |
| `KUBERNETES_POD_UID` | system.kubernetes.pod.uid |

### Transactions [observability-apm-server-api-transactions]

Transactions are events corresponding to an incoming request or similar task occurring in a monitored service.

#### Transaction Schema [api-transaction-schema]

The managed intake service uses JSON Schema to validate requests. The specification for transactions is defined on [GitHub](https://github.com/elastic/apm-server/blob/main/docs/spec/v2/transaction.json) and included below.

::::{dropdown} Click to expand the schema
```json
{
  "$id": "docs/spec/v2/transaction",
  "type": "object",
  "properties": {
    "context": {
      "description": "Context holds arbitrary contextual information for the event.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "cloud": {
          "description": "Cloud holds fields related to the cloud or infrastructure the events are coming from.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "origin": {
              "description": "Origin contains the self-nested field groups for cloud.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "account": {
                  "description": "The cloud account or organization id used to identify different entities in a multi-tenant environment.",
                  "type": [
                    "null",
                    "object"
                  ],
                  "properties": {
                    "id": {
                      "description": "The cloud account or organization id used to identify different entities in a multi-tenant environment.",
                      "type": [
                        "null",
                        "string"
                      ]
                    }
                  }
                },
                "provider": {
                  "description": "Name of the cloud provider.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "region": {
                  "description": "Region in which this host, resource, or service is located.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "service": {
                  "description": "The cloud service name is intended to distinguish services running on different platforms within a provider.",
                  "type": [
                    "null",
                    "object"
                  ],
                  "properties": {
                    "name": {
                      "description": "The cloud service name is intended to distinguish services running on different platforms within a provider.",
                      "type": [
                        "null",
                        "string"
                      ]
                    }
                  }
                }
              }
            }
          }
        },
        "custom": {
          "description": "Custom can contain additional metadata to be stored with the event. The format is unspecified and can be deeply nested objects. The information will not be indexed or searchable in Elasticsearch.",
          "type": [
            "null",
            "object"
          ]
        },
        "message": {
          "description": "Message holds details related to message receiving and publishing if the captured event integrates with a messaging system",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "age": {
              "description": "Age of the message. If the monitored messaging framework provides a timestamp for the message, agents may use it. Otherwise, the sending agent can add a timestamp in milliseconds since the Unix epoch to the message's metadata to be retrieved by the receiving agent. If a timestamp is not available, agents should omit this field.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "ms": {
                  "description": "Age of the message in milliseconds.",
                  "type": [
                    "null",
                    "integer"
                  ]
                }
              }
            },
            "body": {
              "description": "Body of the received message, similar to an HTTP request body",
              "type": [
                "null",
                "string"
              ]
            },
            "headers": {
              "description": "Headers received with the message, similar to HTTP request headers.",
              "type": [
                "null",
                "object"
              ],
              "additionalProperties": false,
              "patternProperties": {
                "[.*]*$": {
                  "type": [
                    "null",
                    "array",
                    "string"
                  ],
                  "items": {
                    "type": "string"
                  }
                }
              }
            },
            "queue": {
              "description": "Queue holds information about the message queue where the message is received.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name holds the name of the message queue where the message is received.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "routing_key": {
              "description": "RoutingKey holds the optional routing key of the received message as set on the queuing system, such as in RabbitMQ.",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "page": {
          "description": "Page holds information related to the current page and page referers. It is only sent from RUM agents.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "referer": {
              "description": "Referer holds the URL of the page that 'linked' to the current page.",
              "type": [
                "null",
                "string"
              ]
            },
            "url": {
              "description": "URL of the current page",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "request": {
          "description": "Request describes the HTTP request information in case the event was created as a result of an HTTP request.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "body": {
              "description": "Body only contais the request bod, not the query string information. It can either be a dictionary (for standard HTTP requests) or a raw request body.",
              "type": [
                "null",
                "string",
                "object"
              ]
            },
            "cookies": {
              "description": "Cookies used by the request, parsed as key-value objects.",
              "type": [
                "null",
                "object"
              ]
            },
            "env": {
              "description": "Env holds environment variable information passed to the monitored service.",
              "type": [
                "null",
                "object"
              ]
            },
            "headers": {
              "description": "Headers includes any HTTP headers sent by the requester. Cookies will be taken by headers if supplied.",
              "type": [
                "null",
                "object"
              ],
              "additionalProperties": false,
              "patternProperties": {
                "[.*]*$": {
                  "type": [
                    "null",
                    "array",
                    "string"
                  ],
                  "items": {
                    "type": "string"
                  }
                }
              }
            },
            "http_version": {
              "description": "HTTPVersion holds information about the used HTTP version.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "method": {
              "description": "Method holds information about the method of the HTTP request.",
              "type": "string",
              "maxLength": 1024
            },
            "socket": {
              "description": "Socket holds information related to the recorded request, such as whether or not data were encrypted and the remote address.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "encrypted": {
                  "description": "Encrypted indicates whether a request was sent as TLS/HTTPS request. DEPRECATED: this field will be removed in a future release.",
                  "type": [
                    "null",
                    "boolean"
                  ]
                },
                "remote_address": {
                  "description": "RemoteAddress holds the network address sending the request. It should be obtained through standard APIs and not be parsed from any headers like 'Forwarded'.",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              }
            },
            "url": {
              "description": "URL holds information sucha as the raw URL, scheme, host and path.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "full": {
                  "description": "Full, possibly agent-assembled URL of the request, e.g. https://example.com:443/search?q=elasticsearch#top.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "hash": {
                  "description": "Hash of the request URL, e.g. 'top'",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "hostname": {
                  "description": "Hostname information of the request, e.g. 'example.com'.\"",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "pathname": {
                  "description": "Path of the request, e.g. '/search'",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "port": {
                  "description": "Port of the request, e.g. '443'. Can be sent as string or int.",
                  "type": [
                    "null",
                    "string",
                    "integer"
                  ],
                  "maxLength": 1024
                },
                "protocol": {
                  "description": "Protocol information for the recorded request, e.g. 'https:'.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "raw": {
                  "description": "Raw unparsed URL of the HTTP request line, e.g https://example.com:443/search?q=elasticsearch. This URL may be absolute or relative. For more details, see https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.1.2.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "search": {
                  "description": "Search contains the query string information of the request. It is expected to have values delimited by ampersands.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            }
          },
          "required": [
            "method"
          ]
        },
        "response": {
          "description": "Response describes the HTTP response information in case the event was created as a result of an HTTP request.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "decoded_body_size": {
              "description": "DecodedBodySize holds the size of the decoded payload.",
              "type": [
                "null",
                "integer"
              ]
            },
            "encoded_body_size": {
              "description": "EncodedBodySize holds the size of the encoded payload.",
              "type": [
                "null",
                "integer"
              ]
            },
            "finished": {
              "description": "Finished indicates whether the response was finished or not.",
              "type": [
                "null",
                "boolean"
              ]
            },
            "headers": {
              "description": "Headers holds the http headers sent in the http response.",
              "type": [
                "null",
                "object"
              ],
              "additionalProperties": false,
              "patternProperties": {
                "[.*]*$": {
                  "type": [
                    "null",
                    "array",
                    "string"
                  ],
                  "items": {
                    "type": "string"
                  }
                }
              }
            },
            "headers_sent": {
              "description": "HeadersSent indicates whether http headers were sent.",
              "type": [
                "null",
                "boolean"
              ]
            },
            "status_code": {
              "description": "StatusCode sent in the http response.",
              "type": [
                "null",
                "integer"
              ]
            },
            "transfer_size": {
              "description": "TransferSize holds the total size of the payload.",
              "type": [
                "null",
                "integer"
              ]
            }
          }
        },
        "service": {
          "description": "Service related information can be sent per event. Information provided here will override the more generic information retrieved from metadata, missing service fields will be retrieved from the metadata information.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "agent": {
              "description": "Agent holds information about the APM agent capturing the event.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "ephemeral_id": {
                  "description": "EphemeralID is a free format ID used for metrics correlation by agents",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "name": {
                  "description": "Name of the APM agent capturing information.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the APM agent capturing information.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "environment": {
              "description": "Environment in which the monitored service is running, e.g. \`production\` or \`staging\`.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "framework": {
              "description": "Framework holds information about the framework used in the monitored service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the used framework",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the used framework",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "id": {
              "description": "ID holds a unique identifier for the service.",
              "type": [
                "null",
                "string"
              ]
            },
            "language": {
              "description": "Language holds information about the programming language of the monitored service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the used programming language",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the used programming language",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "name": {
              "description": "Name of the monitored service.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024,
              "pattern": "^[a-zA-Z0-9 _-]+$"
            },
            "node": {
              "description": "Node must be a unique meaningful name of the service node.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "configured_name": {
                  "description": "Name of the service node",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "origin": {
              "description": "Origin contains the self-nested field groups for service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "id": {
                  "description": "Immutable id of the service emitting this event.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "name": {
                  "description": "Immutable name of the service emitting this event.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "version": {
                  "description": "The version of the service the data was collected from.",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              }
            },
            "runtime": {
              "description": "Runtime holds information about the language runtime running the monitored service",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the language runtime",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the language runtime",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "target": {
              "description": "Target holds information about the outgoing service in case of an outgoing event",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Immutable name of the target service for the event",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "type": {
                  "description": "Immutable type of the target service for the event",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              },
              "anyOf": [
                {
                  "properties": {
                    "type": {
                      "type": "string"
                    }
                  },
                  "required": [
                    "type"
                  ]
                },
                {
                  "properties": {
                    "name": {
                      "type": "string"
                    }
                  },
                  "required": [
                    "name"
                  ]
                }
              ]
            },
            "version": {
              "description": "Version of the monitored service.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "tags": {
          "description": "Tags are a flat mapping of user-defined tags. On the agent side, tags are called labels. Allowed value types are string, boolean and number values. Tags are indexed and searchable.",
          "type": [
            "null",
            "object"
          ],
          "additionalProperties": {
            "type": [
              "null",
              "string",
              "boolean",
              "number"
            ],
            "maxLength": 1024
          }
        },
        "user": {
          "description": "User holds information about the correlated user for this event. If user data are provided here, all user related information from metadata is ignored, otherwise the metadata's user information will be stored with the event.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "domain": {
              "description": "Domain of the logged in user",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "email": {
              "description": "Email of the user.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "id": {
              "description": "ID identifies the logged in user, e.g. can be the primary key of the user",
              "type": [
                "null",
                "string",
                "integer"
              ],
              "maxLength": 1024
            },
            "username": {
              "description": "Name of the user.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        }
      }
    },
    "dropped_spans_stats": {
      "description": "DroppedSpanStats holds information about spans that were dropped (for example due to transaction_max_spans or exit_span_min_duration).",
      "type": [
        "null",
        "array"
      ],
      "items": {
        "type": "object",
        "properties": {
          "destination_service_resource": {
            "description": "DestinationServiceResource identifies the destination service resource being operated on. e.g. 'http://elastic.co:80', 'elasticsearch', 'rabbitmq/queue_name'.",
            "type": [
              "null",
              "string"
            ],
            "maxLength": 1024
          },
          "duration": {
            "description": "Duration holds duration aggregations about the dropped span.",
            "type": [
              "null",
              "object"
            ],
            "properties": {
              "count": {
                "description": "Count holds the number of times the dropped span happened.",
                "type": [
                  "null",
                  "integer"
                ],
                "minimum": 1
              },
              "sum": {
                "description": "Sum holds dimensions about the dropped span's duration.",
                "type": [
                  "null",
                  "object"
                ],
                "properties": {
                  "us": {
                    "description": "Us represents the summation of the span duration.",
                    "type": [
                      "null",
                      "integer"
                    ],
                    "minimum": 0
                  }
                }
              }
            }
          },
          "outcome": {
            "description": "Outcome of the span: success, failure, or unknown. Outcome may be one of a limited set of permitted values describing the success or failure of the span. It can be used for calculating error rates for outgoing requests.",
            "type": [
              "null",
              "string"
            ],
            "enum": [
              "success",
              "failure",
              "unknown",
              null
            ]
          },
          "service_target_name": {
            "description": "ServiceTargetName identifies the instance name of the target service being operated on",
            "type": [
              "null",
              "string"
            ],
            "maxLength": 512
          },
          "service_target_type": {
            "description": "ServiceTargetType identifies the type of the target service being operated on e.g. 'oracle', 'rabbitmq'",
            "type": [
              "null",
              "string"
            ],
            "maxLength": 512
          }
        }
      },
      "minItems": 0
    },
    "duration": {
      "description": "Duration how long the transaction took to complete, in milliseconds with 3 decimal points.",
      "type": "number",
      "minimum": 0
    },
    "experience": {
      "description": "UserExperience holds metrics for measuring real user experience. This information is only sent by RUM agents.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "cls": {
          "description": "CumulativeLayoutShift holds the Cumulative Layout Shift (CLS) metric value, or a negative value if CLS is unknown. See https://web.dev/cls/",
          "type": [
            "null",
            "number"
          ],
          "minimum": 0
        },
        "fid": {
          "description": "FirstInputDelay holds the First Input Delay (FID) metric value, or a negative value if FID is unknown. See https://web.dev/fid/",
          "type": [
            "null",
            "number"
          ],
          "minimum": 0
        },
        "longtask": {
          "description": "Longtask holds longtask duration/count metrics.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "count": {
              "description": "Count is the total number of of longtasks.",
              "type": "integer",
              "minimum": 0
            },
            "max": {
              "description": "Max longtask duration",
              "type": "number",
              "minimum": 0
            },
            "sum": {
              "description": "Sum of longtask durations",
              "type": "number",
              "minimum": 0
            }
          },
          "required": [
            "count",
            "max",
            "sum"
          ]
        },
        "tbt": {
          "description": "TotalBlockingTime holds the Total Blocking Time (TBT) metric value, or a negative value if TBT is unknown. See https://web.dev/tbt/",
          "type": [
            "null",
            "number"
          ],
          "minimum": 0
        }
      }
    },
    "faas": {
      "description": "FAAS holds fields related to Function as a Service events.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "coldstart": {
          "description": "Indicates whether a function invocation was a cold start or not.",
          "type": [
            "null",
            "boolean"
          ]
        },
        "execution": {
          "description": "The request id of the function invocation.",
          "type": [
            "null",
            "string"
          ]
        },
        "id": {
          "description": "A unique identifier of the invoked serverless function.",
          "type": [
            "null",
            "string"
          ]
        },
        "name": {
          "description": "The lambda function name.",
          "type": [
            "null",
            "string"
          ]
        },
        "trigger": {
          "description": "Trigger attributes.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "request_id": {
              "description": "The id of the origin trigger request.",
              "type": [
                "null",
                "string"
              ]
            },
            "type": {
              "description": "The trigger type.",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "version": {
          "description": "The lambda function version.",
          "type": [
            "null",
            "string"
          ]
        }
      }
    },
    "id": {
      "description": "ID holds the hex encoded 64 random bits ID of the event.",
      "type": "string",
      "maxLength": 1024
    },
    "links": {
      "description": "Links holds links to other spans, potentially in other traces.",
      "type": [
        "null",
        "array"
      ],
      "items": {
        "type": "object",
        "properties": {
          "span_id": {
            "description": "SpanID holds the ID of the linked span.",
            "type": "string",
            "maxLength": 1024
          },
          "trace_id": {
            "description": "TraceID holds the ID of the linked span's trace.",
            "type": "string",
            "maxLength": 1024
          }
        },
        "required": [
          "span_id",
          "trace_id"
        ]
      },
      "minItems": 0
    },
    "marks": {
      "description": "Marks capture the timing of a significant event during the lifetime of a transaction. Marks are organized into groups and can be set by the user or the agent. Marks are only reported by RUM agents.",
      "type": [
        "null",
        "object"
      ],
      "additionalProperties": {
        "type": [
          "null",
          "object"
        ],
        "additionalProperties": {
          "type": [
            "null",
            "number"
          ]
        }
      }
    },
    "name": {
      "description": "Name is the generic designation of a transaction in the scope of a single service, eg: 'GET /users/:id'.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "otel": {
      "description": "OTel contains unmapped OpenTelemetry attributes.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "attributes": {
          "description": "Attributes hold the unmapped OpenTelemetry attributes.",
          "type": [
            "null",
            "object"
          ]
        },
        "span_kind": {
          "description": "SpanKind holds the incoming OpenTelemetry span kind.",
          "type": [
            "null",
            "string"
          ]
        }
      }
    },
    "outcome": {
      "description": "Outcome of the transaction with a limited set of permitted values, describing the success or failure of the transaction from the service's perspective. It is used for calculating error rates for incoming requests. Permitted values: success, failure, unknown.",
      "type": [
        "null",
        "string"
      ],
      "enum": [
        "success",
        "failure",
        "unknown",
        null
      ]
    },
    "parent_id": {
      "description": "ParentID holds the hex encoded 64 random bits ID of the parent transaction or span.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "result": {
      "description": "Result of the transaction. For HTTP-related transactions, this should be the status code formatted like 'HTTP 2xx'.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "sample_rate": {
      "description": "SampleRate applied to the monitored service at the time where this transaction was recorded. Allowed values are [0..1]. A SampleRate \u003c1 indicates that not all spans are recorded.",
      "type": [
        "null",
        "number"
      ]
    },
    "sampled": {
      "description": "Sampled indicates whether or not the full information for a transaction is captured. If a transaction is unsampled no spans and less context information will be reported.",
      "type": [
        "null",
        "boolean"
      ]
    },
    "session": {
      "description": "Session holds optional transaction session information for RUM.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "id": {
          "description": "ID holds a session ID for grouping a set of related transactions.",
          "type": "string",
          "maxLength": 1024
        },
        "sequence": {
          "description": "Sequence holds an optional sequence number for a transaction within a session. It is not meaningful to compare sequences across two different sessions.",
          "type": [
            "null",
            "integer"
          ],
          "minimum": 1
        }
      },
      "required": [
        "id"
      ]
    },
    "span_count": {
      "description": "SpanCount counts correlated spans.",
      "type": "object",
      "properties": {
        "dropped": {
          "description": "Dropped is the number of correlated spans that have been dropped by the APM agent recording the transaction.",
          "type": [
            "null",
            "integer"
          ]
        },
        "started": {
          "description": "Started is the number of correlated spans that are recorded.",
          "type": "integer"
        }
      },
      "required": [
        "started"
      ]
    },
    "timestamp": {
      "description": "Timestamp holds the recorded time of the event, UTC based and formatted as microseconds since Unix epoch",
      "type": [
        "null",
        "integer"
      ]
    },
    "trace_id": {
      "description": "TraceID holds the hex encoded 128 random bits ID of the correlated trace.",
      "type": "string",
      "maxLength": 1024
    },
    "type": {
      "description": "Type expresses the transaction's type as keyword that has specific relevance within the service's domain, eg: 'request', 'backgroundjob'.",
      "type": "string",
      "maxLength": 1024
    }
  },
  "required": [
    "trace_id",
    "id",
    "type",
    "span_count",
    "duration"
  ]
}
```

::::

### Spans [observability-apm-server-api-spans]

Spans are events captured by an agent occurring in a monitored service.

#### Span Schema [api-span-schema]

The managed intake service uses JSON Schema to validate requests. The specification for spans is defined on [GitHub](https://github.com/elastic/apm-server/blob/main/docs/spec/v2/span.json) and included below.

::::{dropdown} Click to expand the schema
```json
{
  "$id": "docs/spec/v2/span",
  "type": "object",
  "properties": {
    "action": {
      "description": "Action holds the specific kind of event within the sub-type represented by the span (e.g. query, connect)",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "child_ids": {
      "description": "ChildIDs holds a list of successor transactions and/or spans.",
      "type": [
        "null",
        "array"
      ],
      "items": {
        "type": "string",
        "maxLength": 1024
      },
      "minItems": 0
    },
    "composite": {
      "description": "Composite holds details on a group of spans represented by a single one.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "compression_strategy": {
          "description": "A string value indicating which compression strategy was used. The valid values are \`exact_match\` and \`same_kind\`.",
          "type": "string"
        },
        "count": {
          "description": "Count is the number of compressed spans the composite span represents. The minimum count is 2, as a composite span represents at least two spans.",
          "type": "integer",
          "minimum": 2
        },
        "sum": {
          "description": "Sum is the durations of all compressed spans this composite span represents in milliseconds.",
          "type": "number",
          "minimum": 0
        }
      },
      "required": [
        "compression_strategy",
        "count",
        "sum"
      ]
    },
    "context": {
      "description": "Context holds arbitrary contextual information for the event.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "db": {
          "description": "Database contains contextual data for database spans",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "instance": {
              "description": "Instance name of the database.",
              "type": [
                "null",
                "string"
              ]
            },
            "link": {
              "description": "Link to the database server.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "rows_affected": {
              "description": "RowsAffected shows the number of rows affected by the statement.",
              "type": [
                "null",
                "integer"
              ]
            },
            "statement": {
              "description": "Statement of the recorded database event, e.g. query.",
              "type": [
                "null",
                "string"
              ]
            },
            "type": {
              "description": "Type of the recorded database event., e.g. sql, cassandra, hbase, redis.",
              "type": [
                "null",
                "string"
              ]
            },
            "user": {
              "description": "User is the username with which the database is accessed.",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "destination": {
          "description": "Destination contains contextual data about the destination of spans",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "address": {
              "description": "Address is the destination network address: hostname (e.g. 'localhost'), FQDN (e.g. 'elastic.co'), IPv4 (e.g. '127.0.0.1') IPv6 (e.g. '::1')",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "port": {
              "description": "Port is the destination network port (e.g. 443)",
              "type": [
                "null",
                "integer"
              ]
            },
            "service": {
              "description": "Service describes the destination service",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name is the identifier for the destination service, e.g. 'http://elastic.co', 'elasticsearch', 'rabbitmq' ( DEPRECATED: this field will be removed in a future release",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "resource": {
                  "description": "Resource identifies the destination service resource being operated on e.g. 'http://elastic.co:80', 'elasticsearch', 'rabbitmq/queue_name' DEPRECATED: this field will be removed in a future release",
                  "type": "string",
                  "maxLength": 1024
                },
                "type": {
                  "description": "Type of the destination service, e.g. db, elasticsearch. Should typically be the same as span.type. DEPRECATED: this field will be removed in a future release",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              },
              "required": [
                "resource"
              ]
            }
          }
        },
        "http": {
          "description": "HTTP contains contextual information when the span concerns an HTTP request.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "method": {
              "description": "Method holds information about the method of the HTTP request.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "request": {
              "description": "Request describes the HTTP request information.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "id": {
                  "description": "ID holds the unique identifier for the http request.",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              }
            },
            "response": {
              "description": "Response describes the HTTP response information in case the event was created as a result of an HTTP request.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "decoded_body_size": {
                  "description": "DecodedBodySize holds the size of the decoded payload.",
                  "type": [
                    "null",
                    "integer"
                  ]
                },
                "encoded_body_size": {
                  "description": "EncodedBodySize holds the size of the encoded payload.",
                  "type": [
                    "null",
                    "integer"
                  ]
                },
                "headers": {
                  "description": "Headers holds the http headers sent in the http response.",
                  "type": [
                    "null",
                    "object"
                  ],
                  "additionalProperties": false,
                  "patternProperties": {
                    "[.*]*$": {
                      "type": [
                        "null",
                        "array",
                        "string"
                      ],
                      "items": {
                        "type": "string"
                      }
                    }
                  }
                },
                "status_code": {
                  "description": "StatusCode sent in the http response.",
                  "type": [
                    "null",
                    "integer"
                  ]
                },
                "transfer_size": {
                  "description": "TransferSize holds the total size of the payload.",
                  "type": [
                    "null",
                    "integer"
                  ]
                }
              }
            },
            "status_code": {
              "description": "Deprecated: Use Response.StatusCode instead. StatusCode sent in the http response.",
              "type": [
                "null",
                "integer"
              ]
            },
            "url": {
              "description": "URL is the raw url of the correlating HTTP request.",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "message": {
          "description": "Message holds details related to message receiving and publishing if the captured event integrates with a messaging system",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "age": {
              "description": "Age of the message. If the monitored messaging framework provides a timestamp for the message, agents may use it. Otherwise, the sending agent can add a timestamp in milliseconds since the Unix epoch to the message's metadata to be retrieved by the receiving agent. If a timestamp is not available, agents should omit this field.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "ms": {
                  "description": "Age of the message in milliseconds.",
                  "type": [
                    "null",
                    "integer"
                  ]
                }
              }
            },
            "body": {
              "description": "Body of the received message, similar to an HTTP request body",
              "type": [
                "null",
                "string"
              ]
            },
            "headers": {
              "description": "Headers received with the message, similar to HTTP request headers.",
              "type": [
                "null",
                "object"
              ],
              "additionalProperties": false,
              "patternProperties": {
                "[.*]*$": {
                  "type": [
                    "null",
                    "array",
                    "string"
                  ],
                  "items": {
                    "type": "string"
                  }
                }
              }
            },
            "queue": {
              "description": "Queue holds information about the message queue where the message is received.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name holds the name of the message queue where the message is received.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "routing_key": {
              "description": "RoutingKey holds the optional routing key of the received message as set on the queuing system, such as in RabbitMQ.",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "service": {
          "description": "Service related information can be sent per span. Information provided here will override the more generic information retrieved from metadata, missing service fields will be retrieved from the metadata information.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "agent": {
              "description": "Agent holds information about the APM agent capturing the event.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "ephemeral_id": {
                  "description": "EphemeralID is a free format ID used for metrics correlation by agents",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "name": {
                  "description": "Name of the APM agent capturing information.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the APM agent capturing information.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "environment": {
              "description": "Environment in which the monitored service is running, e.g. \`production\` or \`staging\`.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "framework": {
              "description": "Framework holds information about the framework used in the monitored service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the used framework",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the used framework",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "id": {
              "description": "ID holds a unique identifier for the service.",
              "type": [
                "null",
                "string"
              ]
            },
            "language": {
              "description": "Language holds information about the programming language of the monitored service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the used programming language",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the used programming language",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "name": {
              "description": "Name of the monitored service.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024,
              "pattern": "^[a-zA-Z0-9 _-]+$"
            },
            "node": {
              "description": "Node must be a unique meaningful name of the service node.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "configured_name": {
                  "description": "Name of the service node",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "origin": {
              "description": "Origin contains the self-nested field groups for service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "id": {
                  "description": "Immutable id of the service emitting this event.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "name": {
                  "description": "Immutable name of the service emitting this event.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "version": {
                  "description": "The version of the service the data was collected from.",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              }
            },
            "runtime": {
              "description": "Runtime holds information about the language runtime running the monitored service",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the language runtime",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the language runtime",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "target": {
              "description": "Target holds information about the outgoing service in case of an outgoing event",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Immutable name of the target service for the event",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "type": {
                  "description": "Immutable type of the target service for the event",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              },
              "anyOf": [
                {
                  "properties": {
                    "type": {
                      "type": "string"
                    }
                  },
                  "required": [
                    "type"
                  ]
                },
                {
                  "properties": {
                    "name": {
                      "type": "string"
                    }
                  },
                  "required": [
                    "name"
                  ]
                }
              ]
            },
            "version": {
              "description": "Version of the monitored service.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "tags": {
          "description": "Tags are a flat mapping of user-defined tags. On the agent side, tags are called labels. Allowed value types are string, boolean and number values. Tags are indexed and searchable.",
          "type": [
            "null",
            "object"
          ],
          "additionalProperties": {
            "type": [
              "null",
              "string",
              "boolean",
              "number"
            ],
            "maxLength": 1024
          }
        }
      }
    },
    "duration": {
      "description": "Duration of the span in milliseconds. When the span is a composite one, duration is the gross duration, including \"whitespace\" in between spans.",
      "type": "number",
      "minimum": 0
    },
    "id": {
      "description": "ID holds the hex encoded 64 random bits ID of the event.",
      "type": "string",
      "maxLength": 1024
    },
    "links": {
      "description": "Links holds links to other spans, potentially in other traces.",
      "type": [
        "null",
        "array"
      ],
      "items": {
        "type": "object",
        "properties": {
          "span_id": {
            "description": "SpanID holds the ID of the linked span.",
            "type": "string",
            "maxLength": 1024
          },
          "trace_id": {
            "description": "TraceID holds the ID of the linked span's trace.",
            "type": "string",
            "maxLength": 1024
          }
        },
        "required": [
          "span_id",
          "trace_id"
        ]
      },
      "minItems": 0
    },
    "name": {
      "description": "Name is the generic designation of a span in the scope of a transaction.",
      "type": "string",
      "maxLength": 1024
    },
    "otel": {
      "description": "OTel contains unmapped OpenTelemetry attributes.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "attributes": {
          "description": "Attributes hold the unmapped OpenTelemetry attributes.",
          "type": [
            "null",
            "object"
          ]
        },
        "span_kind": {
          "description": "SpanKind holds the incoming OpenTelemetry span kind.",
          "type": [
            "null",
            "string"
          ]
        }
      }
    },
    "outcome": {
      "description": "Outcome of the span: success, failure, or unknown. Outcome may be one of a limited set of permitted values describing the success or failure of the span. It can be used for calculating error rates for outgoing requests.",
      "type": [
        "null",
        "string"
      ],
      "enum": [
        "success",
        "failure",
        "unknown",
        null
      ]
    },
    "parent_id": {
      "description": "ParentID holds the hex encoded 64 random bits ID of the parent transaction or span.",
      "type": "string",
      "maxLength": 1024
    },
    "sample_rate": {
      "description": "SampleRate applied to the monitored service at the time where this span was recorded.",
      "type": [
        "null",
        "number"
      ]
    },
    "stacktrace": {
      "description": "Stacktrace connected to this span event.",
      "type": [
        "null",
        "array"
      ],
      "items": {
        "type": "object",
        "properties": {
          "abs_path": {
            "description": "AbsPath is the absolute path of the frame's file.",
            "type": [
              "null",
              "string"
            ]
          },
          "classname": {
            "description": "Classname of the frame.",
            "type": [
              "null",
              "string"
            ]
          },
          "colno": {
            "description": "ColumnNumber of the frame.",
            "type": [
              "null",
              "integer"
            ]
          },
          "context_line": {
            "description": "ContextLine is the line from the frame's file.",
            "type": [
              "null",
              "string"
            ]
          },
          "filename": {
            "description": "Filename is the relative name of the frame's file.",
            "type": [
              "null",
              "string"
            ]
          },
          "function": {
            "description": "Function represented by the frame.",
            "type": [
              "null",
              "string"
            ]
          },
          "library_frame": {
            "description": "LibraryFrame indicates whether the frame is from a third party library.",
            "type": [
              "null",
              "boolean"
            ]
          },
          "lineno": {
            "description": "LineNumber of the frame.",
            "type": [
              "null",
              "integer"
            ]
          },
          "module": {
            "description": "Module to which the frame belongs to.",
            "type": [
              "null",
              "string"
            ]
          },
          "post_context": {
            "description": "PostContext is a slice of code lines immediately before the line from the frame's file.",
            "type": [
              "null",
              "array"
            ],
            "items": {
              "type": "string"
            },
            "minItems": 0
          },
          "pre_context": {
            "description": "PreContext is a slice of code lines immediately after the line from the frame's file.",
            "type": [
              "null",
              "array"
            ],
            "items": {
              "type": "string"
            },
            "minItems": 0
          },
          "vars": {
            "description": "Vars is a flat mapping of local variables of the frame.",
            "type": [
              "null",
              "object"
            ]
          }
        },
        "anyOf": [
          {
            "properties": {
              "classname": {
                "type": "string"
              }
            },
            "required": [
              "classname"
            ]
          },
          {
            "properties": {
              "filename": {
                "type": "string"
              }
            },
            "required": [
              "filename"
            ]
          }
        ]
      },
      "minItems": 0
    },
    "start": {
      "description": "Start is the offset relative to the transaction's timestamp identifying the start of the span, in milliseconds.",
      "type": [
        "null",
        "number"
      ]
    },
    "subtype": {
      "description": "Subtype is a further sub-division of the type (e.g. postgresql, elasticsearch)",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "sync": {
      "description": "Sync indicates whether the span was executed synchronously or asynchronously.",
      "type": [
        "null",
        "boolean"
      ]
    },
    "timestamp": {
      "description": "Timestamp holds the recorded time of the event, UTC based and formatted as microseconds since Unix epoch",
      "type": [
        "null",
        "integer"
      ]
    },
    "trace_id": {
      "description": "TraceID holds the hex encoded 128 random bits ID of the correlated trace.",
      "type": "string",
      "maxLength": 1024
    },
    "transaction_id": {
      "description": "TransactionID holds the hex encoded 64 random bits ID of the correlated transaction.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "type": {
      "description": "Type holds the span's type, and can have specific keywords within the service's domain (eg: 'request', 'backgroundjob', etc)",
      "type": "string",
      "maxLength": 1024
    }
  },
  "required": [
    "id",
    "trace_id",
    "name",
    "parent_id",
    "type",
    "duration"
  ],
  "anyOf": [
    {
      "properties": {
        "start": {
          "type": "number"
        }
      },
      "required": [
        "start"
      ]
    },
    {
      "properties": {
        "timestamp": {
          "type": "integer"
        }
      },
      "required": [
        "timestamp"
      ]
    }
  ]
}
```

::::

### Errors [observability-apm-server-api-errors]

An error or a logged error message captured by an agent occurring in a monitored service.

#### Error Schema [api-error-schema]

The managed intake service uses a JSON Schema to validate requests. The specification for errors is defined on [GitHub](https://github.com/elastic/apm-server/blob/main/docs/spec/v2/error.json) and included below.

::::{dropdown} Click to expand the schema
```json
{
  "$id": "docs/spec/v2/error",
  "description": "errorEvent represents an error or a logged error message, captured by an APM agent in a monitored service.",
  "type": "object",
  "properties": {
    "context": {
      "description": "Context holds arbitrary contextual information for the event.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "cloud": {
          "description": "Cloud holds fields related to the cloud or infrastructure the events are coming from.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "origin": {
              "description": "Origin contains the self-nested field groups for cloud.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "account": {
                  "description": "The cloud account or organization id used to identify different entities in a multi-tenant environment.",
                  "type": [
                    "null",
                    "object"
                  ],
                  "properties": {
                    "id": {
                      "description": "The cloud account or organization id used to identify different entities in a multi-tenant environment.",
                      "type": [
                        "null",
                        "string"
                      ]
                    }
                  }
                },
                "provider": {
                  "description": "Name of the cloud provider.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "region": {
                  "description": "Region in which this host, resource, or service is located.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "service": {
                  "description": "The cloud service name is intended to distinguish services running on different platforms within a provider.",
                  "type": [
                    "null",
                    "object"
                  ],
                  "properties": {
                    "name": {
                      "description": "The cloud service name is intended to distinguish services running on different platforms within a provider.",
                      "type": [
                        "null",
                        "string"
                      ]
                    }
                  }
                }
              }
            }
          }
        },
        "custom": {
          "description": "Custom can contain additional metadata to be stored with the event. The format is unspecified and can be deeply nested objects. The information will not be indexed or searchable in Elasticsearch.",
          "type": [
            "null",
            "object"
          ]
        },
        "message": {
          "description": "Message holds details related to message receiving and publishing if the captured event integrates with a messaging system",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "age": {
              "description": "Age of the message. If the monitored messaging framework provides a timestamp for the message, agents may use it. Otherwise, the sending agent can add a timestamp in milliseconds since the Unix epoch to the message's metadata to be retrieved by the receiving agent. If a timestamp is not available, agents should omit this field.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "ms": {
                  "description": "Age of the message in milliseconds.",
                  "type": [
                    "null",
                    "integer"
                  ]
                }
              }
            },
            "body": {
              "description": "Body of the received message, similar to an HTTP request body",
              "type": [
                "null",
                "string"
              ]
            },
            "headers": {
              "description": "Headers received with the message, similar to HTTP request headers.",
              "type": [
                "null",
                "object"
              ],
              "additionalProperties": false,
              "patternProperties": {
                "[.*]*$": {
                  "type": [
                    "null",
                    "array",
                    "string"
                  ],
                  "items": {
                    "type": "string"
                  }
                }
              }
            },
            "queue": {
              "description": "Queue holds information about the message queue where the message is received.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name holds the name of the message queue where the message is received.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "routing_key": {
              "description": "RoutingKey holds the optional routing key of the received message as set on the queuing system, such as in RabbitMQ.",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "page": {
          "description": "Page holds information related to the current page and page referers. It is only sent from RUM agents.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "referer": {
              "description": "Referer holds the URL of the page that 'linked' to the current page.",
              "type": [
                "null",
                "string"
              ]
            },
            "url": {
              "description": "URL of the current page",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "request": {
          "description": "Request describes the HTTP request information in case the event was created as a result of an HTTP request.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "body": {
              "description": "Body only contais the request bod, not the query string information. It can either be a dictionary (for standard HTTP requests) or a raw request body.",
              "type": [
                "null",
                "string",
                "object"
              ]
            },
            "cookies": {
              "description": "Cookies used by the request, parsed as key-value objects.",
              "type": [
                "null",
                "object"
              ]
            },
            "env": {
              "description": "Env holds environment variable information passed to the monitored service.",
              "type": [
                "null",
                "object"
              ]
            },
            "headers": {
              "description": "Headers includes any HTTP headers sent by the requester. Cookies will be taken by headers if supplied.",
              "type": [
                "null",
                "object"
              ],
              "additionalProperties": false,
              "patternProperties": {
                "[.*]*$": {
                  "type": [
                    "null",
                    "array",
                    "string"
                  ],
                  "items": {
                    "type": "string"
                  }
                }
              }
            },
            "http_version": {
              "description": "HTTPVersion holds information about the used HTTP version.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "method": {
              "description": "Method holds information about the method of the HTTP request.",
              "type": "string",
              "maxLength": 1024
            },
            "socket": {
              "description": "Socket holds information related to the recorded request, such as whether or not data were encrypted and the remote address.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "encrypted": {
                  "description": "Encrypted indicates whether a request was sent as TLS/HTTPS request. DEPRECATED: this field will be removed in a future release.",
                  "type": [
                    "null",
                    "boolean"
                  ]
                },
                "remote_address": {
                  "description": "RemoteAddress holds the network address sending the request. It should be obtained through standard APIs and not be parsed from any headers like 'Forwarded'.",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              }
            },
            "url": {
              "description": "URL holds information sucha as the raw URL, scheme, host and path.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "full": {
                  "description": "Full, possibly agent-assembled URL of the request, e.g. https://example.com:443/search?q=elasticsearch#top.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "hash": {
                  "description": "Hash of the request URL, e.g. 'top'",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "hostname": {
                  "description": "Hostname information of the request, e.g. 'example.com'.\"",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "pathname": {
                  "description": "Path of the request, e.g. '/search'",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "port": {
                  "description": "Port of the request, e.g. '443'. Can be sent as string or int.",
                  "type": [
                    "null",
                    "string",
                    "integer"
                  ],
                  "maxLength": 1024
                },
                "protocol": {
                  "description": "Protocol information for the recorded request, e.g. 'https:'.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "raw": {
                  "description": "Raw unparsed URL of the HTTP request line, e.g https://example.com:443/search?q=elasticsearch. This URL may be absolute or relative. For more details, see https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.1.2.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "search": {
                  "description": "Search contains the query string information of the request. It is expected to have values delimited by ampersands.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            }
          },
          "required": [
            "method"
          ]
        },
        "response": {
          "description": "Response describes the HTTP response information in case the event was created as a result of an HTTP request.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "decoded_body_size": {
              "description": "DecodedBodySize holds the size of the decoded payload.",
              "type": [
                "null",
                "integer"
              ]
            },
            "encoded_body_size": {
              "description": "EncodedBodySize holds the size of the encoded payload.",
              "type": [
                "null",
                "integer"
              ]
            },
            "finished": {
              "description": "Finished indicates whether the response was finished or not.",
              "type": [
                "null",
                "boolean"
              ]
            },
            "headers": {
              "description": "Headers holds the http headers sent in the http response.",
              "type": [
                "null",
                "object"
              ],
              "additionalProperties": false,
              "patternProperties": {
                "[.*]*$": {
                  "type": [
                    "null",
                    "array",
                    "string"
                  ],
                  "items": {
                    "type": "string"
                  }
                }
              }
            },
            "headers_sent": {
              "description": "HeadersSent indicates whether http headers were sent.",
              "type": [
                "null",
                "boolean"
              ]
            },
            "status_code": {
              "description": "StatusCode sent in the http response.",
              "type": [
                "null",
                "integer"
              ]
            },
            "transfer_size": {
              "description": "TransferSize holds the total size of the payload.",
              "type": [
                "null",
                "integer"
              ]
            }
          }
        },
        "service": {
          "description": "Service related information can be sent per event. Information provided here will override the more generic information retrieved from metadata, missing service fields will be retrieved from the metadata information.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "agent": {
              "description": "Agent holds information about the APM agent capturing the event.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "ephemeral_id": {
                  "description": "EphemeralID is a free format ID used for metrics correlation by agents",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "name": {
                  "description": "Name of the APM agent capturing information.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the APM agent capturing information.",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "environment": {
              "description": "Environment in which the monitored service is running, e.g. \`production\` or \`staging\`.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "framework": {
              "description": "Framework holds information about the framework used in the monitored service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the used framework",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the used framework",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "id": {
              "description": "ID holds a unique identifier for the service.",
              "type": [
                "null",
                "string"
              ]
            },
            "language": {
              "description": "Language holds information about the programming language of the monitored service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the used programming language",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the used programming language",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "name": {
              "description": "Name of the monitored service.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024,
              "pattern": "^[a-zA-Z0-9 _-]+$"
            },
            "node": {
              "description": "Node must be a unique meaningful name of the service node.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "configured_name": {
                  "description": "Name of the service node",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "origin": {
              "description": "Origin contains the self-nested field groups for service.",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "id": {
                  "description": "Immutable id of the service emitting this event.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "name": {
                  "description": "Immutable name of the service emitting this event.",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "version": {
                  "description": "The version of the service the data was collected from.",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              }
            },
            "runtime": {
              "description": "Runtime holds information about the language runtime running the monitored service",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Name of the language runtime",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                },
                "version": {
                  "description": "Version of the language runtime",
                  "type": [
                    "null",
                    "string"
                  ],
                  "maxLength": 1024
                }
              }
            },
            "target": {
              "description": "Target holds information about the outgoing service in case of an outgoing event",
              "type": [
                "null",
                "object"
              ],
              "properties": {
                "name": {
                  "description": "Immutable name of the target service for the event",
                  "type": [
                    "null",
                    "string"
                  ]
                },
                "type": {
                  "description": "Immutable type of the target service for the event",
                  "type": [
                    "null",
                    "string"
                  ]
                }
              },
              "anyOf": [
                {
                  "properties": {
                    "type": {
                      "type": "string"
                    }
                  },
                  "required": [
                    "type"
                  ]
                },
                {
                  "properties": {
                    "name": {
                      "type": "string"
                    }
                  },
                  "required": [
                    "name"
                  ]
                }
              ]
            },
            "version": {
              "description": "Version of the monitored service.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        },
        "tags": {
          "description": "Tags are a flat mapping of user-defined tags. On the agent side, tags are called labels. Allowed value types are string, boolean and number values. Tags are indexed and searchable.",
          "type": [
            "null",
            "object"
          ],
          "additionalProperties": {
            "type": [
              "null",
              "string",
              "boolean",
              "number"
            ],
            "maxLength": 1024
          }
        },
        "user": {
          "description": "User holds information about the correlated user for this event. If user data are provided here, all user related information from metadata is ignored, otherwise the metadata's user information will be stored with the event.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "domain": {
              "description": "Domain of the logged in user",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "email": {
              "description": "Email of the user.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            },
            "id": {
              "description": "ID identifies the logged in user, e.g. can be the primary key of the user",
              "type": [
                "null",
                "string",
                "integer"
              ],
              "maxLength": 1024
            },
            "username": {
              "description": "Name of the user.",
              "type": [
                "null",
                "string"
              ],
              "maxLength": 1024
            }
          }
        }
      }
    },
    "culprit": {
      "description": "Culprit identifies the function call which was the primary perpetrator of this event.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "exception": {
      "description": "Exception holds information about the original error. The information is language specific.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "attributes": {
          "description": "Attributes of the exception.",
          "type": [
            "null",
            "object"
          ]
        },
        "cause": {
          "description": "Cause can hold a collection of error exceptions representing chained exceptions. The chain starts with the outermost exception, followed by its cause, and so on.",
          "type": [
            "null",
            "array"
          ],
          "items": {
            "type": "object"
          },
          "minItems": 0
        },
        "code": {
          "description": "Code that is set when the error happened, e.g. database error code.",
          "type": [
            "null",
            "string",
            "integer"
          ],
          "maxLength": 1024
        },
        "handled": {
          "description": "Handled indicates whether the error was caught in the code or not.",
          "type": [
            "null",
            "boolean"
          ]
        },
        "message": {
          "description": "Message contains the originally captured error message.",
          "type": [
            "null",
            "string"
          ]
        },
        "module": {
          "description": "Module describes the exception type's module namespace.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "stacktrace": {
          "description": "Stacktrace information of the captured exception.",
          "type": [
            "null",
            "array"
          ],
          "items": {
            "type": "object",
            "properties": {
              "abs_path": {
                "description": "AbsPath is the absolute path of the frame's file.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "classname": {
                "description": "Classname of the frame.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "colno": {
                "description": "ColumnNumber of the frame.",
                "type": [
                  "null",
                  "integer"
                ]
              },
              "context_line": {
                "description": "ContextLine is the line from the frame's file.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "filename": {
                "description": "Filename is the relative name of the frame's file.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "function": {
                "description": "Function represented by the frame.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "library_frame": {
                "description": "LibraryFrame indicates whether the frame is from a third party library.",
                "type": [
                  "null",
                  "boolean"
                ]
              },
              "lineno": {
                "description": "LineNumber of the frame.",
                "type": [
                  "null",
                  "integer"
                ]
              },
              "module": {
                "description": "Module to which the frame belongs to.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "post_context": {
                "description": "PostContext is a slice of code lines immediately before the line from the frame's file.",
                "type": [
                  "null",
                  "array"
                ],
                "items": {
                  "type": "string"
                },
                "minItems": 0
              },
              "pre_context": {
                "description": "PreContext is a slice of code lines immediately after the line from the frame's file.",
                "type": [
                  "null",
                  "array"
                ],
                "items": {
                  "type": "string"
                },
                "minItems": 0
              },
              "vars": {
                "description": "Vars is a flat mapping of local variables of the frame.",
                "type": [
                  "null",
                  "object"
                ]
              }
            },
            "anyOf": [
              {
                "properties": {
                  "classname": {
                    "type": "string"
                  }
                },
                "required": [
                  "classname"
                ]
              },
              {
                "properties": {
                  "filename": {
                    "type": "string"
                  }
                },
                "required": [
                  "filename"
                ]
              }
            ]
          },
          "minItems": 0
        },
        "type": {
          "description": "Type of the exception.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      },
      "anyOf": [
        {
          "properties": {
            "message": {
              "type": "string"
            }
          },
          "required": [
            "message"
          ]
        },
        {
          "properties": {
            "type": {
              "type": "string"
            }
          },
          "required": [
            "type"
          ]
        }
      ]
    },
    "id": {
      "description": "ID holds the hex encoded 128 random bits ID of the event.",
      "type": "string",
      "maxLength": 1024
    },
    "log": {
      "description": "Log holds additional information added when the error is logged.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "level": {
          "description": "Level represents the severity of the recorded log.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "logger_name": {
          "description": "LoggerName holds the name of the used logger instance.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "message": {
          "description": "Message of the logged error. In case a parameterized message is captured, Message should contain the same information, but with any placeholders being replaced.",
          "type": "string"
        },
        "param_message": {
          "description": "ParamMessage should contain the same information as Message, but with placeholders where parameters were logged, e.g. 'error connecting to %s'. The string is not interpreted, allowing differnt placeholders per client languange. The information might be used to group errors together.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "stacktrace": {
          "description": "Stacktrace information of the captured error.",
          "type": [
            "null",
            "array"
          ],
          "items": {
            "type": "object",
            "properties": {
              "abs_path": {
                "description": "AbsPath is the absolute path of the frame's file.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "classname": {
                "description": "Classname of the frame.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "colno": {
                "description": "ColumnNumber of the frame.",
                "type": [
                  "null",
                  "integer"
                ]
              },
              "context_line": {
                "description": "ContextLine is the line from the frame's file.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "filename": {
                "description": "Filename is the relative name of the frame's file.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "function": {
                "description": "Function represented by the frame.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "library_frame": {
                "description": "LibraryFrame indicates whether the frame is from a third party library.",
                "type": [
                  "null",
                  "boolean"
                ]
              },
              "lineno": {
                "description": "LineNumber of the frame.",
                "type": [
                  "null",
                  "integer"
                ]
              },
              "module": {
                "description": "Module to which the frame belongs to.",
                "type": [
                  "null",
                  "string"
                ]
              },
              "post_context": {
                "description": "PostContext is a slice of code lines immediately before the line from the frame's file.",
                "type": [
                  "null",
                  "array"
                ],
                "items": {
                  "type": "string"
                },
                "minItems": 0
              },
              "pre_context": {
                "description": "PreContext is a slice of code lines immediately after the line from the frame's file.",
                "type": [
                  "null",
                  "array"
                ],
                "items": {
                  "type": "string"
                },
                "minItems": 0
              },
              "vars": {
                "description": "Vars is a flat mapping of local variables of the frame.",
                "type": [
                  "null",
                  "object"
                ]
              }
            },
            "anyOf": [
              {
                "properties": {
                  "classname": {
                    "type": "string"
                  }
                },
                "required": [
                  "classname"
                ]
              },
              {
                "properties": {
                  "filename": {
                    "type": "string"
                  }
                },
                "required": [
                  "filename"
                ]
              }
            ]
          },
          "minItems": 0
        }
      },
      "required": [
        "message"
      ]
    },
    "parent_id": {
      "description": "ParentID holds the hex encoded 64 random bits ID of the parent transaction or span.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "timestamp": {
      "description": "Timestamp holds the recorded time of the event, UTC based and formatted as microseconds since Unix epoch.",
      "type": [
        "null",
        "integer"
      ]
    },
    "trace_id": {
      "description": "TraceID holds the hex encoded 128 random bits ID of the correlated trace.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    },
    "transaction": {
      "description": "Transaction holds information about the correlated transaction.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "name": {
          "description": "Name is the generic designation of a transaction in the scope of a single service, eg: 'GET /users/:id'.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "sampled": {
          "description": "Sampled indicates whether or not the full information for a transaction is captured. If a transaction is unsampled no spans and less context information will be reported.",
          "type": [
            "null",
            "boolean"
          ]
        },
        "type": {
          "description": "Type expresses the correlated transaction's type as keyword that has specific relevance within the service's domain, eg: 'request', 'backgroundjob'.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      }
    },
    "transaction_id": {
      "description": "TransactionID holds the hex encoded 64 random bits ID of the correlated transaction.",
      "type": [
        "null",
        "string"
      ],
      "maxLength": 1024
    }
  },
  "required": [
    "id"
  ],
  "allOf": [
    {
      "if": {
        "properties": {
          "transaction_id": {
            "type": "string"
          }
        },
        "required": [
          "transaction_id"
        ]
      },
      "then": {
        "properties": {
          "parent_id": {
            "type": "string"
          }
        },
        "required": [
          "parent_id"
        ]
      }
    },
    {
      "if": {
        "properties": {
          "trace_id": {
            "type": "string"
          }
        },
        "required": [
          "trace_id"
        ]
      },
      "then": {
        "properties": {
          "parent_id": {
            "type": "string"
          }
        },
        "required": [
          "parent_id"
        ]
      }
    },
    {
      "if": {
        "properties": {
          "transaction_id": {
            "type": "string"
          }
        },
        "required": [
          "transaction_id"
        ]
      },
      "then": {
        "properties": {
          "trace_id": {
            "type": "string"
          }
        },
        "required": [
          "trace_id"
        ]
      }
    },
    {
      "if": {
        "properties": {
          "parent_id": {
            "type": "string"
          }
        },
        "required": [
          "parent_id"
        ]
      },
      "then": {
        "properties": {
          "trace_id": {
            "type": "string"
          }
        },
        "required": [
          "trace_id"
        ]
      }
    }
  ],
  "anyOf": [
    {
      "properties": {
        "exception": {
          "type": "object"
        }
      },
      "required": [
        "exception"
      ]
    },
    {
      "properties": {
        "log": {
          "type": "object"
        }
      },
      "required": [
        "log"
      ]
    }
  ]
}
```

::::

### Metrics [observability-apm-server-api-metrics]

Metrics contain application metric data captured by an {{apm-agent}}.

#### Metric Schema [api-metricset-schema]

The managed intake service uses JSON Schema to validate requests. The specification for metrics is defined on [GitHub](https://github.com/elastic/apm-server/blob/main/docs/spec/v2/metricset.json) and included below.

::::{dropdown} Click to expand the schema
```json
{
  "$id": "docs/spec/v2/metricset",
  "type": "object",
  "properties": {
    "faas": {
      "description": "FAAS holds fields related to Function as a Service events.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "coldstart": {
          "description": "Indicates whether a function invocation was a cold start or not.",
          "type": [
            "null",
            "boolean"
          ]
        },
        "execution": {
          "description": "The request id of the function invocation.",
          "type": [
            "null",
            "string"
          ]
        },
        "id": {
          "description": "A unique identifier of the invoked serverless function.",
          "type": [
            "null",
            "string"
          ]
        },
        "name": {
          "description": "The lambda function name.",
          "type": [
            "null",
            "string"
          ]
        },
        "trigger": {
          "description": "Trigger attributes.",
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "request_id": {
              "description": "The id of the origin trigger request.",
              "type": [
                "null",
                "string"
              ]
            },
            "type": {
              "description": "The trigger type.",
              "type": [
                "null",
                "string"
              ]
            }
          }
        },
        "version": {
          "description": "The lambda function version.",
          "type": [
            "null",
            "string"
          ]
        }
      }
    },
    "samples": {
      "description": "Samples hold application metrics collected from the agent.",
      "type": "object",
      "additionalProperties": false,
      "patternProperties": {
        "^[^*\"]*$": {
          "type": [
            "null",
            "object"
          ],
          "properties": {
            "counts": {
              "description": "Counts holds the bucket counts for histogram metrics.  These numbers must be positive or zero.  If Counts is specified, then Values is expected to be specified with the same number of elements, and with the same order.",
              "type": [
                "null",
                "array"
              ],
              "items": {
                "type": "integer",
                "minimum": 0
              },
              "minItems": 0
            },
            "type": {
              "description": "Type holds an optional metric type: gauge, counter, or histogram.  If Type is unknown, it will be ignored.",
              "type": [
                "null",
                "string"
              ]
            },
            "unit": {
              "description": "Unit holds an optional unit for the metric.  - \"percent\" (value is in the range [0,1]) - \"byte\" - a time unit: \"nanos\", \"micros\", \"ms\", \"s\", \"m\", \"h\", \"d\"  If Unit is unknown, it will be ignored.",
              "type": [
                "null",
                "string"
              ]
            },
            "value": {
              "description": "Value holds the value of a single metric sample.",
              "type": [
                "null",
                "number"
              ]
            },
            "values": {
              "description": "Values holds the bucket values for histogram metrics.  Values must be provided in ascending order; failure to do so will result in the metric being discarded.",
              "type": [
                "null",
                "array"
              ],
              "items": {
                "type": "number"
              },
              "minItems": 0
            }
          },
          "allOf": [
            {
              "if": {
                "properties": {
                  "counts": {
                    "type": "array"
                  }
                },
                "required": [
                  "counts"
                ]
              },
              "then": {
                "properties": {
                  "values": {
                    "type": "array"
                  }
                },
                "required": [
                  "values"
                ]
              }
            },
            {
              "if": {
                "properties": {
                  "values": {
                    "type": "array"
                  }
                },
                "required": [
                  "values"
                ]
              },
              "then": {
                "properties": {
                  "counts": {
                    "type": "array"
                  }
                },
                "required": [
                  "counts"
                ]
              }
            }
          ],
          "anyOf": [
            {
              "properties": {
                "value": {
                  "type": "number"
                }
              },
              "required": [
                "value"
              ]
            },
            {
              "properties": {
                "values": {
                  "type": "array"
                }
              },
              "required": [
                "values"
              ]
            }
          ]
        }
      }
    },
    "service": {
      "description": "Service holds selected information about the correlated service.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "name": {
          "description": "Name of the correlated service.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "version": {
          "description": "Version of the correlated service.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      }
    },
    "span": {
      "description": "Span holds selected information about the correlated transaction.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "subtype": {
          "description": "Subtype is a further sub-division of the type (e.g. postgresql, elasticsearch)",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "type": {
          "description": "Type expresses the correlated span's type as keyword that has specific relevance within the service's domain, eg: 'request', 'backgroundjob'.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      }
    },
    "tags": {
      "description": "Tags are a flat mapping of user-defined tags. On the agent side, tags are called labels. Allowed value types are string, boolean and number values. Tags are indexed and searchable.",
      "type": [
        "null",
        "object"
      ],
      "additionalProperties": {
        "type": [
          "null",
          "string",
          "boolean",
          "number"
        ],
        "maxLength": 1024
      }
    },
    "timestamp": {
      "description": "Timestamp holds the recorded time of the event, UTC based and formatted as microseconds since Unix epoch",
      "type": [
        "null",
        "integer"
      ]
    },
    "transaction": {
      "description": "Transaction holds selected information about the correlated transaction.",
      "type": [
        "null",
        "object"
      ],
      "properties": {
        "name": {
          "description": "Name of the correlated transaction.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        },
        "type": {
          "description": "Type expresses the correlated transaction's type as keyword that has specific relevance within the service's domain, eg: 'request', 'backgroundjob'.",
          "type": [
            "null",
            "string"
          ],
          "maxLength": 1024
        }
      }
    }
  },
  "required": [
    "samples"
  ]
}
```

::::

## OpenTelemetry API [observability-apm-server-api-opentelemetry-api]

Elastic supports receiving traces, metrics, and logs over the [OpenTelemetry Protocol (OTLP)](https://opentelemetry.io/docs/specs/otlp/). OTLP is the default transfer protocol for OpenTelemetry and is supported natively by the managed intake service.

The managed intake service supports two OTLP communication protocols on the same port:

* OTLP/HTTP (protobuf)
* OTLP/gRPC

### OTLP/gRPC paths [otlpgrpc-paths]

| Name | Endpoint |
| --- | --- |
| OTLP metrics intake | `/opentelemetry.proto.collector.metrics.v1.MetricsService/Export` |
| OTLP trace intake | `/opentelemetry.proto.collector.trace.v1.TraceService/Export` |
| OTLP logs intake | `/opentelemetry.proto.collector.logs.v1.LogsService/Export` |

### OTLP/HTTP paths [otlphttp-paths]

| Name | Endpoint |
| --- | --- |
| OTLP metrics intake | `/v1/metrics` |
| OTLP trace intake | `/v1/traces` |
| OTLP logs intake | `/v1/logs` |

::::{tip}
See our [OpenTelemetry docs](/solutions/observability/apm/upstream-opentelemetry-collectors-language-sdks.md) to learn how to send data to the managed intake service from an OpenTelemetry agent OpenTelemetry collector.

::::

