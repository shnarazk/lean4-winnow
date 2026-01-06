# lean4-winnow

Parser combinator library for Rust programmers using [Lean4](https://github.com/leanprover/lean4), inspired by [winnow-rs](https://github.com/winnow-rs/winnow).

## Overview

`lean4-winnow` provides a collection of parser combinators built on top of Lean4's standard `Parsec` library. These combinators allow you to build complex parsers by composing simpler ones, making it easy to parse structured text data.

## Features

- **Parser combinators** for common parsing tasks
- **Winnow-compatible API** inspired by the Rust winnow library
- **Built on Parsec** uses Lean4's `Std.Internal.Parsec` for efficient parsing
- **Type-safe** leverages Lean4's type system for correctness guarantees

## Installation

Add `lean4-winnow` to your Lean4 project by adding it as a dependency in your `lakefile.toml`:

```toml
[[require]]
name = "WinnowParsers"
```

Or clone the repository directly:

```bash
git clone https://github.com/shnarazk/lean4-winnow.git
```

## Usage

Import the module in your Lean4 code:

```lean
import WinnowParsers
open WinnowParsers
```

### Basic Examples

#### Parsing numbers

```lean
-- Parse a natural number
#eval parse number "123"  -- some 123

-- Parse a signed integer
#eval parse number_signed "-42"  -- some -42

-- Parse a single digit
#eval parse single_digit "5"  -- some 5
```

#### Parsing text

```lean
-- Parse alphabetic characters
#eval parse alphabets "hello"  -- some "hello"

-- Parse with separators
#eval parse (separated number (pchar ',')) "1,2,3"  -- some #[1, 2, 3]
```

#### Custom parsers

```lean
-- Parse a label (alphabets followed by ':')
def label := many1Chars asciiLetter <* pchar ':'
#eval parse label "name:"  -- some "name"

-- Parse repeated patterns
def values := repeated (whitespaces? *> number <* whitespaces?)
#eval parse values "1 2 3"  -- some #[1, 2, 3]
```

## API Documentation

### Core Parsers

- **`eol`**: Parses end of line
- **`whitespaces`**: Parses one or more spaces or TABs
- **`whitespaces?`**: Parses zero or more spaces or TABs
- **`alphabets`**: Parses one or more ASCII letters [A-Za-z]+
- **`number`**: Parses a natural number
- **`number_signed`**: Parses a signed integer (positive or negative)
- **`number_p`**: Parses a positive integer
- **`number_m`**: Parses a negative integer
- **`single_digit`**: Parses a single digit and converts to its numeric value

### Combinators

- **`separated p s`**: Repeats parser `p` separated by `s`
- **`repeated p`**: Repeats parser `p` (equivalent to `repeat` in Winnow)
- **`repeatTill p e`**: Repeats parser `p` until parser `e` succeeds, returning both results
- **`separator ch`**: Repeats character `ch` one or more times and discards the result
- **`separator₀ ch`**: Repeats character `ch` zero or more times and discards the result

### Utility Functions

- **`parse parser source`**: Runs a parser on a source string and returns an `Option` result

## Development

This project uses:
- Lean 4.27.0-rc1
- Lake build system
- Nix flake for development environment

### Building

```bash
lake build
```

### Testing

The library includes inline tests using `#guard` assertions. These tests are verified during compilation.

## Examples

See the `test` namespace in `WinnowParsers.lean` for more examples:

```lean
namespace test

-- Parse alphabets followed by ':'
def label := many1Chars asciiLetter <* pchar ':'
#guard Parser.run label "Game: 0, " == Except.ok "Game"

-- Parse sequence of "label: number" separated by ","
def fields := separated (separator₀ ' ' *> label *> separator ' ' *> number) (pchar ',')
#guard Parser.run fields "a: 0, bb: 8" == Except.ok #[0, 8]

end test
```

## License

This project is licensed under the Mozilla Public License Version 2.0. See the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Related Projects

- [winnow-rs](https://github.com/winnow-rs/winnow) - A Rust parser combinator library that inspired this project
- [Lean4](https://github.com/leanprover/lean4) - The Lean theorem prover and programming language
