<h1 align="center">mermaid.mq</h1>

A [Mermaid](https://mermaid.js.org/) diagram parser implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- **Flowchart / Graph** — nodes (rect, round, diamond, circle, stadium), edges with optional labels, all direction keywords (`LR`, `RL`, `TD`, `TB`, `BT`)
- **Sequence diagram** — participants, actors, and messages with all arrow types (`->>`, `-->>`, `->`, `-->`, `-x`, `--x`)
- **Pie chart** — title and labeled slices
- **Class diagram** — class names and relationships

## Installation

Copy `mermaid.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp mermaid.mq ~/.local/mq/config/
```

### HTTP Import (no local installation needed)

If `mq` was built with the `http-import` feature, you can import directly from GitHub without any local setup:

```sh
mq -I raw 'import "github.com/harehare/mermaid.mq" | mermaid::mermaid_parse(.)' diagram.mmd
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq -I raw 'import "github.com/harehare/mermaid.mq@v1.0.0" | mermaid::mermaid_parse(.)' diagram.mmd
```

## Usage

```sh
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.)' diagram.mmd
```

## API

### `mermaid_parse(input)`

Parses a Mermaid diagram string and returns structured data.

| Diagram type | Output keys |
|---|---|
| `flowchart` / `graph` | `type`, `direction`, `nodes`, `edges` |
| `sequenceDiagram` | `type`, `participants`, `messages` |
| `pie` | `type`, `title`, `slices` |
| `classDiagram` | `type`, `classes`, `relations` |
| other | `type`, `raw_lines` |

## Examples

### Flowchart

Given `flow.mmd`:

```
graph LR
  A[Start] --> B{Is it?}
  B -->|Yes| C[OK]
  B -->|No| D[Not OK]
  C --> E((End))
```

```sh
# Extract all node IDs
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."nodes" | map(fn(n): n["id"];)' flow.mmd
# => ["A", "B", "C", "D", "E"]

# Extract edges with labels
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."edges" | filter(fn(e): e["label"] != "";)' flow.mmd
# => [{"from":"B","to":"C","label":"Yes"}, {"from":"B","to":"D","label":"No"}]

# Get direction
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."direction"' flow.mmd
# => "LR"
```

### Sequence Diagram

Given `seq.mmd`:

```
sequenceDiagram
  participant Alice
  participant Bob
  Alice->>Bob: Hello Bob!
  Bob-->>Alice: Hi Alice!
  Alice->>Bob: How are you?
```

```sh
# List participants
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."participants" | map(fn(p): p["name"];)' seq.mmd
# => ["Alice", "Bob"]

# Filter messages from Alice
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."messages" | filter(fn(m): m["from"] == "Alice";)' seq.mmd
# => [{"from":"Alice","to":"Bob","arrow":"->>","text":"Hello Bob!"}, ...]
```

### Pie Chart

Given `pie.mmd`:

```
pie title Pets adopted by volunteers
  "Dogs" : 386
  "Cats" : 85
  "Rats" : 15
```

```sh
# Get the title
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."title"' pie.mmd
# => "Pets adopted by volunteers"

# Get the largest slice
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."slices" | sort_by(fn(s): s["value"];) | last()' pie.mmd
# => {"label":"Dogs","value":386}
```

### Class Diagram

Given `class.mmd`:

```
classDiagram
  class Animal
  class Dog
  class Cat
  Animal <|-- Dog : extends
  Animal <|-- Cat : extends
```

```sh
# List all classes
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."classes" | map(fn(c): c["name"];)' class.mmd
# => ["Animal", "Dog", "Cat"]

# List relationships
mq -L . -I raw 'import "mermaid" | mermaid::mermaid_parse(.) | ."relations"' class.mmd
# => [{"from":"Animal","to":"Dog","rel":"<|--","label":"extends"}, ...]
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.5 or later.

## License

MIT
