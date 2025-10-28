# emmy_dap_types

[![Crates.io](https://img.shields.io/crates/v/emmy_dap_types.svg)](https://crates.io/crates/emmy_dap_types)
[![Documentation](https://docs.rs/emmy_dap_types/badge.svg)](https://docs.rs/emmy_dap_types)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A Rust implementation of the [Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/) (DAP) types and utilities.

## Overview

This project is a fork of [dap-rs](https://github.com/sztomi/dap-rs) with additional improvements and cross-editor compatibility enhancements.

`emmy_dap_types` provides strongly-typed Rust structures for the Debug Adapter Protocol, making it easy to build debug adapters or debug clients in Rust.

## Features

- ✅ **Complete DAP Types** - Full implementation of DAP specification types
- ✅ **Cross-Editor Compatible** - Works seamlessly with VS Code, IntelliJ IDEA, Zed, and other DAP clients
- ✅ **Flexible Deserialization** - Custom deserializer handles different client behaviors (empty `arguments` objects vs. missing fields)
- ✅ **Type-Safe** - Leverages Rust's type system for compile-time correctness
- ✅ **Serde Support** - JSON serialization/deserialization using `serde`

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
emmy_dap_types = "0.1"
```

## Usage

```rust
use emmy_dap_types::prelude::*;
use std::io::{stdin, stdout};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let input = stdin();
    let output = stdout();
    let mut server = Server::new(input, output);

    loop {
        let req: Request = server.poll_request()?;
        
        match req.command {
            Command::Initialize(args) => {
                // Handle initialize request
                let response = req.success(ResponseBody::Initialize(Some(Capabilities {
                    supports_configuration_done_request: Some(true),
                    // ... other capabilities
                    ..Default::default()
                })));
                server.respond(response)?;
            }
            Command::Launch(_) => {
                // Handle launch request
                let response = req.success(ResponseBody::Launch);
                server.respond(response)?;
            }
            Command::Threads => {
                // Handle threads request
                let response = req.success(ResponseBody::Threads(ThreadsResponseBody {
                    threads: vec![],
                }));
                server.respond(response)?;
            }
            _ => {
                // Handle other commands
            }
        }
    }
}
```

## Key Improvements Over Original

### 1. Cross-Platform Compatibility

The original `dap-rs` had issues with clients like Zed and IntelliJ IDEA that send empty `arguments: {}` objects for commands that don't require arguments, while VS Code omits the field entirely.

This fork implements a **custom deserializer** that accepts both formats:

```json
// VS Code style (no arguments field)
{"command": "threads", "seq": 1, "type": "request"}

// Zed/IntelliJ style (empty arguments object)
{"command": "threads", "seq": 1, "type": "request", "arguments": {}}
```

Both formats deserialize correctly to the same `Command::Threads` variant.

### 2. Unconditional Serialization Traits

Removed conditional compilation attributes (`#[cfg_attr(feature = "client", ...)]`) to ensure `Serialize` and `Deserialize` are always available, improving ergonomics and compatibility.

### 3. Optimized I/O Operations

The `Server` implementation has been optimized to reduce the number of write operations, improving performance for high-frequency request/response scenarios.

## Architecture

The crate is organized into several modules:

- **`requests`** - Request types and the `Command` enum
- **`responses`** - Response types and bodies
- **`events`** - Event types sent by the debug adapter
- **`types`** - Common types used across requests, responses, and events
- **`server`** - I/O utilities for implementing a debug adapter
- **`errors`** - Error types

## DAP Specification Compliance

This implementation follows the [Debug Adapter Protocol Specification](https://microsoft.github.io/debug-adapter-protocol/specification). All protocol types are documented with links to the corresponding specification sections.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Credits

This project is a fork of [dap-rs](https://github.com/sztomi/dap-rs) by [@sztomi](https://github.com/sztomi). Special thanks to the original author for the foundational work.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## See Also

- [Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/)
- [dap-rs (original)](https://github.com/sztomi/dap-rs)
