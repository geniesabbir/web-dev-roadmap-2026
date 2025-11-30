# Day 2: Anthropic API - Building with Claude

## Introduction

Anthropic's Claude is a powerful AI assistant known for its helpfulness, harmlessness, and honesty. Claude excels at nuanced conversations, complex analysis, coding tasks, and following detailed instructions. Today, you'll learn how to integrate Claude into your applications using the Anthropic API.

## Learning Objectives

By the end of this lesson, you will be able to:
- Set up and authenticate with the Anthropic API
- Make messages API requests
- Implement streaming responses
- Use Claude's tool use (function calling)
- Compare Claude with OpenAI for different use cases

---

## Getting Started

### Setup

```bash
# Install the Anthropic SDK
npm install @anthropic-ai/sdk

# Store your API key securely
# .env.local
ANTHROPIC_API_KEY=sk-ant-...
```

### Basic Configuration

```javascript
// lib/anthropic.js
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export default anthropic;
```

---

## Messages API

### Basic Request

```javascript
import anthropic from '@/lib/anthropic';

async function chat(message) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [
      {
        role: 'user',
        content: message
      }
    ]
  });

  return response.content[0].text;
}

// Usage
const answer = await chat('Explain quantum computing in simple terms.');
```

### System Prompts

```javascript
// System prompt is a separate parameter in Anthropic API
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 1024,
  system: `You are an expert software architect with 20 years of experience.
           You provide detailed, practical advice on system design.
           Always consider scalability, maintainability, and security.
           Use examples and diagrams (in ASCII) when helpful.`,
  messages: [
    {
      role: 'user',
      content: 'How should I design a real-time notification system?'
    }
  ]
});
```

### Multi-Turn Conversations

```javascript
// Managing conversation history
class ClaudeConversation {
  constructor(systemPrompt) {
    this.system = systemPrompt;
    this.messages = [];
  }

  async send(userMessage) {
    // Add user message
    this.messages.push({
      role: 'user',
      content: userMessage
    });

    // Create completion
    const response = await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2048,
      system: this.system,
      messages: this.messages,
    });

    // Extract assistant response
    const assistantMessage = response.content[0].text;

    // Add to history
    this.messages.push({
      role: 'assistant',
      content: assistantMessage
    });

    return {
      content: assistantMessage,
      usage: response.usage,
      stopReason: response.stop_reason
    };
  }

  // Trim history to manage context length
  trimHistory(maxMessages = 20) {
    if (this.messages.length > maxMessages) {
      this.messages = this.messages.slice(-maxMessages);
    }
  }

  clear() {
    this.messages = [];
  }
}

// Usage
const chat = new ClaudeConversation(
  'You are a helpful coding assistant specializing in React and TypeScript.'
);

const response1 = await chat.send('How do I create a custom hook?');
const response2 = await chat.send('Can you add error handling to that?');
```

---

## Model Selection

```javascript
// Available Claude models
const CLAUDE_MODELS = {
  // Most capable, best for complex tasks
  'claude-sonnet-4-20250514': {
    name: 'Claude Sonnet 4',
    inputCost: 0.003,    // per 1K tokens
    outputCost: 0.015,
    contextWindow: 200000,
    bestFor: ['complex reasoning', 'coding', 'analysis']
  },

  // Fast and efficient
  'claude-3-5-haiku-20241022': {
    name: 'Claude 3.5 Haiku',
    inputCost: 0.0008,
    outputCost: 0.004,
    contextWindow: 200000,
    bestFor: ['simple tasks', 'high volume', 'quick responses']
  },

  // Previous generation (still available)
  'claude-3-opus-20240229': {
    name: 'Claude 3 Opus',
    inputCost: 0.015,
    outputCost: 0.075,
    contextWindow: 200000,
    bestFor: ['most complex tasks', 'research']
  }
};

// Choose model based on task
function selectModel(task) {
  switch (task.type) {
    case 'simple':
    case 'classification':
    case 'extraction':
      return 'claude-3-5-haiku-20241022';

    case 'coding':
    case 'analysis':
    case 'writing':
      return 'claude-sonnet-4-20250514';

    case 'research':
    case 'complex-reasoning':
      return 'claude-3-opus-20240229';

    default:
      return 'claude-sonnet-4-20250514';
  }
}
```

---

## Streaming Responses

### Server-Side Streaming

```javascript
// Stream Claude responses
async function streamChat(message) {
  const stream = await anthropic.messages.stream({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [{ role: 'user', content: message }]
  });

  for await (const event of stream) {
    if (event.type === 'content_block_delta' &&
        event.delta.type === 'text_delta') {
      process.stdout.write(event.delta.text);
    }
  }

  // Get final message after stream completes
  const finalMessage = await stream.finalMessage();
  return finalMessage;
}

// Next.js API route with streaming
// app/api/claude/route.js
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic();

export async function POST(req) {
  const { messages, system } = await req.json();

  const stream = await anthropic.messages.stream({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 2048,
    system: system || 'You are a helpful assistant.',
    messages
  });

  // Create readable stream for response
  const encoder = new TextEncoder();

  const readable = new ReadableStream({
    async start(controller) {
      for await (const event of stream) {
        if (event.type === 'content_block_delta' &&
            event.delta.type === 'text_delta') {
          controller.enqueue(encoder.encode(event.delta.text));
        }
      }
      controller.close();
    }
  });

  return new Response(readable, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
    }
  });
}
```

### Stream Event Types

```javascript
// Understanding stream events
const stream = await anthropic.messages.stream({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Hello!' }]
});

for await (const event of stream) {
  switch (event.type) {
    case 'message_start':
      // Initial message metadata
      console.log('Message ID:', event.message.id);
      console.log('Model:', event.message.model);
      break;

    case 'content_block_start':
      // Start of a content block
      console.log('Content block started:', event.content_block.type);
      break;

    case 'content_block_delta':
      // Incremental content
      if (event.delta.type === 'text_delta') {
        process.stdout.write(event.delta.text);
      }
      break;

    case 'content_block_stop':
      // Content block finished
      console.log('\nContent block finished');
      break;

    case 'message_delta':
      // Message-level changes
      console.log('Stop reason:', event.delta.stop_reason);
      console.log('Output tokens:', event.usage.output_tokens);
      break;

    case 'message_stop':
      // Message complete
      console.log('Message complete');
      break;
  }
}
```

---

## Tool Use (Function Calling)

### Defining Tools

```javascript
// Claude's tool use system
const tools = [
  {
    name: 'get_weather',
    description: 'Get the current weather in a given location. Call this whenever the user asks about weather conditions.',
    input_schema: {
      type: 'object',
      properties: {
        location: {
          type: 'string',
          description: 'The city and country, e.g., "London, UK"'
        },
        unit: {
          type: 'string',
          enum: ['celsius', 'fahrenheit'],
          description: 'The temperature unit to use'
        }
      },
      required: ['location']
    }
  },
  {
    name: 'search_database',
    description: 'Search for products in the database',
    input_schema: {
      type: 'object',
      properties: {
        query: {
          type: 'string',
          description: 'The search query'
        },
        category: {
          type: 'string',
          description: 'Optional category filter'
        },
        limit: {
          type: 'integer',
          description: 'Maximum number of results',
          default: 10
        }
      },
      required: ['query']
    }
  },
  {
    name: 'send_email',
    description: 'Send an email to a recipient',
    input_schema: {
      type: 'object',
      properties: {
        to: {
          type: 'string',
          description: 'Email recipient address'
        },
        subject: {
          type: 'string',
          description: 'Email subject line'
        },
        body: {
          type: 'string',
          description: 'Email body content'
        }
      },
      required: ['to', 'subject', 'body']
    }
  }
];
```

### Handling Tool Calls

```javascript
// Complete tool use flow
async function chatWithTools(userMessage) {
  const messages = [{ role: 'user', content: userMessage }];

  // First API call
  let response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    tools,
    messages
  });

  // Keep processing until no more tool calls
  while (response.stop_reason === 'tool_use') {
    // Find tool use blocks
    const toolUseBlocks = response.content.filter(
      block => block.type === 'tool_use'
    );

    // Add assistant's response (including tool calls)
    messages.push({
      role: 'assistant',
      content: response.content
    });

    // Process each tool call and collect results
    const toolResults = [];

    for (const toolUse of toolUseBlocks) {
      console.log(`Calling tool: ${toolUse.name}`);
      console.log('Input:', toolUse.input);

      // Execute the tool
      const result = await executeToolCall(toolUse.name, toolUse.input);

      toolResults.push({
        type: 'tool_result',
        tool_use_id: toolUse.id,
        content: JSON.stringify(result)
      });
    }

    // Add tool results to messages
    messages.push({
      role: 'user',
      content: toolResults
    });

    // Get next response
    response = await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1024,
      tools,
      messages
    });
  }

  // Return final text response
  const textBlock = response.content.find(block => block.type === 'text');
  return textBlock?.text || '';
}

// Tool implementations
async function executeToolCall(name, input) {
  switch (name) {
    case 'get_weather':
      return await fetchWeather(input.location, input.unit);

    case 'search_database':
      return await searchProducts(input.query, input.category, input.limit);

    case 'send_email':
      return await sendEmail(input.to, input.subject, input.body);

    default:
      return { error: `Unknown tool: ${name}` };
  }
}

// Example usage
const response = await chatWithTools(
  "What's the weather in Paris? Also search for umbrellas in our store."
);
```

---

## Vision (Image Analysis)

```javascript
// Claude can analyze images
async function analyzeImage(imageUrl, question) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [
      {
        role: 'user',
        content: [
          {
            type: 'image',
            source: {
              type: 'url',
              url: imageUrl
            }
          },
          {
            type: 'text',
            text: question
          }
        ]
      }
    ]
  });

  return response.content[0].text;
}

// With base64 image
async function analyzeBase64Image(base64Data, mediaType, question) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [
      {
        role: 'user',
        content: [
          {
            type: 'image',
            source: {
              type: 'base64',
              media_type: mediaType, // 'image/jpeg', 'image/png', etc.
              data: base64Data
            }
          },
          {
            type: 'text',
            text: question
          }
        ]
      }
    ]
  });

  return response.content[0].text;
}

// Usage examples
const description = await analyzeImage(
  'https://example.com/product.jpg',
  'Describe this product and identify any text visible in the image.'
);

// Multiple images
const comparison = await anthropic.messages.create({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 1024,
  messages: [
    {
      role: 'user',
      content: [
        { type: 'image', source: { type: 'url', url: 'https://example.com/before.jpg' } },
        { type: 'image', source: { type: 'url', url: 'https://example.com/after.jpg' } },
        { type: 'text', text: 'Compare these two images and describe the differences.' }
      ]
    }
  ]
});
```

---

## Error Handling

```javascript
import Anthropic from '@anthropic-ai/sdk';

async function safeClaudeRequest(messages, options = {}) {
  const maxRetries = options.maxRetries || 3;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await anthropic.messages.create({
        model: options.model || 'claude-sonnet-4-20250514',
        max_tokens: options.maxTokens || 1024,
        system: options.system,
        messages
      });

      return response;

    } catch (error) {
      // Rate limit - exponential backoff
      if (error instanceof Anthropic.RateLimitError) {
        const waitTime = Math.pow(2, attempt) * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await new Promise(r => setTimeout(r, waitTime));
        continue;
      }

      // Server error - retry
      if (error instanceof Anthropic.InternalServerError) {
        if (attempt < maxRetries - 1) {
          console.log('Server error. Retrying...');
          await new Promise(r => setTimeout(r, 1000));
          continue;
        }
      }

      // Authentication error
      if (error instanceof Anthropic.AuthenticationError) {
        throw new Error('Invalid API key');
      }

      // Bad request
      if (error instanceof Anthropic.BadRequestError) {
        throw new Error(`Invalid request: ${error.message}`);
      }

      // Overloaded
      if (error instanceof Anthropic.APIError && error.status === 529) {
        const waitTime = Math.pow(2, attempt) * 2000;
        console.log(`API overloaded. Waiting ${waitTime}ms...`);
        await new Promise(r => setTimeout(r, waitTime));
        continue;
      }

      throw error;
    }
  }

  throw new Error('Max retries exceeded');
}
```

---

## Comparing Claude and OpenAI

```
┌─────────────────────────────────────────────────────────────────┐
│                  CLAUDE VS OPENAI COMPARISON                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FEATURE              CLAUDE              GPT-4                  │
│  ───────              ──────              ─────                  │
│                                                                  │
│  Context Window       200K tokens         128K tokens            │
│  Best for Coding      Excellent           Excellent              │
│  Long Documents       Better              Good                   │
│  Safety/Refusals      More conservative   Moderate               │
│  Following Details    Very precise        Good                   │
│  Creative Writing     Strong              Strong                 │
│  Cost (Sonnet/4o)     Lower               Higher                 │
│                                                                  │
│  CHOOSE CLAUDE FOR:                                              │
│  • Very long documents (>100K tokens)                           │
│  • Detailed instruction following                               │
│  • Analysis and summarization                                   │
│  • When you need conservative outputs                           │
│                                                                  │
│  CHOOSE OPENAI FOR:                                              │
│  • Broader plugin/integration ecosystem                         │
│  • Image generation (DALL-E)                                    │
│  • Whisper (speech-to-text)                                     │
│  • Fine-tuning options                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Multi-Provider Strategy

```javascript
// Use both providers for different tasks
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic();
const openai = new OpenAI();

async function smartChat(messages, task = 'general') {
  // Route to appropriate provider
  if (task === 'long-document' || task === 'analysis') {
    // Use Claude for long context
    return await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 4096,
      messages: convertToAnthropicFormat(messages)
    });
  }

  if (task === 'image-generation') {
    // Only OpenAI has DALL-E
    return await openai.images.generate({
      model: 'dall-e-3',
      prompt: messages[messages.length - 1].content
    });
  }

  // Default to fastest/cheapest for simple tasks
  return await anthropic.messages.create({
    model: 'claude-3-5-haiku-20241022',
    max_tokens: 1024,
    messages: convertToAnthropicFormat(messages)
  });
}

// Fallback strategy
async function chatWithFallback(messages) {
  try {
    // Try primary provider
    return await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1024,
      messages: convertToAnthropicFormat(messages)
    });
  } catch (error) {
    console.log('Claude failed, falling back to OpenAI');

    // Fallback to secondary
    return await openai.chat.completions.create({
      model: 'gpt-4o',
      messages: convertToOpenAIFormat(messages)
    });
  }
}
```

---

## Building a Claude Chatbot

```javascript
// Complete Claude chatbot implementation
// app/api/claude-chat/route.js
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic();

const SYSTEM_PROMPT = `You are Claude, an AI assistant created by Anthropic.
You are helpful, harmless, and honest.

For this application, you serve as a technical documentation assistant.
You help developers understand APIs, debug code, and learn new technologies.

Guidelines:
- Provide accurate, up-to-date technical information
- Include code examples when helpful
- Admit when you're unsure about something
- Be concise but thorough`;

export async function POST(req) {
  try {
    const { messages, conversationId } = await req.json();

    // Validate input
    if (!messages || !Array.isArray(messages) || messages.length === 0) {
      return Response.json(
        { error: 'Messages array is required' },
        { status: 400 }
      );
    }

    // Convert to Anthropic format and limit history
    const anthropicMessages = messages
      .slice(-20)
      .map(msg => ({
        role: msg.role,
        content: msg.content
      }));

    // Stream response
    const stream = await anthropic.messages.stream({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2048,
      system: SYSTEM_PROMPT,
      messages: anthropicMessages
    });

    const encoder = new TextEncoder();
    let fullResponse = '';

    const readable = new ReadableStream({
      async start(controller) {
        try {
          for await (const event of stream) {
            if (event.type === 'content_block_delta' &&
                event.delta.type === 'text_delta') {
              const text = event.delta.text;
              fullResponse += text;
              controller.enqueue(encoder.encode(text));
            }
          }

          // Save to database after complete
          if (conversationId) {
            await saveConversation(conversationId, messages, fullResponse);
          }

          controller.close();
        } catch (error) {
          controller.error(error);
        }
      }
    });

    return new Response(readable, {
      headers: {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
      }
    });

  } catch (error) {
    console.error('Claude chat error:', error);

    if (error instanceof Anthropic.AuthenticationError) {
      return Response.json({ error: 'API key error' }, { status: 401 });
    }

    if (error instanceof Anthropic.RateLimitError) {
      return Response.json({ error: 'Rate limited' }, { status: 429 });
    }

    return Response.json(
      { error: 'Failed to process request' },
      { status: 500 }
    );
  }
}
```

---

## Practice Exercises

### Exercise 1: Multi-Turn Assistant

Build a conversation assistant that:
- Maintains context across messages
- Has a specific persona/expertise
- Handles errors gracefully

### Exercise 2: Image Analysis App

Create an app that:
- Accepts image uploads
- Analyzes images with Claude
- Extracts specific information

### Exercise 3: Tool-Using Agent

Build an agent that can:
- Search a database
- Perform calculations
- Take actions based on user requests

---

## Key Takeaways

1. **Use system prompts effectively** - They're separate from messages in Claude
2. **Choose the right model** - Haiku for speed, Sonnet for balance, Opus for complexity
3. **Leverage the context window** - 200K tokens enables analyzing long documents
4. **Handle tool calls in a loop** - Claude may need multiple tool calls
5. **Stream for UX** - Makes responses feel faster
6. **Consider multi-provider** - Use both Claude and OpenAI where each excels

---

## What's Next?

Tomorrow, we'll explore the **Vercel AI SDK** - a unified way to work with multiple AI providers with built-in streaming, React hooks, and edge support.
