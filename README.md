# Lagent

For those of us who want to be lazy and build agents.

A simple to use set of abstractions for building AI agents and workflows. This system allows developers to instantiate agents and instruct workflows to complete tasks, with built-in support for memory and file operations.

## Features

- 🤖 Easy agent creation with purpose-specific configurations
- 💭 Built-in memory system using Pinecone for vector storage
- 📝 File system operations and command execution
- 🔄 Batched vector operations for efficient memory management
- 🎯 Purpose-optimized model selection
- 🚀 TypeScript support out of the box

## Prerequisites

- Node.js >= 20.0.0
- A Pinecone account and index
- An OpenAI API key

## Installation

1. Install the dependencies:

```bash
yarn install
```

2. Create a `.env` file in your project root:

```bash
cp .env.example .env
```

## Configuration

Edit your `.env` file with the following required variables:

```env
# OpenAI Configuration
OPENAI_API_KEY="your-openai-api-key"
OPENAI_EMBEDDING_MODEL="text-embedding-ada-002"

# Pinecone Configuration
PINECONE_API_KEY="your-pinecone-api-key"
PINECONE_INDEX_NAME="your-index-name"
```

## Basic Usage

See the [example](./example/index.ts) for a complete example of how to use the system.

Here's a simple example of creating a system with an agent:

```typescript
import { System, AgentConfig, SystemConfig } from './src';

// Configure the system
const systemConfig: SystemConfig = {
  openaiApiKey: process.env.OPENAI_API_KEY!,
  pineconeApiKey: process.env.PINECONE_API_KEY!,
  pineconeIndexName: process.env.PINECONE_INDEX_NAME!,
  embeddingModel: process.env.OPENAI_EMBEDDING_MODEL,
};

// Configure an agent
const agents: AgentConfig[] = [
  {
    name: 'coder',
    purpose: 'coding', // Will use purpose-specific model
    systemPrompt: 'You are a helpful coding assistant.',
    response: {
      type: 'code',
      language: 'typescript',
    },
  },
];

// Create the system and initialize
const system = new System(agents, systemConfig);
await system.initialize();

// Get the agent and use it
const agent = system.getAgent('coder');

// Generate code
const response = await agent.prompt('Create a function that calculates the fibonacci sequence');
console.log(response);

// Save the response to memory
await agent.saveInMemory(response, 'code', {
  topic: 'algorithms',
  difficulty: 'medium',
});

// Search memory
const memories = await agent.searchMemory(
  'fibonacci implementation',
  5, // Number of results
  { topic: 'algorithms' } // Optional filter
);
```

## Multi-Agent System

You can create multiple agents with different purposes to handle various tasks. Here's an example of using multiple agents to create and document code:

```typescript
import { System, AgentConfig, SystemConfig } from './src';

const systemConfig: SystemConfig = {
  openaiApiKey: process.env.OPENAI_API_KEY!,
  pineconeApiKey: process.env.PINECONE_API_KEY!,
  pineconeIndexName: process.env.PINECONE_INDEX_NAME!,
  embeddingModel: process.env.OPENAI_EMBEDDING_MODEL,
};

// Configure multiple agents with different purposes
const agents: AgentConfig[] = [
  {
    name: 'coder',
    purpose: 'coding',
    systemPrompt: 'You are a TypeScript expert focused on writing clean, efficient code.',
    response: {
      type: 'code',
      language: 'typescript',
    },
  },
  {
    name: 'documenter',
    purpose: 'reasoning',
    systemPrompt: 'You are a technical writer who creates clear documentation for code.',
    response: {
      type: 'markdown',
    },
  },
];

const system = new System(agents, systemConfig);
await system.initialize();

// Get both agents
const coder = system.getAgent('coder');
const documenter = system.getAgent('documenter');

// Use coder to create a function
const code = await coder.prompt('Create a function that implements quicksort');
await coder.writeFile('quicksort.ts', code);

// Use documenter to create documentation
const docs = await documenter.prompt(`Create documentation for this code: ${code}`);
await documenter.writeFile('quicksort.md', docs);

// Save both to memory with related metadata
await coder.saveInMemory(code, 'code', {
  algorithm: 'quicksort',
  category: 'sorting',
});

await documenter.saveInMemory(docs, 'markdown', {
  algorithm: 'quicksort',
  type: 'documentation',
});

// Later, you can search either agent's memory
const codeExamples = await coder.searchMemory('quicksort implementation');
const documentation = await documenter.searchMemory('quicksort documentation');
```

Each agent can:

- Have its own purpose and model selection
- Maintain separate memory contexts
- Be optimized for specific tasks
- Share information through file operations
- Have custom system prompts and response types

This allows you to create specialized agents that work together to complete complex tasks, each focusing on their area of expertise.

## Agent Configuration

Agents can be configured with different purposes:

- `coding`: Optimized for code generation
- `reasoning`: For complex problem solving
- `answering`: For general Q&A

Each purpose automatically selects the most appropriate model:

```typescript
const PURPOSE_MODEL_MAP = {
  reasoning: 'gpt-4o',
  answering: 'gpt-4o-mini',
  coding: 'o1',
};
```

## Memory System

The system uses Pinecone for vector storage, automatically handling:

- Content chunking
- Embedding generation
- Batch processing
- Metadata management

Clear the memory store:

```bash
yarn memory:clear
```

## File Operations

Agents can perform file operations:

```typescript
await agent.writeFile('output.ts', response);
await agent.readFile('input.txt');
await agent.appendFile('log.txt', 'New entry');
await agent.createDirectory('new-folder');
```

## Command Execution

Agents can execute system commands:

```typescript
const result = await agent.executeCommand('ls -la');
```

## Architecture

The system is built with maintainability in mind:

- `Agent`: Main class for agent operations
- `System`: Manages multiple agents
- `Adapters`: Clean abstractions for external services
  - `OpenAIAdapter`: Handles all OpenAI operations
  - `PineconeAdapter`: Manages vector operations
- `Actions`: File system and command operations
- `Logger`: Consistent logging across the system

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT
