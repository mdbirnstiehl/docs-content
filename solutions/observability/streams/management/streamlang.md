---
applies_to:
  serverless: ga
  stack: ga 9.2+
products:
  - id: observability
  - id: elasticsearch
  - id: kibana
  - id: cloud-serverless
  - id: cloud-hosted
  - id: cloud-enterprise
  - id: cloud-kubernetes
  - id: elastic-stack
---
# Streamlang reference [streams-streamlang-reference]

Streamlang is a YAML-based DSL for defining stream processing and routing logic. You can write Streamlang directly using the [YAML editing mode](./extract.md#streams-editing-yaml-mode) in the **Processing** tab, or use the [interactive mode](./extract.md#streams-editing-interactive-mode) which generates Streamlang behind the scenes.

Streamlang provides a consistent processing interface that can be transpiled to multiple execution targets, including {{es}} ingest pipelines and ES|QL. This allows processing to run at ingest time or query time without rewriting rules.

## Structure [streams-streamlang-structure]

A Streamlang configuration is a YAML document with a single top-level `steps` array. Each step is either a [processor](#streams-streamlang-processors) (an `action` block) or a [condition block](#streams-streamlang-condition-blocks) (a `condition` block with nested steps):

```yaml
steps:
  - action: <processor_type>
    # processor-specific parameters
  - action: <processor_type>
    # processor-specific parameters
    where:
      # optional condition
  - condition:
      field: <field_path>
      eq: <value>
      steps:
        - action: <processor_type>
          # nested processor
```

Steps run in order. Each processor transforms the document, and the result is passed to the next step.

## Processors [streams-streamlang-processors]

Processors are the building blocks of a Streamlang configuration. Each processor has an `action` field that specifies the type of operation to perform.

All processors support the following common options:

| Option | Type | Description |
| --- | --- | --- |
| `description` | string | A human-readable description of the processor. |
| `ignore_failure` | boolean | When `true`, document processing continues even if this processor fails. |
| `where` | [condition](#streams-streamlang-conditions) | A condition that must be met for the processor to run. |

### Grok [streams-streamlang-grok]

Parses unstructured text using predefined or custom [grok patterns](./extract/grok.md#streams-grok-example). Patterns are tried in order until one matches.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field to parse. |
| `patterns` | string[] | Yes | One or more grok patterns, tried in order. |
| `pattern_definitions` | object | No | Custom pattern definitions as key-value pairs. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

```yaml
- action: grok
  from: body.message
  patterns:
    - "%{IP:attributes.client_ip} %{WORD:attributes.method} %{URIPATHPARAM:attributes.path}"
  pattern_definitions:
    MY_PATTERN: "%{YEAR}-%{MONTHNUM}-%{MONTHDAY}"
```

### Dissect [streams-streamlang-dissect]

Parses structured text using delimiter-based [dissect patterns](./extract/dissect.md#streams-dissect-example). Faster than grok for consistently formatted logs.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field to parse. |
| `pattern` | string | Yes | Dissect pattern with `%{field}` placeholders. |
| `append_separator` | string | No | Separator used when concatenating target fields. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

```yaml
- action: dissect
  from: body.message
  pattern: "%{attributes.timestamp} %{attributes.level} %{attributes.message}"
```

### Date [streams-streamlang-date]

Parses date strings into timestamps. Refer to [Date processor](./extract/date.md) for supported format strings.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field containing the date string. |
| `formats` | string[] | Yes | Date formats to try, in order (for example, `ISO8601`, `UNIX`, or a Java time pattern). |
| `to` | string | No | Target field for the parsed date. Defaults to `@timestamp`. |
| `output_format` | string | No | Format for the output date string. Must be a valid Java time pattern. |
| `timezone` | string | No | Timezone to use when parsing. Defaults to `UTC`. |
| `locale` | string | No | Locale to use when parsing month names or weekdays. |

```yaml
- action: date
  from: attributes.timestamp
  formats:
    - "yyyy-MM-dd'T'HH:mm:ss.SSSZ"
    - "yyyy-MM-dd HH:mm:ss"
  to: attributes.parsed_time
  output_format: "yyyy-MM-dd"
  timezone: "America/New_York"
```

### Set [streams-streamlang-set]

Assigns a value to a field. Creates the field if it doesn't exist or overwrites its current value.

Specify exactly one of `value` or `copy_from`.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `to` | string | Yes | Target field. |
| `value` | any | One of `value` or `copy_from` | A literal value to assign. |
| `copy_from` | string | One of `value` or `copy_from` | A source field to copy the value from. |
| `override` | boolean | No | When `false`, the target field is only set if it doesn't already exist. |

```yaml
- action: set
  to: attributes.environment
  value: production

- action: set
  to: attributes.backup_message
  copy_from: body.message
```

### Rename [streams-streamlang-rename]

Moves a field's value to a new field name and removes the original.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field to rename. |
| `to` | string | Yes | New field name. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |
| `override` | boolean | No | When `true`, allow overwriting an existing target field. |

```yaml
- action: rename
  from: attributes.old_name
  to: attributes.new_name
```

### Remove [streams-streamlang-remove]

Removes a field from the document.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Field to remove. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the field is missing. |

```yaml
- action: remove
  from: attributes.temp_field
```

### Remove by prefix [streams-streamlang-remove-by-prefix]

Removes a field and all nested fields matching a prefix.

:::{note}
The `where` clause is not supported on `remove_by_prefix`. This processor cannot be used inside condition blocks.
:::

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Prefix to match. All fields under this prefix are removed. |

```yaml
- action: remove_by_prefix
  from: attributes.debug
```

### Append [streams-streamlang-append]

Adds values to an array field. Creates the field as an array if it doesn't exist.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `to` | string | Yes | Array field to append to. |
| `value` | array | Yes | Values to append. |
| `allow_duplicates` | boolean | No | When `false`, duplicate values are not appended. |

```yaml
- action: append
  to: attributes.tags
  value:
    - processed
    - reviewed
```

### Convert [streams-streamlang-convert]

Converts a field value to a different data type.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field containing the value to convert. |
| `type` | string | Yes | Target data type: `integer`, `long`, `double`, `boolean`, or `string`. |
| `to` | string | No | Target field for the converted value. Defaults to the source field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

:::{note}
When using `convert` inside a condition (`where` block), you must set a `to` field that is different from `from`.
:::

```yaml
- action: convert
  from: attributes.status_code
  type: integer
  to: attributes.status_code_int
```

### Replace [streams-streamlang-replace]

Replaces portions of a string field that match a regular expression with a replacement string.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field containing the string. |
| `pattern` | string | Yes | Regular expression pattern to match (Java regex). |
| `replacement` | string | Yes | Replacement string. Supports capture group references (for example, `$1`, `$2`). |
| `to` | string | No | Target field for the result. Defaults to the source field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

```yaml
- action: replace
  from: attributes.email
  pattern: "(\\w+)@(\\w+\\.\\w+)"
  replacement: "***@$2"
```

### Drop document [streams-streamlang-drop]

Prevents a document from being indexed. A `where` condition is required to avoid accidentally dropping all documents.

:::{warning}
If no condition is set, the default `always` condition drops every document.
:::

```yaml
- action: drop_document
  where:
    field: attributes.path
    eq: "/health"
```

### Math [streams-streamlang-math]

Evaluates an arithmetic expression and stores the result.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `expression` | string | Yes | A TinyMath expression. Can reference fields directly (for example, `attributes.price * attributes.quantity`). |
| `to` | string | Yes | Target field for the result. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if any referenced field is missing. |

```yaml
- action: math
  expression: "attributes.bytes / attributes.duration"
  to: attributes.throughput
```

### Redact [streams-streamlang-redact]

Redacts sensitive data in a string field by matching grok patterns and replacing the matched content.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field to redact. |
| `patterns` | string[] | Yes | Grok patterns that match sensitive data. |
| `pattern_definitions` | object | No | Custom pattern definitions. |
| `prefix` | string | No | Prefix for the redacted placeholder. Defaults to `<`. |
| `suffix` | string | No | Suffix for the redacted placeholder. Defaults to `>`. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. Defaults to `true`. |

```yaml
- action: redact
  from: body.message
  patterns:
    - "%{IP:client_ip}"
    - "%{EMAILADDRESS:email}"
```

### Uppercase [streams-streamlang-uppercase]

Converts a string field to uppercase.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field. |
| `to` | string | No | Target field. Defaults to the source field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

```yaml
- action: uppercase
  from: attributes.level
```

### Lowercase [streams-streamlang-lowercase]

Converts a string field to lowercase.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field. |
| `to` | string | No | Target field. Defaults to the source field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

```yaml
- action: lowercase
  from: attributes.method
  to: attributes.method_lower
```

### Trim [streams-streamlang-trim]

Removes leading and trailing whitespace from a string field.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field. |
| `to` | string | No | Target field. Defaults to the source field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

```yaml
- action: trim
  from: attributes.name
```

### Join [streams-streamlang-join]

Concatenates the values of multiple fields with a delimiter.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string[] | Yes | Source fields to join. |
| `delimiter` | string | Yes | Delimiter placed between values. |
| `to` | string | Yes | Target field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if any source field is missing. |

```yaml
- action: join
  from:
    - attributes.first_name
    - attributes.last_name
  delimiter: " "
  to: attributes.full_name
```

### Split [streams-streamlang-split]

Splits a string field into an array.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Source field to split. |
| `separator` | string | Yes | Regex separator pattern. |
| `to` | string | No | Target field. Defaults to the source field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |
| `preserve_trailing` | boolean | No | When `true`, keep empty trailing elements. |

:::{note}
When using `split` inside a condition (`where` block), you must set a `to` field that is different from `from`.
:::

```yaml
- action: split
  from: attributes.tags_string
  separator: ","
  to: attributes.tags
```

### Sort [streams-streamlang-sort]

Sorts an array field.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | Yes | Array field to sort. |
| `to` | string | No | Target field. Defaults to the source field. |
| `order` | string | No | Sort order: `asc` (default) or `desc`. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if the source field is missing. |

```yaml
- action: sort
  from: attributes.scores
  order: desc
```

### Concat [streams-streamlang-concat]

Concatenates a mix of field values and literal strings into a single field.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | array | Yes | Items to concatenate. Each item is either `{ type: "field", value: "<field_name>" }` or `{ type: "literal", value: "<text>" }`. |
| `to` | string | Yes | Target field. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if any referenced field is missing. |

```yaml
- action: concat
  from:
    - type: literal
      value: "User: "
    - type: field
      value: attributes.username
    - type: literal
      value: " (ID: "
    - type: field
      value: attributes.user_id
    - type: literal
      value: ")"
  to: attributes.user_summary
```

### Network direction [streams-streamlang-network-direction]

Determines network traffic direction (inbound, outbound, internal, external) based on source and destination IP addresses.

Specify exactly one of `internal_networks` or `internal_networks_field`.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `source_ip` | string | Yes | Field containing the source IP address. |
| `destination_ip` | string | Yes | Field containing the destination IP address. |
| `target_field` | string | No | Target field for the direction result. |
| `internal_networks` | string[] | One of `internal_networks` or `internal_networks_field` | List of internal network CIDR ranges. |
| `internal_networks_field` | string | One of `internal_networks` or `internal_networks_field` | Field containing the list of internal networks. |
| `ignore_missing` | boolean | No | When `true`, skip this processor if a source field is missing. |

```yaml
- action: network_direction
  source_ip: attributes.source.ip
  destination_ip: attributes.destination.ip
  target_field: attributes.network.direction
  internal_networks:
    - "10.0.0.0/8"
    - "172.16.0.0/12"
    - "192.168.0.0/16"
```

## Conditions [streams-streamlang-conditions]

Conditions are Boolean expressions used to control when processors run and how data is routed. They appear in `where` clauses on processors, in [condition blocks](#streams-streamlang-condition-blocks), and in stream [partitioning](./partitioning.md).

### Comparison operators [streams-streamlang-comparison-operators]

Each comparison condition specifies a `field` and an operator with a value:

| Operator | Description | Example value |
| --- | --- | --- |
| `eq` | Equals | `"active"`, `200` |
| `neq` | Not equals | `"error"` |
| `lt` | Less than | `100` |
| `lte` | Less than or equal to | `100` |
| `gt` | Greater than | `0` |
| `gte` | Greater than or equal to | `1` |
| `contains` | Field value contains the substring | `"error"` |
| `startsWith` | Field value starts with the string | `"/api"` |
| `endsWith` | Field value ends with the string | `".log"` |
| `includes` | Multivalue field includes the value | `"admin"` |

```yaml
where:
  field: attributes.status
  eq: active
```

### Range conditions [streams-streamlang-range-conditions]

Use `range` to match values within a numeric range. You can combine any of `gt`, `gte`, `lt`, and `lte`:

```yaml
where:
  field: attributes.status_code
  range:
    gte: 200
    lt: 300
```

### Existence conditions [streams-streamlang-existence-conditions]

Use `exists` to check whether a field is present:

```yaml
# Field must exist
where:
  field: attributes.user_id
  exists: true

# Field must not exist
where:
  field: attributes.temp
  exists: false
```

### Logical operators [streams-streamlang-logical-operators]

Combine conditions using `and`, `or`, and `not`:

```yaml
# All conditions must be true
where:
  and:
    - field: attributes.env
      eq: production
    - field: attributes.level
      eq: error

# At least one condition must be true
where:
  or:
    - field: attributes.level
      eq: error
    - field: attributes.level
      eq: warn

# Negate a condition
where:
  not:
    field: attributes.path
    startsWith: "/internal"
```

### Special conditions [streams-streamlang-special-conditions]

| Condition | Description |
| --- | --- |
| `always: {}` | Always evaluates to `true`. |
| `never: {}` | Always evaluates to `false`. |

## Condition blocks [streams-streamlang-condition-blocks]

Condition blocks group processors that should only run when a condition is met. Use a `condition` step with nested `steps`:

```yaml
steps:
  - condition:
      field: attributes.env
      eq: production
      steps:
        - action: set
          to: attributes.is_prod
          value: true
        - action: remove
          from: attributes.debug_info
```

Condition blocks can be nested for complex logic:

```yaml
steps:
  - condition:
      field: attributes.source
      eq: webserver
      steps:
        - action: grok
          from: body.message
          patterns:
            - "%{IP:attributes.client_ip} %{WORD:attributes.method} %{URIPATHPARAM:attributes.path} %{NUMBER:attributes.status}"
        - condition:
            field: attributes.status
            gte: 500
            steps:
              - action: set
                to: attributes.alert_level
                value: critical
```

## Field naming [streams-streamlang-field-naming]

For [wired streams](../wired-streams.md), fields must follow OTel-compatible namespacing. Custom fields must use one of these prefixes:

- `attributes.*`
- `body.structured.*`
- `resource.attributes.*`
- `scope.attributes.*`

The following special fields are allowed without a namespace prefix: `@timestamp`, `observed_timestamp`, `trace_id`, `span_id`, `severity_text`, `severity_number`, `event_name`, `body`, and `body.text`.

System-managed fields like `stream.name` are reserved and cannot be modified by processors.

## Examples [streams-streamlang-examples]

### Parse and enrich web server logs [streams-streamlang-example-webserver]

```yaml
steps:
  - action: grok
    from: body.message
    patterns:
      - "%{IP:attributes.client_ip} - %{DATA:attributes.user} \\[%{HTTPDATE:attributes.timestamp}\\] \"%{WORD:attributes.method} %{URIPATHPARAM:attributes.path} HTTP/%{NUMBER:attributes.http_version}\" %{NUMBER:attributes.status} %{NUMBER:attributes.bytes}"
  - action: date
    from: attributes.timestamp
    formats:
      - "dd/MMM/yyyy:HH:mm:ss Z"
  - action: convert
    from: attributes.status
    type: integer
  - action: convert
    from: attributes.bytes
    type: long
  - action: remove
    from: attributes.timestamp
    ignore_missing: true
```

### Conditionally tag and drop documents [streams-streamlang-example-conditional]

```yaml
steps:
  - condition:
      field: attributes.level
      eq: DEBUG
      steps:
        - action: drop_document
  - condition:
      and:
        - field: attributes.level
          eq: ERROR
        - field: attributes.service
          eq: payments
      steps:
        - action: append
          to: attributes.tags
          value:
            - critical
            - pager
        - action: set
          to: attributes.priority
          value: 1
```

### Redact sensitive data [streams-streamlang-example-redact]

```yaml
steps:
  - action: redact
    from: body.message
    patterns:
      - "%{EMAILADDRESS:email}"
      - "%{IP:ip_address}"
  - action: replace
    from: attributes.auth_header
    pattern: "Bearer .+"
    replacement: "Bearer [REDACTED]"
```
