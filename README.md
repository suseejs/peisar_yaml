<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD041 -->
<div align="center">
 <img src="https://pub-c9ba018358dd48a99b70013b65a25e5f.r2.dev/logo/peisar.webp" width="160" height="160" alt="prisar" />
  <h1>Peisar YAML</h1>
  <p>YAML parsing helpers for Rust and an optional N-API interface for Node.js</p>
</div>

`peisar_yaml` provides YAML parsing helpers for Rust and an optional N-API
interface for Node.js. With the default features, it can parse YAML into a
dynamic `serde_json::Value`, deserialize it into your own Serde types, read
YAML files, and extract YAML front matter from Markdown.

## Rust usage

Add the crate to your `Cargo.toml` (adjust the path to the checkout):

```toml
[dependencies]
peisar_yaml = "0.1.0"
serde = { version = "1", features = ["derive"] }
```

Parse into a dynamic JSON-compatible value:

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

The default Rust API provides:

| Function                         | Description                                                                       |
| -------------------------------- | --------------------------------------------------------------------------------- |
| `yaml_parser(yaml_str)`          | Parse YAML into a `serde_json::Value`.                                            |
| `parse_yaml_config<T>(yaml_str)` | Deserialize YAML into a caller-provided `T: DeserializeOwned`.                    |
| `load_yaml<T>(path)`             | Read a YAML file and deserialize it into a caller-provided `T: DeserializeOwned`. |
| `frontmatter(content)`           | Return the Markdown body and optional YAML metadata.                              |

These functions return `Result` with parser or file-read errors represented as
strings. Relative paths passed to `load_yaml` are resolved from the process's
current working directory. `frontmatter` returns the input unchanged with no
metadata when there is no complete leading front-matter block; invalid YAML in
a complete block returns an error.

## Node.js / N-API

Enable the `npm` Cargo feature to compile the N-API exports:

```toml
peisar_yaml = { version = "0.1.0", features = ["npm"] }
```

The N-API interface exposes these JavaScript functions:

| JavaScript function    | Description                                                                                             |
| ---------------------- | ------------------------------------------------------------------------------------------------------- |
| `yamlParser(yamlStr)`  | Parse YAML and return its JSON-compatible value.                                                        |
| `loadYaml(filePath)`   | Read and parse YAML, returning its JSON-compatible value.                                               |
| `frontmatter(content)` | Return `{ cleanContent, yamlData }`; `yamlData` is absent when there is no complete front-matter block. |

The N-API feature also provides the Rust `parse_yaml_config<T>` function, but
the JavaScript interface does not expose it. In this feature, the Rust
`yaml_parser`, `load_yaml`, and `frontmatter` functions return N-API errors
instead of string errors. YAML parse failures from JavaScript are thrown as
errors; file paths passed to `loadYaml` are resolved from the process's
current working directory. File-read failures in the current N-API wrapper
panic rather than returning a recoverable error.

The crate provides the native N-API exports; using them from Node.js requires
a loader/package that supplies the platform-specific native binding.

## Development

Run the Rust tests and generate the API documentation with:

```sh
cargo test
cargo doc --no-deps
```
