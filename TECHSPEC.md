# Technical Specification (TECHSPEC.md)

## Overview

`waleed-agent` is a lightweight, single-binary agent designed to act as a "tool-calling bridge" between a local LLM (via OpenAI-compatible API) and a local environment (filesystem and terminal). It allows an LLM to execute actions on the host system in a controlled, jailed environment.

## Quick Config Summary

| Category | Key Options | Default Value | Description |
| :--- | :--- | :--- | :--- |
| **Upstream** | `provider` | `openai` | LLM API provider (openai|anthropic). |
| | `baseURL` | `http://localhost:11434/v1` | Base URL for the LLM API. |
| | `model` | `llama2` | The model to use. |
| **Server** | `port` | `8484` | The port the HTTP server listens on. |
| | `host` | `127.0.0.1` | The network interface the server binds to. |
| **Workspace** | `path` | `null` (CWD) | The absolute path of the workspace. Defaults to CWD if not specified. |
| | `jail` | `true` | Boolean to enforce filesystem and terminal sandboxing. |
| **Tools** | `terminal.enabled` | `true` | Enables/disables terminal execution. |
| | `filesystem.enabled` | `true` | Enables/disables filesystem operations. |
| **Agent** | `maxIterations` | `5` | Maximum number of tool-call iterations before stopping. |

## Complete Config Reference

### 1. Upstream Configuration (`upstream`)

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `provider` | `string` | `openai` | Specifies the LLM provider (`openai` or `anthropic`). |
| `apiKey` | `string` | `null` | API key for the upstream provider. |
| `baseURL` | `string` | `http://localhost:11434/v1` | The base URL for the LLM API. |
| `model` | `string` | `llama2` | The model identifier to use. |
| `maxTokens` | `int` | `4096` | Maximum number of tokens the LLM can generate. |
| `temperature` | `float64` | `0.7` | Controls randomness. |
| `topP` | `float64` | `0.9` | Nucleus sampling parameter. |
| `frequencyPenalty` | `float64` | `0.0` | Penalty for repeating tokens. |
| `presencePenalty` | `float64` | `0.0` | Penalty for tokens that have appeared. |
| `stop` | `array` | `[]` | Array of strings that, if generated, will stop the response. |
| `seed` | `int` | `null` | Random seed for reproducibility. |
| `reasoningEffort` | `string` | `medium` | Controls the depth of internal reasoning. |
| `responseFormat` | `object` | `null` | Defines the expected JSON output format. |
| `streamOptions` | `object` | `{chunkSize: 1024}` | Options related to SSE chunking. |
| `extraHeaders` | `object` | `{}` | Custom HTTP headers to send to the upstream API. |

### 2. Server Configuration (`server`)

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `port` | `int` | `8484` | The port the HTTP server listens on. |
| `host` | `string` | `127.0.0.1` | The IP address the server binds to. |
| `apiKey` | `string` | `null` | Optional server-side API key for authentication. |
| `logLevel` | `string` | `text` | Logging verbosity (`text` or `json`). |
| `tls.enabled` | `bool` | `false` | Enables TLS/SSL for secure communication. |
| `tls.certFile` | `string` | `null` | Path to the TLS certificate file. |
| `tls.keyFile` | `string` | `null` | Path to the TLS private key file. |

### 3. Workspace Configuration (`workspace`)

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `path` | `string` | `null` | Absolute path to the workspace. If `null`, CWD is used. |
| `jail` | `bool` | `true` | If true, all file/terminal operations are restricted to the workspace path. |

### 4. Tools Configuration (`tools`)

#### Filesystem Tools (`tools.filesystem`)
| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `enabled` | `bool` | `true` | Enables/disables filesystem tools. |
| `confirmDestructive` | `bool` | `true` | Requires confirmation for destructive actions (e.g., delete, overwrite). |
| `maxFileSizeBytes` | `int` | `10485760` | Maximum size of files that can be operated on. |
| `allowedExtensions` | `array` | `[]` | List of file extensions allowed for operations. |
| `allowedPaths` | `array` | `null` | List of glob patterns defining allowed paths within the workspace. |

#### Terminal Tools (`tools.terminal`)
| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `enabled` | `bool` | `true` | Enables/disables terminal execution. |
| `allowedCommands` | `array` | `["ls", "cd"]` | List of commands permitted to run. |
| `deniedCommands` | `array` | `[]` | List of commands that are strictly forbidden. |
| `timeoutMs` | `int` | `5000` | Timeout in milliseconds for a single command execution. |
| `maxOutputBytes` | `int` | `1048576` | Maximum bytes of output captured from a command. |
| `shell` | `string` | `auto` | The shell to use (`auto`, `bash`, `sh`, `cmd`, `powershell`). |
| `envWhitelist` | `array` | `[]` | List of environment variables allowed in the execution context. |
| `workingDirPolicy` | `string` | `workspace` | Defines how the working directory is determined (`workspace` or `subdir`). |

### 5. Agent Configuration (`agent`)

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `maxIterations` | `int` | `5` | The maximum number of tool-call/LLM interaction cycles. |
| `streamToolProgress` | `bool` | `true` | If true, tool execution results are streamed back to the client. |
| `continueOnToolError` | `bool` | `true` | If true, the agent attempts to continue even if a tool execution fails. |

### 6. Logging Configuration (`logging`)

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `enabled` | `bool` | `true` | Enables/disables file logging. |
| `dir` | `string` | `logs` | Subdirectory inside the workspace where logs are stored. |

## Environment Variable Overrides

The following environment variables can override configuration settings:
*   `AGENT_WORKSPACE`: Overrides the workspace path.
*   `AGENT_UPSTREAM_API_KEY`: Overrides the upstream API key.
*   `AGENT_SERVER_PORT`: Overrides the server port.
*   `AGENT_SERVER_API_KEY`: Overrides the server API key.
*   `AGENT_LOG_LEVEL`: Overrides the logging level.

## CLI Flags

The following flags can be used to override configuration:
*   `--workspace`: Sets the workspace path.
*   `--config`: Specifies an alternative configuration file path.
*   `--port`: Sets the server port.

## API Endpoints

### GET `/v1/models`
**Purpose:** Returns a static list of supported models for UI compatibility.
**Response:**
```json
{
  "object": "list",
  "data": [
    {"id": "llama2", "object": "model"},
    {"id": "mistral", "object": "model"},
    {"id": "gpt-4o", "object": "model"}
  ]
}
```

### POST `/v1/chat/completions`
**Purpose:** The main endpoint for receiving conversation history and generating responses.
**Request Body (Example):**
```json
{
  "model": "llama2",
  "messages": [
    {"role": "system", "content": "You are a helpful agent."},
    {"role": "user", "content": "What is the current date?"}
  ]
}
```
**Response Body (Non-Streaming Example):**
```json
{
  "id": "chatcmpl-dummy",
  "object": "chat.completion",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Agent is ready to process requests."
      }
    }
  ]
}
```
**Streaming Format (SSE):**
The agent supports Server-Sent Events (SSE) with reasoning deltas. The client will receive chunks containing:
*   `content_block_start`
*   `content_block_delta` (containing `text_delta`, `thinking_delta`, `input_json_delta`)
*   `content_block_stop`
*   `message_stop`

## Tool Definitions (JSON Schemas)

The following tools are available and are executed within the jailed workspace:

### `read_file(path)`
**Description:** Returns file contents as JSON.
**Schema:** `{"type": "object", "properties": {"path": {"type": "string"}}}`

### `write_file(path, content)`
**Description:** Creates parent directories and overwrites the file.
**Schema:** `{"type": "object", "properties": {"path": {"type": "string"}, "content": {"type": "string"}}}`

### `edit_file(path, old_string, new_string)`
**Description:** Replaces the first occurrence of `old_string` with `new_string`.
**Schema:** `{"type": "object", "properties": {"path": {"type": "string"}, "old_string": {"type": "string"}, "new_string": {"type": "string"}}}`

### `list_directory(path)`
**Description:** Returns directory entries.
**Schema:** `{"type": "object", "properties": {"path": {"type": "string"}}}`

### `create_directory(path)`
**Description:** Creates a directory.
**Schema:** `{"type": "object", "properties": {"path": {"type": "string"}}}`

### `delete_file(path)`
**Description:** Deletes a file.
**Schema:** `{"type": "object", "properties": {"path": {"type": "string"}}}`

### `run_terminal_command(command, working_dir?)`
**Description:** Runs a shell command.
**Schema:** `{"type": "object", "properties": {"command": {"type": "string"}, "working_dir": {"type": "string"}}}`

## Security Considerations

*   **Jail Enforcement:** All file and terminal operations are strictly contained within the configured workspace path.
*   **Path Validation:** The agent rejects absolute paths and `..` traversal attempts to prevent sandbox escape.
*   **Rate Limiting:** Terminal commands are subject to configurable timeouts and output size limits.
*   **Logging:** Sensitive information, particularly upstream API keys, is never included in logs.

## Building and Cross-Compilation

**Module Name:** `waleed-agent`
**Entry Point:** `cmd/waleed-agent/main.go`

**Building (Linux/macOS):**
```bash
go build -o waleed-agent ./cmd/waleed-agent
```

**Cross-Compilation Examples:**
*   **Linux:** `GOOS=linux GOARCH=amd64 go build -o waleed-agent ./cmd/waleed-agent`
*   **Windows:** `GOOS=windows GOARCH=amd64 go build -o waleed-agent.exe ./cmd/waleed-agent`

## Roadmap / Unimplemented Features

*   **Full SSE Streaming:** Currently simulated; full implementation of Server-Sent Events is pending.
*   **Anthropic API Integration:** Full message format conversion and SSE parsing for Anthropic is pending.
*   **CLI Argument Parsing:** Full implementation of `--workspace` and other CLI flags is pending.
*   **`edit_file` Tool:** The `edit_file` tool is defined in the schema but not yet implemented in the tool executor.
