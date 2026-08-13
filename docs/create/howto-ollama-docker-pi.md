# How to run a local coding agent with Ollama in Docker

## Overview

Running Ollama from the CLI is the quickest way to chat with a model. If you
want a local model that a coding agent can actually drive - reading files,
running commands, writing code - run Ollama in Docker and point a coding agent
at it.

This guide runs [JetBrains Mellum2](https://ollama.com/JetBrains) in a
container and uses [Pi](https://pi.dev) as the coding agent.

!!! note "Why Mellum2?"
    A coding agent needs the model to emit *structured* tool calls, which
    Ollama's OpenAI-compatible endpoint turns into a real `tool_calls`
    response. Mellum2 emits tool calls reliably, which makes it usable
    as an agent on modest hardware.

## Before you start

If you are new to Ollama, start with
[How to run a local model with Ollama](howto-ollama.md).

You need:

- [Docker](https://www.docker.com/) with Compose, to run Ollama.
- [Pi](https://pi.dev), the coding agent that talks to the local model.
- About 16GB of RAM.

Check the [model size requirements](https://ollama.com/JetBrains) before
picking a tag.

## Start Ollama in Docker

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

## Point Pi at the local model

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

## Stop and clean up

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
