# Changelog

All notable changes to this package are documented here.

## 1.1.0

- Use the published `Eccelerators.Cli 1.3.0` and `Livt.IO 1.2.0` packages.
- Register `help`, `hello`, `bye` and `echo` as `ICommand` components using the
  CLI package's constructor-wired registration chain and dispatcher.
- Show transport setup, command ownership and registration together in
  `UartCliApp`; keep the unknown-command response application-defined.
- Use `BufferedUart` with acceptance-based transmit and receive operations,
  preserving queued output and pending commands under backpressure.
- Use named constructor arguments, immutable locals and direct boolean checks
  to make application intent clearer.
- Add two fast Livt tests covering command responses, prompts, exact matching
  and retries with a full output queue.
- Replace the long interactive session with a default-suite UART smoke test that
  checks two commands, echoes and prompts with bounded receive polling.
- Expose a compile-time baud parameter, retaining the 115200 Hz default and using
  1 MHz for the UART smoke test.
- Update installation examples and document the inherited clock and board setup.

## 1.0.0 - 2026-08-26

- Add a complete UART application for the published `Eccelerators.Cli` core.
- Provide static `help`, `hello`, `bye`, and `echo` command dispatch.
- Preserve CLI input and output through explicit UART backpressure handling.
- Cover prompts, editing, CRLF handling, arguments, errors, and consecutive
  commands with an end-to-end UART integration test.
- Tested the design using Vivado IP for `xc7a35tcpg236-1`.
