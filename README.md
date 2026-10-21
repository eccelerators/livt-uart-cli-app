# UART CLI application for Livt

This repository is the complete UART example for the reusable
`Eccelerators.Cli` package. It connects `Eccelerators.Cli.Cli` to
`Livt.IO.BufferedUart` and demonstrates constructor-registered commands, arguments,
editing, prompts, and lossless transmit backpressure.

Version `1.1.0` uses `Eccelerators.Cli 1.3.0` and `Livt.IO 1.2.0`.

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

`UartCliApp` owns the `BufferedUart`, `Cli`, command instances, registrations and
`CommandDispatcher`. Its constructor shows the complete integration in one place.
All commands borrow the same CLI instance. One continuous process:

1. services pending prompts;
2. dispatches a completed command;
3. attempts to enqueue one CLI output byte through `TryTransmit`;
4. attempts `TryReceive` only when the CLI can accept another input byte.

Responses are written to the CLI output FIFO before command completion. UART
bytes are consumed from that FIFO only after `BufferedUart.TryTransmit()` succeeds.
Success means FIFO acceptance, not completion on the wire; rejected output stays
queued for retry. `TryReceive` removes a byte only on success and assigns zero
on failure, so empty input is not passed to the CLI as a received zero byte.

One process owns CLI input, preserving capacity between checking and accepting
a received byte. The basic UART has finite receive storage and no RTS/CTS pins;
the peer must pace input if command processing stalls for an extended period.

Each operation implements the `ICommand` interface provided by `Eccelerators.Cli`
(the contract is shown here for reference; applications import it):

```livt
interface ICommand
{
	fn Matches() bool
	fn TryExecute() bool
}
```

`Matches()` checks the pending command without changing terminal state.
`TryExecute()` atomically accepts its response or returns false without emitting
partial output. Only `CommandDispatcher` calls `CompleteCommand()`. The first
matching handler wins, even when blocked; later handlers and the unknown-command
fallback must not run in that case.

`UartCliApp` owns the help, hello, bye and echo components alongside the CLI.
Registration happens in its constructor: each `CommandRegistration` connects an
`ICommand` to the next registry entry. The dispatcher receives the completed
chain, so registration always precedes command processing. The first entry has
highest priority; `UnknownCommand` terminates the chain and can also serve as an
empty registry.

```livt
this.echoRegistration = new CommandRegistration(this.echo, this.unknown)
this.byeRegistration = new CommandRegistration(this.bye, this.echoRegistration)
this.helloRegistration = new CommandRegistration(this.hello, this.byeRegistration)
this.helpRegistration = new CommandRegistration(this.help, this.helloRegistration)
this.dispatcher = new CommandDispatcher(this.cli, this.helpRegistration)
```

Registration wires hardware that already exists in the bitstream. The chain is
finite and fixed at construction; there is no runtime allocation, registration
count, or add/remove API. To extend it, implement `ICommand`, instantiate that
component and a `CommandRegistration` in `UartCliApp`, then connect the new entry
at the intended priority. The dispatcher and existing commands remain unchanged.
Keep the help text in sync with the available commands.

`ICommand`, `ICommandRegistry`, `CommandRegistration` and `CommandDispatcher`
come from `Eccelerators.Cli`. Only command behavior and fallback policy live in
this demo. Each registration adds hardware and scheduled calls; synthesis cost
has not been measured for this release.

Named constructor arguments explain configuration at the call site. `let` marks
immutable locals; the receive byte remains `var` because `TryReceive` writes it.

## Dependencies

The demo uses the published command-pattern API in `Eccelerators.Cli 1.3.0`:

```toml
[dependencies]
"Eccelerators.Cli" = "1.3.0"
"Livt.IO" = "1.2.0"
```

Run `livt sync` to download dependencies from the registry. No sibling package
checkout is required.

After publishing, consume this complete application from another Livt project
with:

```toml
[dependencies]
"UartCliApp" = "1.1.0"
```

## Hardware interface

`Livt.App.UartCliApp` takes an `rx: in logic` pin and a `tx: out logic` pin.
The enclosing FPGA design supplies its clock/reset context, pin assignments and
I/O standards. This package does not select a board or configure a Vivado wrapper.

The default buffered UART uses approximately 115200 baud, 8 data bits, no parity,
and one stop bit (8-N-1). Bit timing derives from the inherited clock context;
keep its frequency metadata consistent with the actual board clock. At 100 MHz,
the UART uses 868 clock ticks per bit. Connect a logic-level UART adapter that
matches the board's I/O voltage; do not connect RS-232 voltage levels directly.

## Build and test

```bash
livt sync
livt validate
livt test
livt build --release -W all,error
```

The default suite runs `CliCommandsTest` directly against the real CLI parser
and output buffers. It checks all commands, empty and multiword echo, exact name
matching, repeated commands, prompts, and retry behavior when the output queue is
full. These two application tests have no baud-rate waits and do not instantiate
UART hardware. Registration order, idle dispatch and caller-defined fallback are
covered by `CommandDispatcherTest` in the CLI package.

The slower `UartCliAppTest.RunsCompleteInteractiveSession` scenario is retained
as commented-out source and excluded from the default suite. The fast tests do
not replace its pin-level serial transport and editing coverage. Release
validation passed both fast tests and the release build against the registry
dependencies; the long UART scenario and FPGA synthesis were not rerun for this
release.

## Project layout

```text
src/UartCliApp.lvt
src/UnknownCommand.lvt
src/HelpCommand.lvt
src/HelloCommand.lvt
src/ByeCommand.lvt
src/EchoCommand.lvt
tests/CliCommandsTest.lvt
tests/UartCliAppTest.lvt
CHANGELOG.md
livt.toml
```

## License

MIT. See [LICENSE](LICENSE).
