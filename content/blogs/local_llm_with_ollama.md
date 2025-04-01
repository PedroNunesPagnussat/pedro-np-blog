---
title: "How to Run Local LLMs with Ollama"
date: 2025-03-31
author: "Pedro Nunes Pagnussat"
tags: ["LLM", "AI", "Chat Bot"]
description: "A guide on how to run local LLMs using Ollama, covering the basics, chat MVP, and Ollama API usage."
---

Running local LLMs with Ollama is an exciting way to bring powerful AI models directly on your hardware, enhancing privacy, network dependancy, and reducing development costs.

## Why Run Local LLMs?

- **Privacy:** Your data stays entirely on your local machine, crucial for sensitive information.
- **Speed and Availability:** No network latency or Availability problems 
- **Pricing:** Enhancing privacy, reducing network dependency, and lowering development costs

## Getting Started with Ollama

### Download

**Linux:**

```sh
curl -fsSL https://ollama.com/install.sh | sh
```

**Windows:** Visit the [Ollama download page](https://ollama.com/download/windows) and download the installer (`.exe`).

**MacOS** Visit the [Ollama download page](https://ollama.com/download/mac) and download the installer (`.app`)


### Pulling Models

To download a model:
```
ollama pull <MODEL>
```

Example with Llama2:
```
ollama pull llama2
```

### Running Models

Run a model directly from your terminal:
```
ollama run <MODEL>
```

Example:
```
ollama run llama2
```

Interact with your model by typing prompts. To exit, simply type:
```
/bye
```

### Saving and Loading chats

Ollama provides simple commands to save and load chat histories. You can save a chat session directly from your terminal by running when in a chat:

```
/save chat-session-name
```

Later, load the saved session to continue your conversation:

```
/load chat-session-name
```

This makes it easy to pick up conversations right where you left off.

## Using the Ollama API

Create a simple terminal-based chat application:
```python
import json

import requests

# Initialize chat history
messages = [{"role": "system", "content": "You're my assistant named Jarvis. I'm Tony"}]

while True:
    try:
        user_input = input("\nYou: ")
    except KeyboardInterrupt:
        print("\nGoodbye!")
        break

    if user_input.lower() in ["exit", "quit"]:
        print("Ending chat...")
        break

    # Add user message to history
    messages.append({"role": "user", "content": user_input})

    # Stream the assistant's response
    print("Jarvis: ", end="", flush=True)
    with requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": "llama2",
            "prompt": "\n".join([f"{m['role']}: {m['content']}" for m in messages]),
            "stream": True,
        },
        stream=True,
    ) as response:
        response.raise_for_status()
        full_response = ""
        for line in response.iter_lines():
            if line:
                token = json.loads(line.decode("utf-8"))
                chunk = token.get("response", "")
                if chunk:
                    full_response += chunk
                    print(chunk, end="", flush=True)
                if token.get("done"):
                    break

    # Add assistant response to history
    messages.append({"role": "assistant", "content": full_response})
```

```
You: Hello Jarvis
Jarvis: Hello, Mr. Stark! *adjusts glasses* It's a pleasure to assist you as always. Is there something you need help with today?
You:   
```
## Using Ollama python package:

**Note:** Before running the Python script below, ensure you've installed Ollama:
```
pip install ollama
```

```python
import ollama

MODEL_NAME = "llama2"
ollama.pull(MODEL_NAME)

SYSTEM_PROMPT = "You're my assistant named Jarvis. I'm Tony"
USER_PROMPT = "Hello, how are you?"

messages = [
    {"role": "system", "content": SYSTEM_PROMPT},
    {"role": "user", "content": USER_PROMPT},
]


print("Jarvis: ", end="", flush=True)

response = ollama.chat(model=MODEL_NAME, messages=messages, stream=True)

# Print response chunks progressively
full_response = []
for chunk in response:
    content = chunk.get("message", {}).get("content", "")
    if content:
        print(content, end="", flush=True)
        full_response.append(content)

print("\n")  # Add final newlines
```

**Example Output:**
```
Jarvis:
Ah, good day to you, sir! *adjusts monocle* I am functioning within normal parameters, thank you for inquiring. How may I assist you today? Is there something in particular you require, or perhaps a task you'd like me to attend to? 😊
```

## Pulling other models

You can leverage advanced reasoning models like DeepSeek to perform complex tasks and even orchestrate calls to more powerful external models, such as GPT-4. This approach allows combining the speed and privacy of local models with the advanced capabilities of external APIs.

Pull a reasoning model with:

```
ollama pull deepseek-r1:7b
```

Here's an example demonstrating how to use DeepSeek locally to make API calls to GPT-4 for tasks that require advanced reasoning:

```python
import ollama
import openai

# Get initial response from local model
ollama.pull("deepseek-r1:7b")
reasoning_response = ollama.generate(
    "deepseek-r1:7b",
    prompt="Analyze and explain the core reasoning steps for: [Your Complex Issue Here]",
)

reasoning_response = reasoning_response["response"].split("</think>")[0].strip()

api_response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[
        {"role": "system", "content": "Expand only on these reasoning elements:"},
        {"role": "user", "content": reasoning_response},
    ],
)

print(api_response)
```

This approach optimally leverages local and remote AI resources.

Running LLMs locally is a game-changer for privacy, speed, and cost. Whether you're just tinkering or building serious tools, Ollama makes it approachable. If you build something cool, let me know—I'd love to see it!