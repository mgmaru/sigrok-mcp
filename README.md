# sigrok-mcp

An MCP server that lets you operate logic analyzers and oscilloscopes from AI clients in natural language.
By layering a thin MCP interface on top of [libsigrok](https://sigrok.org/), it brings 200+ measurement instruments within reach of Claude Desktop and other MCP clients.

[日本語版 README はこちら](./README.ja.md)

## What it is

The world of test and measurement has long suffered from tight coupling between hardware and vendor software. Buying an inexpensive logic analyzer or USB scope often means living with dated, OS-locked, or feature-limited applications.

The existing answer is [sigrok](https://sigrok.org/), an open-source project that abstracts 200+ instruments behind a common API and ships PulseView as its GUI.

`sigrok-mcp` adds an AI-accessible entry point on top of that foundation.

```
You  ──→  Claude Desktop  ──→  sigrok-mcp  ──→  libsigrok  ──→  Hardware
```

You describe your intent in natural language. Claude picks the right tools, drives the hardware through libsigrok, and helps interpret the results.

## What it does

The intended toolset:

- `list_devices`: Enumerate connected instruments
- `capture_waveform`: Acquire a waveform with given settings
- `get_waveform_summary`: Extract features (frequency, amplitude, etc.)
- `decode_protocol`: Decode I2C / SPI / UART / and more
- `export_waveform`: Save waveforms to standard formats

A typical session might look like:

> **You**: The I2C signal from this sensor looks off. Can you check it?
>
> **Claude**: I captured Ch0 for one second. As I2C, the traffic decodes to 0x68 → 0x0B → 0xFF, but I see three transactions where ACK is missing. The slave may not be responding correctly, or the wiring is loose. Worth checking the pull-up resistor values.

## Why an MCP server

Building a full GUI is expensive. An MCP server lets us reuse Claude Desktop and other MCP clients as the UI, so we can focus the project entirely on the hardware abstraction layer and tool definitions.

AI integration with measurement instruments is also still rare in the open-source world. There is room to plant a flag here.

## Design principles

LLMs are probabilistic by nature, while measurement demands determinism, reproducibility, and traceability. To avoid that conflict:

- **The LLM only translates intent and orchestrates tools.** It never makes hardware decisions or estimates measurements probabilistically.
- **All commands to hardware are deterministic code.** MCP tool implementations are plain code; the LLM does not interfere with execution.
- **Dangerous operations require human approval.** A safety layer guards against unintended behavior.

## Supported hardware

Anything supported by libsigrok. The first verification targets:

- **Demo driver** (virtual device, no hardware required) for early development
- **Hantek 6022BE** (a few-thousand-yen USB scope, well-tested with sigrok) as the first physical target

## Installation

```bash
pip install sigrok-mcp
```

Add to your Claude Desktop MCP configuration:

```json
{
  "mcpServers": {
    "sigrok": {
      "command": "sigrok-mcp"
    }
  }
}
```

See [INSTALL.md](./INSTALL.md) for details.

## License

GPLv3, inherited from libsigrok.

## Related projects

- [sigrok](https://sigrok.org/) - The instrument abstraction layer this project builds on
- [PulseView](https://sigrok.org/wiki/PulseView) - The official sigrok GUI frontend
- [Model Context Protocol](https://modelcontextprotocol.io/) - The AI tool integration protocol from Anthropic

## Status

Currently in **design phase**. Implementation has not started yet.
See [DESIGN.md](./DESIGN.md) for design discussions and progress.

## Contributing

Welcome. We especially appreciate users who can test on their own hardware. See [CONTRIBUTING.md](./CONTRIBUTING.md) for details (in preparation).
