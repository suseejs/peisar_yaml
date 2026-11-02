<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD041 -->
<div align="center">
 <img src="https://pub-c9ba018358dd48a99b70013b65a25e5f.r2.dev/logo/peisar.webp" width="160" height="160" alt="prisar" />
  <h1>Peisar YAML</h1>
  <p>A YAML parsing helpers for Rust and an optional N-API interface for Node.js</p>
</div>

`peisar_yaml` provides YAML parsing helpers for Rust and an optional N-API
interface for Node.js. The default Rust API can parse YAML into a dynamic
`serde_json::Value`, deserialize it into your own Serde types, read YAML files,
and extract YAML front matter from Markdown.

## Rust usage

Add the crate to your `Cargo.toml` (adjust the path to the checkout):

```toml
[dependencies]
peisar_yaml = { path = "../peisar_yaml" }
serde = { version = "1", features = ["derive"] }
```

Parse into a dynamic value:

```rust
let value = peisar_yaml::yaml_parser("name: example\nretries: 3\n")?;
assert_eq!(value["name"], "example");
# Ok::<(), String>(())
```

Deserialize YAML into a type or load it from a file:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct Settings {
    host: String,
    port: u16,
}

let settings: Settings =
    peisar_yaml::parse_yaml_config("host: localhost\nport: 8080\n")?;
let from_file: Settings = peisar_yaml::load_yaml("settings.yaml")?;
# let _ = (settings, from_file);
# Ok::<(), String>(())
```

Extract front matter and the remaining Markdown:

```rust
let parsed = peisar_yaml::frontmatter(
    "---\ntitle: Example\n---\n\n# Content\n".to_string(),
)?;
assert_eq!(parsed.clean_content(), "# Content\n");
assert_eq!(parsed.yaml_data().unwrap()["title"], "Example");
# Ok::<(), String>(())
```

Parsing and file-loading functions return `Result`; errors are represented as
strings in the default Rust API. Relative file paths are resolved against the
process's current working directory.

## Node.js / N-API

Enable the `npm` Cargo feature to compile the N-API exports:

```toml
peisar_yaml = { path = "../peisar_yaml", features = ["npm"] }
```

This feature exposes `yaml_parser`, `load_yaml`, and `frontmatter` through
N-API. Building a distributable Node.js package additionally requires an
N-API-compatible packaging setup in the consuming project. In this interface,
`load_yaml` resolves paths from the process's current working directory.

## Development

Run the Rust tests and generate the API documentation with:

```sh
cargo test
cargo doc --no-deps
```
