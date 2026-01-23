---
applies_to:
  serverless: ga
  stack: preview =9.1, ga 9.2+
---

# Streamlang [streams-streamlang]

Streamlang is a YAML-friendly domain-specific language (DSL) for defining processing configurations for Streams. It provides a consistent interface for specifying how documents should be transformed, regardless of where the processing is executed.

Streamlang enables seamless movement between query-time and ingest-time processing by abstracting the underlying execution targets:

- **{{es}} ingest pipelines**: Traditional ingest-time processing
- **ES|QL**: Query-time processing
- **OTTL**: OpenTelemetry collector processing (planned)

When you configure processors and conditions in the Streams UI, you're working with Streamlang. The UI provides two editing modes:

- **Interactive mode**: A form-based editor for building Streamlang configurations visually
- **YAML mode**: A code editor for writing Streamlang directly as YAML

You can switch between modes based on your preference. If your configuration uses advanced features like deeply nested conditions that can't be represented in interactive mode, the UI automatically defaults to YAML mode.

## Why Streamlang? [streams-streamlang-why]

Streamlang was created because existing processing languages weren't suited for spanning both ingest-time and query-time processing:

- **ES|QL** is inherently a query-time language. Using a subset for ingest-time processing would create a confusing experience.
- **Ingest pipelines** with Painless scripting are too flexible to run at acceptable speeds at query time.
- **OTTL** (OpenTelemetry Transformation Language) is too flexible in ways that are hard to unify with the limitations of ingest pipelines and ES|QL.

Streamlang provides a unified interface over these different feature sets, so you can specify processing independent of where it's eventually executed.

## Editing modes [streams-streamlang-modes]

The Streams processing UI provides two modes for editing Streamlang configurations:

### Interactive mode [streams-streamlang-interactive-mode]

Interactive mode provides a form-based interface for creating and editing processors. This mode is ideal for:

- Building simple processing configurations
- Users who prefer a guided, visual approach
- Configurations that don't require deeply nested conditions

### YAML mode [streams-streamlang-yaml-mode]

YAML mode provides a code editor for writing Streamlang directly. This mode is useful for:

- Advanced configurations with complex or deeply nested conditions
- Copying and pasting configurations between streams
- Users who prefer working with code

In YAML mode, simulation works differently than in interactive mode:

- **Explicit simulation**: Select the **Simulate** button to run the simulation
- **Full simulation**: Runs all steps against the sample data
- **Run up to step**: Set breakpoints to simulate only up to a specific step, useful for debugging complex configurations

The UI automatically selects YAML mode when the current configuration can't be represented in interactive mode (for example, when nesting levels are too deep).

## Structure [streams-streamlang-structure]

A Streamlang configuration consists of a `steps` array containing one or more actions. Each step defines a processing operation to apply to documents.

```yaml
steps:
- action: <action_type>
  <action_parameters>
- action: <action_type>
  <action_parameters>
```

### Example configuration [streams-streamlang-example]

The following example demonstrates various Streamlang actions:

```yaml
steps:
- action: rename
  from: attributes.old_name
  to: attributes.new_name
- action: set
  to: attributes.status
  value: active
- action: grok
  from: body.message
  patterns:
  - "%{IP:attributes.client_ip} - %{WORD:attributes.method}"
- action: date
  from: attributes.timestamp
  formats:
  - yyyy-MM-dd'T'HH:mm:ss.SSSZ
  - yyyy-MM-dd HH:mm:ss
  to: attributes.parsed_time
  output_format: yyyy-MM-dd
- action: dissect
  from: body.log
  pattern: "%{attributes.client} %{attributes.method} %{attributes.path}"
- action: append
  to: attributes.tags
  value:
  - new_tag
```

## Actions [streams-streamlang-actions]

Streamlang supports the following actions, which correspond to the [processors](./extract.md#streams-extract-processors) available in the Streams UI:

| Action | Description |
|--------|-------------|
| `append` | Adds values to an existing array field, or creates the field as an array if it doesn't exist. |
| `convert` | Converts a field to a different type (string, integer, long, float, double, boolean). |
| `date` | Converts date strings into timestamps with options for timezone, locale, and output formatting. |
| `dissect` | Extracts fields from structured log messages using defined delimiters. |
| `drop` | Drops the document without raising errors, useful for conditional filtering. |
| `grok` | Extracts fields from unstructured log messages using predefined or custom patterns. |
| `math` | Evaluates arithmetic or logical expressions. |
| `remove` | Removes existing fields from documents. |
| `rename` | Changes the name of a field, moving its value to a new field name. |
| `replace` | Replaces parts of a string field using a regular expression pattern. |
| `set` | Assigns a specific value to a field, creating or overwriting as needed. |

For detailed information about each action, refer to the individual [processor documentation](./extract.md#streams-extract-processors).

### Common action parameters [streams-streamlang-parameters]

Most actions share these common parameters:

- `from`: The source field to read from
- `to`: The target field to write to
- `value`: The value to set or append
- `condition`: A [condition](#streams-streamlang-conditions) that controls when the action runs

## Conditions [streams-streamlang-conditions]

Conditions are boolean expressions used to control when actions execute. They're also used in other Streams features like [partitioning](./partitioning.md) for routing data and significant events.

### Filter conditions [streams-streamlang-filter-conditions]

Filter conditions compare a field against a value using an operator:

```yaml
condition:
  field: attributes.status
  eq: active
```

#### Supported operators [streams-streamlang-operators]

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `eq: active` |
| `neq` | Not equals | `neq: inactive` |
| `lt` | Less than | `lt: 100` |
| `lte` | Less than or equals | `lte: 100` |
| `gt` | Greater than | `gt: 0` |
| `gte` | Greater than or equals | `gte: 1` |
| `contains` | Contains substring | `contains: error` |
| `startsWith` | Starts with string | `startsWith: WARN` |
| `endsWith` | Ends with string | `endsWith: .log` |
| `exists` | Field exists | `exists: true` |

#### Range conditions [streams-streamlang-range]

For range comparisons, you can combine multiple bounds:

```yaml
condition:
  field: attributes.response_time
  range:
    gte: 100
    lt: 500
```

### Logical conditions [streams-streamlang-logical]

Combine multiple conditions using logical operators:

#### AND condition [streams-streamlang-and]

All conditions must be true:

```yaml
condition:
  and:
  - field: attributes.env
    eq: prod
  - field: attributes.level
    eq: error
```

#### OR condition [streams-streamlang-or]

At least one condition must be true:

```yaml
condition:
  or:
  - field: attributes.level
    eq: error
  - field: attributes.level
    eq: critical
```

#### NOT condition [streams-streamlang-not]

Negates a condition:

```yaml
condition:
  not:
    field: attributes.env
    eq: dev
```

### Special conditions [streams-streamlang-special]

#### Always condition [streams-streamlang-always]

Always evaluates to true:

```yaml
condition:
  always: {}
```

#### Never condition [streams-streamlang-never]

Always evaluates to false:

```yaml
condition:
  never: {}
```

### Conditional actions [streams-streamlang-conditional-actions]

Add a `condition` clause to any action to make it conditional:

```yaml
steps:
- action: set
  to: attributes.flag
  value: 'yes'
  condition:
    field: attributes.status
    eq: active
```

### Nested conditional steps [streams-streamlang-nested-steps]

Group multiple actions under a shared condition:

```yaml
steps:
- condition:
    field: attributes.env
    eq: prod
  steps:
  - action: set
    to: attributes.prod_flag
    value: prod-env
  - action: append
    to: attributes.tags
    value:
    - production
```

## Field namespacing [streams-streamlang-namespacing]

For wired streams, fields must follow the OpenTelemetry schema with proper namespacing:

- `attributes.*`: General attributes
- `body.structured.*`: Structured body content
- `resource.attributes.*`: Resource attributes
- `scope.attributes.*`: Scope attributes

Special fields allowed outside these namespaces include:
- `@timestamp`
- `trace_id`
- `span_id`
- `severity_text`
- `body`

:::{note}
The `stream.name` field is reserved and cannot be modified.
:::

## Validation [streams-streamlang-validation]

Streamlang performs static analysis on processing configurations for wired streams:

- **Field namespacing**: Validates that fields follow the required schema.
- **Reserved fields**: Blocks modification of system-managed fields.
- **Type safety**: Infers and tracks field types through the pipeline to ensure processors receive compatible types.
- **Mixed types detection**: Detects when conditional processors create fields with multiple possible types, which ES|QL doesn't support.

## Limitations [streams-streamlang-limitations]

Due to the nature of different transpilation targets, 100% identical behavior can't be guaranteed across ingest pipelines, ES|QL, and OTTL.

### Consistently typed fields [streams-streamlang-typed-fields]

In ES|QL, a column can only have a single type. Ingest pipelines follow a per-document processing model where types aren't enforced across documents.

For example, this pattern works in ingest pipelines but fails in ES|QL:

```yaml
steps:
- action: set
  to: attributes.result
  value: 123
  condition:
    field: attributes.type
    eq: number
- action: set
  to: attributes.result
  value: "text"
  condition:
    field: attributes.type
    eq: string
```

Because `attributes.result` could be a number or string depending on the document, ES|QL rejects this as invalid.

### Type conversion differences [streams-streamlang-conversion]

ES|QL is more lenient with type conversions than ingest pipelines. When using the `convert` action, behavior might differ between targets:

- String to integer conversions in ingest pipelines require the string to represent a valid 32-bit signed integer
- Float strings (like `"3.14"`) won't convert to integers in ingest pipelines

### Multi-value handling [streams-streamlang-multivalue]

Fields can contain one or multiple values. ES|QL and ingest processors don't handle these cases identically:

```json
{ "myfield": "A" }
{ "myfield": ["A", "B", "C"] }
```

For example, grok in ES|QL handles multiple values automatically, while the grok processor in ingest pipelines does not.

### Conditional execution [streams-streamlang-conditional-execution]

Since ES|QL enforces a consistent table shape, you can't conditionally cast a field from one type to another, while ingest pipelines allow this on a per-document basis.

Affected operations include:
- Conditionally casting or converting field types
- Grok patterns that parse a field as different types in different branches
- Deleting multiple fields with wildcards (for example, `my.field*`)

### Arrays and flattening [streams-streamlang-arrays]

Ingest pipelines operate on deeply nested JSON objects where arrays can occur at any level. ES|QL flattens documents into a table structure:

**Ingest pipeline view:**
```json
{
  "my": {
    "field": [
      { "x": 1 },
      { "x": 2 }
    ]
  }
}
```

**ES|QL view:**
```json
{
  "my.field.x": [1, 2]
}
```

Operations like renaming or deleting `my.field` behave differently. In an ingest pipeline, the entire nested structure moves. In ES|QL, only `my.field.x` exists as a column, so the operation fails.

## Classic streams escape hatch [streams-streamlang-escape-hatch]

For classic streams, you can define processing as a raw {{es}} ingest pipeline using the [manual pipeline configuration](./extract/manual-pipeline-configuration.md). This provides full access to all ingest processors and Painless scripting.

:::{warning}
Using the escape hatch binds processing to the {{es}} ingest node, disabling the ability to move processing to query-time ES|QL or an OpenTelemetry collector. This escape hatch isn't available for wired streams.
:::

## Learn more [streams-streamlang-learn-more]

- [Process documents](./extract.md): Configure processors using the Streams UI
- [Partitioning](./partitioning.md): Route data using Streamlang conditions
- [Grok processor](./extract/grok.md): Parse unstructured logs with grok patterns
- [Dissect processor](./extract/dissect.md): Extract fields using delimiters

