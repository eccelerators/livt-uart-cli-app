# UART CLI App for Livt

This package provides a small UART-driven command-line application in Livt. It
collects bytes from UART, echoes typed characters, dispatches complete command
lines, and streams command responses back over UART.

For background on the line-buffered command loop and command components, see
[DESIGN_NOTE.md](DESIGN_NOTE.md).

## 📋 Overview

The current package is organized around one top-level application and a small
set of command components:

- `UartCliApp` owns the UART instance, input queue, and command loop.
- `CommandParser` converts a completed input line into a command component.
- `HelpCommand`, `HelloCommand`, `ByeCommand`, and `UnknownCommand` provide
  concrete response behavior.

The example supports these commands:

- `help`
- `hello`
- `bye`

Unknown input returns `Unknown`. `CRLF` terminal input is supported by
ignoring `CR` and executing the command on `LF`.

## 📁 Project Structure

```text
.
├── src/
│   ├── ByeCommand.lvt
│   ├── CommandInterface.lvt
│   ├── CommandParser.lvt
│   ├── HelloCommand.lvt
│   ├── HelpCommand.lvt
│   ├── UartCliApp.lvt
│   └── UnknownCommand.lvt
├── tests/
│   └── UartCliAppTest.lvt
├── DESIGN_NOTE.md
├── LICENSE
├── README.md
└── livt.toml
```

## 🔨 Building

Build the package with:

```bash
livt build
```

The package configuration is defined in [`livt.toml`](livt.toml). The current
project name there is `UartCliApp`.

## 🧪 Running Tests

Run the full test suite with:

```bash
livt test
```

Configured test components:

- `UartCliAppTest`

The integration test simulates UART sessions for `hello`, `bye`, `help`, and an
unknown command submitted with `CRLF`.

## 📚 Component Guide

### `UartCliApp`

Top-level UART command application in the `Livt.App` namespace.

Features:

- UART receive and transmit integration through the external `Uart` package
- typed-character echo
- command execution on `LF`
- `CR` ignored for common `CRLF` terminal input
- FIFO line buffering through the external `Queue` package

Constructor:

- `new(rx: in logic, tx: out logic)`

### Commands

All commands implement `CommandInterface`.

| Command | Input | Response |
| ------- | ----- | -------- |
| `HelpCommand` | `help` | `hello bye` |
| `HelloCommand` | `hello` | `Hello` |
| `ByeCommand` | `bye` | `Bye` |
| `UnknownCommand` | any other line | `Unknown` |

## 💡 Usage

Instantiate `UartCliApp` with UART RX and TX signals and connect it to a serial
terminal configured for the same baud settings as the underlying `Uart` package.
By default, the `Uart` package uses `TICKS_PER_BIT = 868`, which corresponds to
115200 baud with a 100 MHz clock.

A typical terminal session looks like:

```text
help
hello bye
hello
Hello
bye
Bye
```

Typed command characters are echoed before the response is sent.

## 🔧 Configuration

This package depends on:

- `Uart = "1.0.0"`
- `Queue = "1.0.0"`

`Queue` is used as the bounded FIFO input-line buffer.

## 📝 Notes

- This is an application-level example, not a UART implementation.
- The UART transport is provided by the external `Uart` package.
- The command loop is intentionally small and bounded so the design remains easy
  to understand and test.
- Input longer than the queue capacity is ignored by the queue once it is full.
- Command responses are kept within the UART transmit FIFO capacity.

## 🤝 Contributing

Contributions are welcome. Areas that would be natural extensions for this
package include:

- configurable command-line length
- explicit overflow reporting for overlong input
- paced response scheduling for longer command output
- prompt and line-ending response formatting
- argument parsing beyond exact command matching
- backspace handling or simple line editing

## 📄 License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
