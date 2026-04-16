# Design Note

This note records the structure of the UART CLI application and the current
implementation constraints that shape the sample.

## UART Command Loop

`UartCliApp` demonstrates that FPGA-oriented Livt designs can use familiar
software architecture ideas without hiding the hardware data flow. The top-level
component combines UART transport, a bounded input buffer, and a small command
dispatcher.

The loop follows a compact read-eval-print shape:

1. Read one byte from UART when data is available.
2. Echo normal input bytes back to the UART transmitter.
3. Ignore `CR` so common `CRLF` terminal input works as one command.
4. On `LF`, parse the buffered line.
5. Execute the selected command component.

The design is intentionally small, deterministic, and bounded so it stays
readable as an application example.

## Component Responsibilities

The current design splits responsibilities across a few small components:

- `UartCliApp` owns the UART instance, input queue, byte echo, and line boundary
  handling.
- `CommandParser` receives a completed line and selects a command component.
- `CommandInterface` defines the shared execution contract.
- `HelpCommand`, `HelloCommand`, `ByeCommand`, and `UnknownCommand` provide the
  response behavior.

That separation keeps the UART-facing loop in one place while making command
behavior easy to extend.

## Response Length

The `Uart` helper provides an all-or-nothing `Send()` operation: it sends a byte
array only when the transmit FIFO has enough free space for the full response.
For that reason, command responses in this sample are intentionally short. The
input echo can occupy part of the transmit FIFO when the command response is
queued.

A richer CLI could add a response scheduler that feeds longer messages into UART
over time.

## Queue Dependency

The app currently uses the external `Queue` package as its bounded input-line
buffer. `Queue@1.0.0` provides FIFO behavior, so `CommandParser` can compare
command bytes in the same order the terminal sends them.

## Command Pattern

The command pattern keeps the example extensible without making it large:

- the parser decides which command should run
- each command component owns only its response text
- the top-level app does not need to know command-specific behavior

Natural next steps include commands such as `status`, `version`, or `echo`, plus
argument parsing once Livt examples need richer command input.

## Line Endings

Serial tools do not always agree on line endings. Some send `LF`, some send
`CRLF`, and some can be configured either way. This example executes commands on
`LF` and ignores `CR`, which makes standard `CRLF` input work without executing
the command twice.

## Current Limitations

- Input buffering is bounded by the queue dependency.
- Overlong input is dropped by the queue once the queue is full.
- Command responses are intentionally short enough for the current UART transmit
  FIFO while echoed input bytes may still be queued.
- The app does not currently emit a prompt or automatically append response
  line endings.
- The parser uses exact command matching and does not support arguments.
