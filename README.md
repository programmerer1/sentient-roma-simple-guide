# ROMA Quick Start Guide

Undoubtedly, **ROMA** is a very powerful product with a wide range of features.  
However, this guide is intended only for **basic familiarization**, so it will not cover all capabilities.  
Here you will find the simplest way to install and interact with ROMA via the API.

---

## Step 1. Install ROMA

In this example, we will install ROMA using **Docker**.

```bash
git clone https://github.com/sentient-agi/ROMA.git
cd ROMA

./setup.sh --docker
```

Or, alternatively:

```bash
git clone https://github.com/sentient-agi/ROMA.git
cd ROMA
cp .env.example .env
cd docker

docker compose up -d
```

## Step 2. Configure the Environment
Go back to the ROMA root folder and open the .env file.
Add your API keys for the supported services (not all are required).
For example, you can set only OPENROUTER_API_KEY and EXA_API_KEY.

.env file
```bash
# SentientResearchAgent Environment Configuration
# Copy this file to .env and fill in your API keys

# ===== LLM Provider Keys =====
# OpenRouter API key (primary LLM provider)
OPENROUTER_API_KEY=openrouter_api_key

# OpenAI API key (optional - for direct OpenAI usage)
OPENAI_API_KEY=your_openai_key_here

# Google GenAI API key (optional - for Gemini models)
GOOGLE_GENAI_API_KEY=your_google_genai_key_here

# Anthropic API key (optional - for Claude models)
ANTHROPIC_API_KEY=your_anthropic_key_here
..............................
```

Open the .sentient.yaml file and update the configuration:

```bash
llm:
  provider: "openrouter"
  api_key: "${OPENROUTER_API_KEY}"
```
⚠️ Important Note
By default, ROMA uses paid models and processes requests quite aggressively, which may quickly spend your credits on services such as OpenRouter.

While testing, it is recommended to switch all agents to free models.
To do this, edit the following file:
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml
Example configuration:

```bash
- name: "CoreResearchPlanner"
  type: "planner"
  adapter_class: "PlannerAdapter"
  description: "Specialized planner for research-focused tasks"
  model:
    provider: "litellm"
    model_id: "openrouter/deepseek/deepseek-chat-v3.1:free"
```
⚠️ Note: In the free version, some limitations apply.
This repository also includes my own configuration, so you can reuse it if you wish.

Step 3. Restart Docker with New Settings
After applying the changes, restart the containers:

```bash
cd docker

docker compose -f docker-compose.yml down
docker compose -f docker-compose.yml up -d
```
