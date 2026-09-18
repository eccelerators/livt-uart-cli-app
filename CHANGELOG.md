# Changelog

All notable changes to this package are documented here.

## Unreleased

- Migrate to `Livt.IO 1.2.0-dev` and `Eccelerators.Cli 1.1.0`.
- Use `BufferedUart` with acceptance-based transmit and receive operations.
- Update the interactive UART regression for context-derived timing and the
  current buffered API, including empty-read and receive-error checks.

## 1.0.0 - 2026-08-26

- Add a complete UART application for the published `Eccelerators.Cli` core.
- Provide static `help`, `hello`, `bye`, and `echo` command dispatch.
- Preserve CLI input and output through explicit UART backpressure handling.
- Cover prompts, editing, CRLF handling, arguments, errors, and consecutive
  commands with an end-to-end UART integration test.
- Tested the design using Vivado IP for `xc7a35tcpg236-1`.
