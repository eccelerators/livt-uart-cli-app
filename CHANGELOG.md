# Changelog

All notable changes to this package are documented here.

## 1.0.0 - 2026-08-26

- Add a complete UART application for the published `Eccelerators.Cli` core.
- Provide static `help`, `hello`, `bye`, and `echo` command dispatch.
- Preserve CLI input and output through explicit UART backpressure handling.
- Cover prompts, editing, CRLF handling, arguments, errors, and consecutive
  commands with an end-to-end UART integration test.
- Tested the design using Vivado IP for `xc7a35tcpg236-1`.
