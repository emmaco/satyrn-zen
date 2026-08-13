# How to run a local model with Ollama

## Overview

This guide explains how to run a small local model using Ollama's CLI.
It is a good starting point for anyone looking to run a local model.

The guide works for any local system since it does not require a GPU
and runs on any operating system (Windows, macOS, Linux).

## Before you start

This guide assumes you have a basic understanding of the terminal.

We recommend starting with a Qwen3.5 model. Select the specific model
based on the available RAM on your local system:

- Qwen3.5:2B for less than 16GB of RAM
- Qwen3.5:4B for 16GB+ of RAM

## Run the Qwen3.5 model

1. Install Ollama.

    Use [Ollama's installation instructions](https://docs.ollama.com/quickstart)
    for your operating system.

1. Run the Ollama CLI from the terminal. Type `ollama run qwen3.5:2b` in the terminal. This command will
   download the model and run it in ollama.

    ```sh
    ollama run qwen3.5:2b
    ```

    You should see a list of available commands.

    ```sh
    ❯ ollama
    Ollama 0.32.5

    ▸ Chat, Code, & Work
        Chat with models, code, search the web, and delegate real work

      Launch Claude Code
        Anthropic's coding tool with subagents

      Launch OpenCode
        Anomaly's open-source coding agent

      Launch Hermes Agent (install)
        Self-improving AI agent built by Nous Research

      Launch OpenClaw (install)
        Personal AI with 100+ skills


    ↑/↓ navigate • enter launch • → configure • esc quit
    ```

1. Select "Chat, Code, & Work" to launch the chat interface.

    You will see this output:

    ```sh
    ❯ ollama
    Select model to run: Type to filter...

      Recommended
      ▸ glm-5.2:cloud (Sign in required)
          Long-horizon coding and agentic engineering with a solid 1M context
        minimax-m3:cloud
          State-of-the-art coding & agent capabilities with multimodal reasoning and
        gemma4:26b
          Agentic workflows and multimodal reasoning, ~19GB, (not downloaded)

      More
        gemma4:12b
        qwen3.5:2b

    ↑/↓ navigate • enter select • ← back
    ```

1. Type `qwen3.5:2b` to use the Qwen3.5:2B model.

    You will see this output:

    ```sh
    ❯ ollama
    ╭──────────────────────────────────────────────────────────────────────────────╮
    │ what changed on this branch?                                                 │
    ╰──────────────────────────────────────────────────────────────────────────────╯
      qwen3.5:2b
    ```

1. Enter a prompt in the text box.

    ```sh
    ❯ ollama
    ╭──────────────────────────────────────────────────────────────────────────────╮
    │ What standard library in Python is used to run asynchronous code             │
    ╰──────────────────────────────────────────────────────────────────────────────╯
      qwen3.5:2b
    ```

1. View the model's response.

    Congratulations! You have successfully run a small model locally on your machine.

1. Optional: Type another prompt.
1. Enter `/bye` to exit the prompt and then `ESC` to quit Ollama.

## Run a local coding agent with Ollama in Docker

The CLI setup above is the quickest way to chat with a model. If you want a
local model that a coding agent can actually drive - reading files, running
commands, writing code - run Ollama in Docker and point a coding agent at it.

This section runs [JetBrains Mellum2](https://ollama.com/JetBrains) in a
container and uses [Pi](https://pi.dev) as the coding agent.

!!! note "Why Mellum2?"
    A coding agent needs the model to emit *structured* tool calls, which
    Ollama's OpenAI-compatible endpoint turns into a real `tool_calls`
    response. Small general-purpose models often print the tool call as chat
    text instead, so the agent has nothing to execute. Mellum2 emits tool
    calls reliably, which makes it usable as an agent on modest hardware.

### Before you start

You need:

- [Docker](https://www.docker.com/) with Compose, to run Ollama.
- [Pi](https://pi.dev), the coding agent that talks to the local model.
- About 16GB of RAM.

Check the [model size requirements](https://ollama.com/JetBrains) before
picking a tag.

### Start Ollama in Docker

1. Create a `docker-compose.yaml` file in an empty directory.

    ```yaml
    services:
      ollama:
        image: ollama/ollama
        container_name: ollama
        ports:
          - "11434:11434"
        volumes:
          - ollama_data:/root/.ollama

    volumes:
      ollama_data:
    ```

    The named `ollama_data` volume keeps downloaded models between restarts,
    so you only pull each model once.

1. Start the container from that directory.

    ```sh
    docker compose up -d
    ```

1. Pull the model into the running container. This is a one-time download.

    ```sh
    docker exec -it ollama ollama pull JetBrains/mellum2-instruct-q4_k_m
    ```

1. Confirm the model is available.

    ```sh
    docker exec -it ollama ollama list
    ```

    You should see `JetBrains/mellum2-instruct-q4_k_m:latest` in the list.

The model is now served on `http://localhost:11434`, with an
OpenAI-compatible API at `http://localhost:11434/v1`.

### Point Pi at the local model

1. Register the local Ollama provider in Pi's global config. This is also a
   one-time step.

    ```sh
    mkdir -p ~/.pi/agent
    ```

    Create `~/.pi/agent/models.json` with the following content:

    ```json
    {
      "providers": {
        "ollama": {
          "baseUrl": "http://localhost:11434/v1",
          "api": "openai-completions",
          "apiKey": "ollama",
          "models": [
            {
              "id": "JetBrains/mellum2-instruct-q4_k_m:latest",
              "name": "Mellum2 Instruct q4_k_m (local)"
            }
          ]
        }
      }
    }
    ```

    Ollama ignores the `apiKey` value, but the field must be present because
    the endpoint is OpenAI-compatible.

1. Launch Pi from the project directory.

    ```sh
    pi
    ```

1. Give the agent a task, such as "list the files in this directory" or
   "create a file called `hello.py` that prints hello".

### Stop and clean up

1. Stop the container when you're done.

    ```sh
    docker compose down
    ```

    Downloaded models stay in the `ollama_data` volume, so the next
    `docker compose up -d` starts without re-downloading.

1. Optional: remove the downloaded models too.

    ```sh
    docker compose down -v
    ```
