# Time MCP Server

This standalone fork of the [upstream Time server](https://github.com/modelcontextprotocol/servers/tree/main/src/time)
provides current time, timezone conversion, and elapsed-time queries through MCP.
It detects the host's local timezone by default and adds an explicit UTC boolean.
Python 3.10 or newer is required.

## Installation

Clone this repository and run these commands in its root, beside `pyproject.toml`:

```sh
uv sync --locked --no-dev
uv run --locked --no-dev mcp-server-time
```

uv manages Python and the locked project environment on Windows, macOS, and Linux.
No environment activation is required.

## Tools

| Tool | Arguments |
|---|---|
| `get_current_time` | Optional `timezone` and `utc` |
| `convert_time` | Required `time` in `HH:MM`; optional `source_timezone` and `target_timezone` |
| `get_time_ago` | Optional `timezone`, `utc`, `days_ago`, `hours_ago`, `minutes_ago`, and `seconds_ago` |

Omitting a timezone uses the detected local timezone. `--local-timezone` overrides
that default. Timezone names are exact, case-sensitive IANA keys.

For `get_current_time` and `get_time_ago`, `utc: true` returns UTC and takes
precedence over an explicit timezone or `--local-timezone`. Omit `utc` or pass
`false` to use local or explicitly selected time. Only JSON booleans are accepted.

On an `America/Chicago` host, `get_current_time({})` reports that timezone with
its date-appropriate offset. `get_current_time({"utc": true})` reports `UTC` and
an ISO timestamp ending in `+00:00`.

`get_time_ago` accepts nonnegative integers for its offsets, which add together.
For example, `{"hours_ago": 2, "minutes_ago": 30, "utc": true}` requests two and a
half elapsed hours ago in UTC. A day is 24 elapsed hours, including across daylight
saving changes. Fractions, numeric strings, booleans, and dates before year 1
are rejected. `convert_time` reports both occurrences of an ambiguous daylight-saving
time; a nonexistent wall-clock time returns a tool error.

## Client configuration

Replace `<checkout-directory>` with this clone's absolute path. If the client
cannot locate uv on `PATH`, set `command` to its installed executable.

```json
{
  "mcpServers": {
    "time": {
      "command": "uv",
      "args": ["run", "--locked", "--no-dev", "--directory", "<checkout-directory>", "mcp-server-time"]
    }
  }
}
```

The client starts and stops the default stdio server. MCP 2025-11-25 and
2026-07-28 clients are supported automatically. Tools return structured results
and matching JSON text; failures are readable MCP tool errors.

## Server options

| Option | Meaning |
|---|---|
| `--local-timezone` | Override the detected default with an IANA timezone |
| `--transport` | `stdio` by default; `streamable-http` enables the HTTP transport |

```sh
uv run --locked --no-dev mcp-server-time --transport streamable-http
```

The optional HTTP endpoint is `http://127.0.0.1:3001/mcp` by default.
`HOST` and `PORT` configure the listener. Browser Origins must be localhost
origins; the SDK also checks Host and Origin on loopback.

## Development

```sh
uv sync --locked --all-extras --dev
uv run --locked ruff check .
uv run --locked ruff format --check .
uv run --locked pyright
uv run --locked pytest
uv build
```

Time CI runs on Windows, macOS, and Linux with Python 3.10 and 3.15.
System timezone data is used with the locked `tzdata` fallback.

## Upstream history

Only the upstream `src/time` history is retained. The fork is one commit above
that extracted server baseline; [UPSTREAM.json](UPSTREAM.json) records its source.

## License

MIT. See [LICENSE](LICENSE).
