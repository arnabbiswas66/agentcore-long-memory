# AgentCore Long Memory Example

This project demonstrates how to use **AWS Bedrock AgentCore Memory** with **LangGraph** to create a nutrition assistant that remembers user preferences across conversations. The assistant uses long-term memory to provide personalized recommendations based on previously captured user preferences.

## Overview

The project contains two implementations of the same functionality:

1. **`agentcore-long-memory-latest-apis.ipynb`** - Uses the latest LangChain and LangGraph APIs
2. **`agentcore-long-memory.ipynb`** - Uses the AWS example implementation as-is (with deprecated APIs)

Both implementations achieve the same core functionality but use different API patterns.

## Functionality

### Core Features

The nutrition assistant provides the following capabilities:

1. **Long-Term Memory Storage**: Uses AWS Bedrock AgentCore Memory to store user preferences persistently across sessions
2. **Custom Memory Strategies**: Implements a custom "NutritionPreferences" strategy that automatically extracts and consolidates user food preferences from conversations
3. **Context-Aware Responses**: Retrieves relevant user preferences before generating responses, allowing the assistant to provide personalized recommendations
4. **Cross-Session Memory**: Remembers user preferences across different conversation sessions (same `actor_id`, different `thread_id`)

### How It Works

1. **Memory Initialization**:
   - Creates or retrieves a Bedrock AgentCore Memory instance
   - Configures a custom memory strategy with extraction and consolidation prompts
   - Sets up namespaces for organizing preferences (`/{actorId}/preferences`)

2. **Pre-Model Hook**:
   - Saves incoming human messages to the memory store
   - Searches for relevant user preferences based on the current message
   - Injects retrieved preferences as context before the LLM processes the message
   - This allows the assistant to reference past preferences in its responses

3. **Post-Model Hook**:
   - Saves the LLM's response to the memory store
   - Triggers the extraction strategy to identify and store new preferences from the conversation

4. **Memory Extraction & Consolidation**:
   - The custom strategy uses Bedrock models to extract food preferences from conversations
   - Preferences are automatically categorized (e.g., "food", "nutrition", "fitness")
   - Consolidation logic determines whether to add new memories, update existing ones, or skip redundant information

### Example Workflow

**First Conversation:**
- User: "I'm cooking salmon with rice and veggies. What can I add to improve protein and vitamins?"
- System extracts preferences: "Enjoys salmon with rice and vegetables", "Seeks high protein and vitamin-rich foods for weightlifting"
- Assistant provides personalized recommendations

**Second Conversation (New Session):**
- User: "What should I make for dinner tonight?"
- System retrieves stored preferences from the first conversation
- Assistant suggests: "Your favorite: Baked salmon with brown rice and roasted vegetables" (referencing the remembered preference)

## Key Components

### Custom Memory Prompts

The `custom_memory_prompts.py` file contains:

- **Extraction Prompt**: Instructs the model to identify explicit and implicit food preferences from conversations
- **Consolidation Prompt**: Determines how to handle new preferences (AddMemory, UpdateMemory, or SkipMemory)

### Memory Store

Uses `AgentCoreMemoryStore` from `langgraph_checkpoint_aws` to:
- Store conversation messages in namespaces organized by `actor_id` and `thread_id`
- Search for relevant preferences using semantic search
- Integrate with Bedrock AgentCore's extraction and consolidation strategies

### Configuration

The system requires:
- **Actor ID**: Identifies the user (e.g., "user-1", "user-2")
- **Thread ID**: Identifies the conversation session (e.g., "session-1", "session-2")
- **Memory Execution Role ARN**: AWS IAM role for Bedrock AgentCore operations
- **Region**: AWS region for Bedrock services

## Implementation Differences

### Latest APIs (`agentcore-long-memory-latest-apis.ipynb`)

- Uses `from langchain.agents import create_agent`
- Uses middleware decorators: `@before_model` and `@after_model`
- Hooks receive `state, runtime: Runtime` parameters
- Accesses context via `runtime.context` with fallback handling
- Uses `middleware=[pre_model_hook, post_model_hook]` parameter

### AWS Example (`agentcore-long-memory.ipynb`)

- Uses `from langgraph.prebuilt import create_react_agent` (deprecated)
- Hooks are plain functions with `state, config: RunnableConfig, *, store: BaseStore` parameters
- Accesses context via `config["configurable"]`
- Uses `pre_model_hook` and `post_model_hook` as direct parameters

## Setup

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Configure AWS credentials and region:
   ```bash
   export AWS_REGION=us-east-1
   export MEMORY_EXECUTION_ROLE_ARN=arn:aws:iam::<account-id>:role/BedrockAgentCoreExecutionRole
   ```

3. Run either notebook to see the assistant in action

## Requirements

- AWS account with Bedrock AgentCore access
- IAM role with permissions for Bedrock AgentCore Memory operations
- Python 3.12+
- Required packages (see `requirements.txt`)

