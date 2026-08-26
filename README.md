# UART CLI application for Livt

This repository is the complete UART example for the reusable
`Eccelerators.Cli` package. It connects `Eccelerators.Cli.Cli` to
`Livt.IO.Uart` and demonstrates static FPGA command dispatch, arguments,
editing, prompts, and lossless transmit backpressure.

The first stable release is `1.0.0`.

## Terminal behavior

- prompt: `> `
- local echo enabled
- `CR`, `LF`, and `CRLF` accepted
- backspace and delete editing
- responses terminated with `CRLF`
- 64-byte maximum command line
- 64-byte buffered CLI output
- eight maximum arguments; longer argument lists are rejected before dispatch

Supported commands:

| Command | Response |
|---|---|
| `help` | `help hello bye echo` |
| `hello` | `Hello` |
| `bye` | `Bye` |
| `echo one two` | `one two` |
| any other command | `Unknown command` |

Example:

```text
> help
help hello bye echo
> echo FPGA CLI
FPGA CLI
> hello
Hello
>
```

## Architecture

`UartCliApp` owns one `Livt.IO.Uart` and one `Eccelerators.Cli.Cli`. Its continuous
process:

1. services pending prompts;
2. dispatches a completed command;
3. moves one queued CLI byte into UART when transmit space is available;
4. supplies one received UART byte when the CLI can accept it.

Responses are written to the CLI output FIFO before command completion. UART
bytes are consumed from that FIFO only after `Uart.Transmit()` succeeds.

Commands are deliberately dispatched in `UartCliApp.DispatchCommand()`. To add
a synthesized command, add an exact `CommandEquals()` branch and a handler that
returns false until its complete response can be queued. This ownership pattern
keeps application policy separate from the reusable CLI core. The `echo` command
delegates its reusable space-separated argument formatting and atomic CRLF
output to `Cli.TryWriteArgumentsLine()`.

## Dependencies

The project uses the published CLI and UART packages:

```toml
[dependencies]
"Eccelerators.Cli" = "1.0.0"
"Livt.IO" = "1.0.0"
```

Both dependencies are synchronized from the package registry. The CLI package
uses `Livt.IO.Ram` for its parser and output buffers, so applications do not need
to provide separate CLI storage components.

After publishing, consume this complete application from another Livt project
with:

```toml
[dependencies]
"UartCliApp" = "1.0.0"
```

## Hardware interface

The generated Vivado wrapper exposes `Clk`, active-high `Rst`, `rx`, and `tx`.
Its clock context is fixed at 100 MHz. The `Livt.IO` UART uses 868 clock ticks
per bit, which provides 115200 baud at that clock rate, with 8 data bits, no
parity, and one stop bit (8-N-1). Connect `rx` and `tx` to 3.3 V UART logic; use
an appropriate USB-to-UART adapter rather than RS-232 voltage levels.

## Build and test

```bash
livt validate
livt test
livt build --release -W all,error
```

The UART integration test runs an interactive session covering the prompt,
help, argument echo, excessive arguments, backspace editing, CRLF handling,
unknown commands, and multiple consecutive command responses.

## Project layout

```text
src/UartCliApp.lvt
tests/UartCliAppTest.lvt
CHANGELOG.md
livt.toml
```

## License

MIT. See [LICENSE](LICENSE).
