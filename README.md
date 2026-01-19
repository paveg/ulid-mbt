# ulid

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![MoonBit](https://img.shields.io/badge/MoonBit-v0.1.0-purple.svg)](https://www.moonbitlang.com/)

ULID (Universally Unique Lexicographically Sortable Identifier) implementation for [MoonBit](https://www.moonbitlang.com/), compliant with [ulid/spec](https://github.com/ulid/spec).

## Prerequisites

This project requires the **MoonBit toolchain**. Install it from the official website:

- 🌙 [MoonBit](https://www.moonbitlang.com/) - A modern programming language optimized for WebAssembly

```bash
# Verify installation
moon version
```

## Installation

```bash
moon add paveg/ulid
```

## Development

```bash
# Build the project
moon build

# Run tests
moon test

# Format code
moon fmt

# Check for errors without building
moon check
```

## Packages

- `paveg/ulid/lib`: Core ULID implementation

## Features

- [x] Full ULID specification compliance ([ulid/spec](https://github.com/ulid/spec))
- [x] Crockford's Base32 encoding/decoding
- [x] Monotonic ULID generation (strictly increasing within same millisecond)
- [x] String and binary (16-byte) representation
- [x] Validation and comparison support
- [x] Based on MoonBit standard library (`@random`)

## Usage

### Basic ULID Generation

```moonbit
// Create a generator with time source and random generator
let rand = @random.Rand::chacha8()
fn get_time() -> Int64 {
  // Return current Unix timestamp in milliseconds
  // This is runtime-dependent (see Design Notes below)
  1234567890123L
}

let gen = @lib.Generator::new(get_time, rand)
let ulid = gen.generate()
println(ulid.to_string())  // e.g., "01ARYZ6S41DEADBEEFDEADBEEF"
```

### Monotonic Generation

```moonbit
// Monotonic generator ensures strict ordering within same millisecond
let mono_gen = @lib.MonotonicGenerator::new(get_time, rand)
let ulid1 = mono_gen.generate()
let ulid2 = mono_gen.generate()
// ulid1 < ulid2 guaranteed (lexicographically)
```

### Parsing and Validation

```moonbit
// Parse from string
match @lib.ULID::from_string("01ARYZ6S410000000000000000") {
  Ok(ulid) => println(ulid.timestamp())
  Err(e) => println(e)
}

// Validate without parsing
if @lib.is_valid("01ARYZ6S410000000000000000") {
  println("Valid ULID")
}
```

### Binary Representation

```moonbit
// Convert to 16-byte binary (big-endian)
let bytes = ulid.to_bytes()

// Create from binary
match @lib.ULID::from_bytes(bytes) {
  Ok(ulid) => println(ulid.to_string())
  Err(e) => println(e)
}
```

### Comparison

```moonbit
let ulid1 = @lib.ULID::from_string("00000000000000000000000000").unwrap()
let ulid2 = @lib.ULID::from_string("01ARYZ6S410000000000000000").unwrap()

// Compare
if ulid1.compare(ulid2) < 0 {
  println("ulid1 < ulid2")
}

// Equality
if ulid1 == ulid2 {
  println("Equal")
}
```

## API Reference

### Types

| Type                 | Description                                           |
| -------------------- | ----------------------------------------------------- |
| `ULID`               | ULID structure (48-bit timestamp + 80-bit randomness) |
| `Generator`          | ULID generator with injectable time source            |
| `MonotonicGenerator` | Monotonic ULID generator                              |

### Functions

| Function                                  | Description                         |
| ----------------------------------------- | ----------------------------------- |
| `ULID::from_parts(timestamp, randomness)` | Create ULID from components         |
| `ULID::from_string(s)`                    | Parse ULID from 26-character string |
| `ULID::from_bytes(bytes)`                 | Create ULID from 16-byte binary     |
| `ulid.to_string()`                        | Convert to 26-character string      |
| `ulid.to_bytes()`                         | Convert to 16-byte binary           |
| `ulid.timestamp()`                        | Get timestamp (milliseconds)        |
| `ulid.randomness()`                       | Get randomness bytes                |
| `ulid.compare(other)`                     | Compare two ULIDs                   |
| `is_valid(s)`                             | Validate ULID string                |
| `Generator::new(get_time, rand)`          | Create generator                    |
| `gen.generate()`                          | Generate new ULID                   |
| `MonotonicGenerator::new(get_time, rand)` | Create monotonic generator          |

## Design Notes

### Time Source Injection

MoonBit's standard library doesn't provide a built-in way to get the current system time because it's runtime-dependent (Wasm, native, etc.). Therefore, this library uses **dependency injection** for the time source:

```moonbit
fn get_time() -> Int64 {
  // Implement based on your runtime environment:
  // - Browser: Use JavaScript interop to call Date.now()
  // - Node.js: Use JavaScript interop
  // - Native: Use FFI to call system time functions
  // - Testing: Return mock timestamps
}
```

This design:

- Follows MoonBit standard library patterns
- Enables easy testing with mock time
- Allows runtime-specific implementations

### Random Generation

This library uses MoonBit's `@random.Rand` with ChaCha8 algorithm from the standard library. For cryptographically secure randomness in production, consider injecting entropy from the runtime environment into the seed.

## ULID Specification

See the official specification: [ulid/spec](https://github.com/ulid/spec)

- **Format**: 26-character string (Crockford's Base32)
- **Components**: 48-bit timestamp + 80-bit randomness
- **Encoding**: `0123456789ABCDEFGHJKMNPQRSTVWXYZ` (excludes I, L, O, U)
- **Sorting**: Lexicographically sortable
- **Maximum**: `7ZZZZZZZZZZZZZZZZZZZZZZZZZ` (year 10889)

## License

Apache-2.0
