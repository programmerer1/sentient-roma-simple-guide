# ROMA Quick Start Guide

Undoubtedly, **ROMA** https://github.com/sentient-agi/ROMA is a very powerful product with a wide range of features.  
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
This repository also includes my own configuration (agents.yaml and sentient.yaml), so you can reuse it if you wish.

## Step 3. Restart Docker with New Settings
After applying the changes, restart the containers:

```bash
cd docker

docker compose -f docker-compose.yml down
docker compose -f docker-compose.yml up -d
```

So, after installing and running the Docker container, we have two types of services available. The frontend, where we can interact with ROMA through the web interface at http://localhost:3000, and the backend, the system we will interact with directly via the API at http://localhost:5000

You can make API requests using any method that is convenient for you—Curl, custom scripts, or Postman.
So, to begin, we register our request by creating a project. In the request, we will send the query prompt (goal) and the settings.

Our request:
```bash
{
    "goal": "What is recursion?", // Agent's goal at startup
    "config": {
        "profile": {
            "name": "general_agent",       // Internal profile name of the agent
            "displayName": "General Agent" // Display name (for UI)
        },
        "llm": {
            "provider": "litellm",                               // LLM provider (here LiteLLM proxy)
            "model": "openrouter/deepseek/deepseek-chat-v3.1:free", // Model being used
            "temperature": 0.0,                                  // Temperature (0 = strict, 1 = more creative)
            "timeout": 30,                                       // Request timeout to the model (in seconds)
            "max_retries": 3                                     // Number of retries on error
        },
        "execution": {
            "force_root_node_planning": false, // Force building a plan before execution (false = do not force)
            "rate_limit_rpm": 5,              // API request limit (requests per minute)
            "skip_atomization": true,         // Whether to skip splitting the task into subtasks
            "max_concurrent_nodes": 6,        // Maximum number of parallel steps (nodes)
            "max_execution_steps": 3,         // Maximum number of execution steps (prevents overusing credits)
            "max_recursion_depth": 1,         // Limit for nested task depth
            "task_timeout_seconds": 100,      // Timeout for the whole task (in seconds)
            "enable_hitl": false,             // Enable Human-In-The-Loop (manual intervention)
            "hitl_root_plan_only": false,     // HITL only for the root plan
            "hitl_timeout_seconds": 300,      // Timeout for waiting for human intervention
            "hitl_after_plan_generation": false, // Ask human after plan generation
            "hitl_after_modified_plan": false,   // Ask human after plan modification
            "hitl_after_atomizer": false,        // Ask human after task atomization
            "hitl_before_execute": false,        // Ask human before executing a step
            "state_batch_size": 5,            // Batch size for state updates
            "ws_batch_size": 5                // Batch size for WebSocket responses
        },
        "cache": {
            "enabled": true,         // Enable caching
            "ttl_seconds": 3600,     // Cache lifetime (1 hour)
            "max_size": 500,         // Maximum number of elements in cache
            "cache_type": "memory"   // Cache type (memory = RAM)
        },
        "project": {
            "goal": "What is recursion?",  // Project goal (duplicates the top-level goal)
            "max_steps": 3                 // Step limit for the project
        }
    },
    "max_steps": 3 // Global step limit (duplicates project.max_steps)
}
```

