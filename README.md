# waleed-agent

A lightweight, single-binary capability bridge that gives any local LLM safe access to a filesystem and terminal acess to a workspace in your filesystem through an OpenAI-compatible API.

No bloated agent framework.  
No vector database.  
No orchestration maze.  
Just tools.

---

# What is this?

`waleed-agent` is a tiny local agent runtime designed to give existing LLMs “hands and feet”.

It sits between:
- your favorite chat frontend
- your preferred LLM
- and your local workspace

The agent exposes an OpenAI-compatible endpoint (`/v1/chat/completions`) that any compatible UI or application can connect to.

Under the hood it:
- forwards requests to your upstream LLM
- executes approved local tools
- safely restricts access to a jailed workspace
- returns results back to the client as if the model had native local capabilities

The goal is simple:

> Give local LLMs safe workspace access without the bloat of modern agent frameworks (Hence the Digital Hands and Feet). 

---

# Philosophy

Most AI agent projects try to become:
- an IDE
- an orchestration engine
- a workflow platform
- a memory system
- a cloud service
- an autonomous ecosystem

`waleed-agent` intentionally does not.

This project focuses on one thing only:

> Extending an LLM with controlled local access capabilities.

The intelligence remains in the model.  
The frontend remains your choice.  
The agent simply provides secure execution capabilities.

Think of it as prosthetics for your LLM — not a replacement brain.

---

# Why I Built This

I wanted:
- a tiny standalone binary
- no Docker
- no Node.js dependency forest
- no Python environments
- no bundled frontend
- no hidden orchestration layers

I already had:
- local models
- OpenAI-compatible APIs
- chat frontends I liked

I just needed a clean way to give models controlled access to:
- files
- directories
- terminal execution

So I built `waleed-agent`.

---

# Features

- Single portable binary
- OpenAI-compatible API
- Workspace-jail enforcement
- File operations
- Terminal execution
- Streaming support
- Minimal dependencies
- Cross-platform architecture
- Frontend agnostic
- Model agnostic
- MIT licensed

---

# Security Model

Security is the defining priority of this project.

All operations are designed around strict workspace containment.

## Current protections 
`(All configurable in the config file, I intentially did not hardcore any possible configurations for extreme flixablity).`

- Filesystem jail enforcement
- Rejection of absolute paths
- Rejection of `..` traversal
- Configurable command restrictions
- Command timeouts
- Output size limits
- Environment variable whitelisting
- API keys are never logged

## Important Warning

`waleed-agent` is currently experimental software.

While significant effort is being placed into security and sandboxing, this project should currently be treated as:

> Trusted local tooling for controlled environments.

- Do NOT expose it directly to the internet.  
- Do NOT use it with untrusted prompts or users.  
- Security hardening is ongoing and remains the primary focus before a stable `v1.0`.
- Please take your time reading and configuring the config file, especially the allowed and denied commands, this is the most important feature and every system and usecase is unique so i did not hardcode force anything that's why `waleed-agent` will not run without a config file beside it.
- You can add waleed-agent to your shell / system variables to call it from anywhere in any folder/directory

---

# Architecture

```text
Frontend/UI
    ↓
waleed-agent
    ↓
Local or Remote LLM API
```

## Quickstart

Building from source:

1.  **Download/Build:**
    ```bash
    go build -o waleed-agent ./cmd/waleed-agent
    ```
2.  **Run:**
    ```bash
    ./waleed-agent
    ```
3.  **Connect:**
    Connect your chat UI to `http://localhost:8484/v1/chat/completions`.

Using pre-compiled release binaries:

- Just unzip it, move it and it's config file to any folder/directory you want and run it!! Dead simple.

## Configuration

Configuration is managed via a hierarchy:
1.  `--config` CLI flag
2.  `waleed-agent.json` in the workspace directory
3.  `waleed-agent.default.json` in the binary directory
4.  Current Working Directory (CWD)

For a complete list of all options, refer to the [TECHSPEC.md](TECHSPEC.md).
