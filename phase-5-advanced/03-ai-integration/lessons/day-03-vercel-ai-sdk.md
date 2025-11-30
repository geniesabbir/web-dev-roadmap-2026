# Day 3: Vercel AI SDK - Unified AI Development

## Introduction

The Vercel AI SDK is a powerful toolkit that simplifies building AI-powered applications. It provides a unified interface for multiple AI providers, built-in streaming support, React hooks for chat UIs, and edge runtime compatibility. Today, you'll learn how to use the AI SDK to build production-ready AI features quickly.

## Learning Objectives

By the end of this lesson, you will be able to:
- Set up the Vercel AI SDK with multiple providers
- Use React hooks for chat interfaces
- Implement streaming responses
- Handle tool calls with the SDK
- Build generative UI components

---

## Getting Started

### Installation

```bash
# Core AI SDK
npm install ai

# Provider packages (install what you need)
npm install @ai-sdk/openai
npm install @ai-sdk/anthropic
npm install @ai-sdk/google
npm install @ai-sdk/mistral
```

### Provider Setup

```javascript
// lib/ai.js
import { createOpenAI } from '@ai-sdk/openai';
import { createAnthropic } from '@ai-sdk/anthropic';
import { createGoogleGenerativeAI } from '@ai-sdk/google';

// OpenAI provider
export const openai = createOpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Anthropic provider
export const anthropic = createAnthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Google provider
export const google = createGoogleGenerativeAI({
  apiKey: process.env.GOOGLE_API_KEY,
});

// Usage with specific models
export const gpt4o = openai('gpt-4o');
export const claude = anthropic('claude-sonnet-4-20250514');
export const gemini = google('gemini-1.5-pro');
```

---

## Core Functions

### generateText

```javascript
import { generateText } from 'ai';
import { openai } from '@/lib/ai';

// Simple text generation
async function generate() {
  const { text, usage } = await generateText({
    model: openai('gpt-4o'),
    prompt: 'Explain React hooks in one paragraph.',
  });

  console.log(text);
  console.log('Tokens:', usage);
}

// With system prompt and messages
async function chat() {
  const { text } = await generateText({
    model: openai('gpt-4o'),
    system: 'You are a helpful coding assistant.',
    messages: [
      { role: 'user', content: 'How do I use useEffect?' },
      { role: 'assistant', content: 'useEffect is...' },
      { role: 'user', content: 'Show me an example with cleanup.' }
    ],
  });

  return text;
}

// With options
const result = await generateText({
  model: openai('gpt-4o'),
  prompt: 'Write a haiku about coding.',
  maxTokens: 100,
  temperature: 0.8,
});
```

### streamText

```javascript
import { streamText } from 'ai';
import { anthropic } from '@/lib/ai';

// Stream text response
async function stream() {
  const result = await streamText({
    model: anthropic('claude-sonnet-4-20250514'),
    prompt: 'Write a short story about a robot learning to code.',
  });

  // Iterate over stream
  for await (const textPart of result.textStream) {
    process.stdout.write(textPart);
  }

  // Or get full text after stream
  const fullText = await result.text;

  // Get usage after completion
  const usage = await result.usage;
}

// API route with streaming
// app/api/generate/route.js
import { streamText } from 'ai';
import { openai } from '@/lib/ai';

export async function POST(req) {
  const { prompt } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'),
    prompt,
  });

  // Return as streaming response
  return result.toDataStreamResponse();
}
```

### generateObject

```javascript
import { generateObject } from 'ai';
import { openai } from '@/lib/ai';
import { z } from 'zod';

// Generate structured data
async function extractRecipe() {
  const { object } = await generateObject({
    model: openai('gpt-4o'),
    schema: z.object({
      name: z.string(),
      ingredients: z.array(z.object({
        item: z.string(),
        amount: z.string(),
      })),
      steps: z.array(z.string()),
      prepTime: z.number(),
      cookTime: z.number(),
      servings: z.number(),
    }),
    prompt: 'Generate a recipe for chocolate chip cookies.',
  });

  console.log(object);
  // {
  //   name: "Classic Chocolate Chip Cookies",
  //   ingredients: [
  //     { item: "flour", amount: "2 1/4 cups" },
  //     ...
  //   ],
  //   steps: ["Preheat oven...", ...],
  //   prepTime: 15,
  //   cookTime: 12,
  //   servings: 24
  // }
}

// Extract data from text
async function extractContactInfo(text) {
  const { object } = await generateObject({
    model: openai('gpt-4o'),
    schema: z.object({
      name: z.string().nullable(),
      email: z.string().email().nullable(),
      phone: z.string().nullable(),
      company: z.string().nullable(),
    }),
    prompt: `Extract contact information from: ${text}`,
  });

  return object;
}

// Streaming object generation
import { streamObject } from 'ai';

async function streamRecipe() {
  const result = await streamObject({
    model: openai('gpt-4o'),
    schema: recipeSchema,
    prompt: 'Generate a recipe for pasta carbonara.',
  });

  for await (const partialObject of result.partialObjectStream) {
    console.log(partialObject);
    // Partial objects as they're generated
  }

  const finalObject = await result.object;
}
```

---

## React Hooks

### useChat

```jsx
// components/Chat.jsx
'use client';

import { useChat } from 'ai/react';

export function Chat() {
  const {
    messages,
    input,
    handleInputChange,
    handleSubmit,
    isLoading,
    error,
    reload,
    stop,
    setMessages,
  } = useChat({
    api: '/api/chat',
    initialMessages: [],
    onResponse: (response) => {
      console.log('Response started:', response);
    },
    onFinish: (message) => {
      console.log('Message complete:', message);
    },
    onError: (error) => {
      console.error('Chat error:', error);
    },
  });

  return (
    <div className="chat-container">
      <div className="messages">
        {messages.map((message) => (
          <div
            key={message.id}
            className={`message ${message.role}`}
          >
            <div className="avatar">
              {message.role === 'user' ? '👤' : '🤖'}
            </div>
            <div className="content">
              {message.content}
            </div>
          </div>
        ))}

        {isLoading && (
          <div className="message assistant loading">
            <div className="typing-indicator">
              <span></span><span></span><span></span>
            </div>
          </div>
        )}
      </div>

      {error && (
        <div className="error">
          Error: {error.message}
          <button onClick={reload}>Retry</button>
        </div>
      )}

      <form onSubmit={handleSubmit} className="input-form">
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Type a message..."
          disabled={isLoading}
        />
        <button type="submit" disabled={isLoading || !input.trim()}>
          Send
        </button>
        {isLoading && (
          <button type="button" onClick={stop}>
            Stop
          </button>
        )}
      </form>
    </div>
  );
}

// API route for useChat
// app/api/chat/route.js
import { streamText } from 'ai';
import { openai } from '@/lib/ai';

export async function POST(req) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'),
    system: 'You are a helpful assistant.',
    messages,
  });

  return result.toDataStreamResponse();
}
```

### useCompletion

```jsx
// For single-turn text completion
'use client';

import { useCompletion } from 'ai/react';

export function CompletionDemo() {
  const {
    completion,
    input,
    handleInputChange,
    handleSubmit,
    isLoading,
  } = useCompletion({
    api: '/api/complete',
  });

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Enter a prompt..."
        />
        <button type="submit" disabled={isLoading}>
          Complete
        </button>
      </form>

      {completion && (
        <div className="completion">
          {completion}
        </div>
      )}
    </div>
  );
}

// API route
// app/api/complete/route.js
import { streamText } from 'ai';
import { openai } from '@/lib/ai';

export async function POST(req) {
  const { prompt } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'),
    prompt,
  });

  return result.toDataStreamResponse();
}
```

### useObject

```jsx
// For streaming structured data
'use client';

import { experimental_useObject as useObject } from 'ai/react';
import { z } from 'zod';

const recipeSchema = z.object({
  name: z.string(),
  ingredients: z.array(z.string()),
  steps: z.array(z.string()),
});

export function RecipeGenerator() {
  const { object, submit, isLoading } = useObject({
    api: '/api/recipe',
    schema: recipeSchema,
  });

  return (
    <div>
      <button
        onClick={() => submit({ dish: 'spaghetti bolognese' })}
        disabled={isLoading}
      >
        Generate Recipe
      </button>

      {object && (
        <div className="recipe">
          <h2>{object.name ?? 'Loading...'}</h2>

          <h3>Ingredients</h3>
          <ul>
            {object.ingredients?.map((ing, i) => (
              <li key={i}>{ing}</li>
            ))}
          </ul>

          <h3>Steps</h3>
          <ol>
            {object.steps?.map((step, i) => (
              <li key={i}>{step}</li>
            ))}
          </ol>
        </div>
      )}
    </div>
  );
}
```

---

## Tool Calling

### Defining Tools

```javascript
import { generateText, tool } from 'ai';
import { openai } from '@/lib/ai';
import { z } from 'zod';

// Define tools with Zod schemas
const tools = {
  getWeather: tool({
    description: 'Get the current weather in a location',
    parameters: z.object({
      location: z.string().describe('The city and country'),
      unit: z.enum(['celsius', 'fahrenheit']).optional(),
    }),
    execute: async ({ location, unit = 'celsius' }) => {
      // Call weather API
      const response = await fetch(
        `https://api.weather.com?location=${location}&unit=${unit}`
      );
      return response.json();
    },
  }),

  searchProducts: tool({
    description: 'Search for products in the store',
    parameters: z.object({
      query: z.string(),
      category: z.string().optional(),
      maxPrice: z.number().optional(),
    }),
    execute: async ({ query, category, maxPrice }) => {
      const products = await prisma.product.findMany({
        where: {
          name: { contains: query, mode: 'insensitive' },
          category: category || undefined,
          price: maxPrice ? { lte: maxPrice } : undefined,
        },
        take: 5,
      });
      return products;
    },
  }),

  createTask: tool({
    description: 'Create a new task in the todo list',
    parameters: z.object({
      title: z.string(),
      dueDate: z.string().optional(),
      priority: z.enum(['low', 'medium', 'high']).optional(),
    }),
    execute: async ({ title, dueDate, priority }) => {
      const task = await prisma.task.create({
        data: { title, dueDate, priority },
      });
      return task;
    },
  }),
};
```

### Using Tools with generateText

```javascript
import { generateText } from 'ai';
import { openai } from '@/lib/ai';

async function assistantWithTools(userMessage) {
  const result = await generateText({
    model: openai('gpt-4o'),
    tools,
    maxSteps: 5, // Max tool call rounds
    system: 'You are a helpful assistant that can check weather, search products, and manage tasks.',
    prompt: userMessage,
    onStepFinish: ({ text, toolCalls, toolResults }) => {
      // Called after each step
      if (toolCalls) {
        console.log('Tool calls:', toolCalls);
      }
      if (toolResults) {
        console.log('Tool results:', toolResults);
      }
    },
  });

  return result.text;
}

// Usage
const response = await assistantWithTools(
  "What's the weather in Paris? Also, find me some umbrellas under $30."
);
```

### Streaming with Tools

```javascript
import { streamText } from 'ai';

async function streamWithTools(userMessage) {
  const result = await streamText({
    model: openai('gpt-4o'),
    tools,
    maxSteps: 5,
    prompt: userMessage,
  });

  // Stream includes tool call info
  for await (const part of result.fullStream) {
    switch (part.type) {
      case 'text-delta':
        process.stdout.write(part.textDelta);
        break;

      case 'tool-call':
        console.log('\nCalling tool:', part.toolName);
        break;

      case 'tool-result':
        console.log('Tool result:', part.result);
        break;
    }
  }
}

// API route with tools
// app/api/assistant/route.js
import { streamText } from 'ai';
import { openai } from '@/lib/ai';

export async function POST(req) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'),
    tools,
    maxSteps: 5,
    messages,
  });

  return result.toDataStreamResponse();
}
```

### React Hook with Tools

```jsx
'use client';

import { useChat } from 'ai/react';

export function AssistantChat() {
  const { messages, input, handleSubmit, handleInputChange } = useChat({
    api: '/api/assistant',
    maxSteps: 5,
  });

  return (
    <div>
      <div className="messages">
        {messages.map((message) => (
          <div key={message.id}>
            <strong>{message.role}:</strong>
            {message.content}

            {/* Show tool invocations */}
            {message.toolInvocations?.map((tool, i) => (
              <div key={i} className="tool-call">
                <div>Tool: {tool.toolName}</div>
                <div>Args: {JSON.stringify(tool.args)}</div>
                {tool.state === 'result' && (
                  <div>Result: {JSON.stringify(tool.result)}</div>
                )}
              </div>
            ))}
          </div>
        ))}
      </div>

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Ask me anything..."
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

---

## Generative UI

```jsx
// Server-side UI generation with tools
// app/api/generative-ui/route.js
import { streamUI } from 'ai/rsc';
import { openai } from '@/lib/ai';
import { z } from 'zod';

export async function POST(req) {
  const { messages } = await req.json();

  const result = await streamUI({
    model: openai('gpt-4o'),
    messages,
    text: ({ content }) => <div>{content}</div>,
    tools: {
      showWeather: {
        description: 'Show weather widget for a location',
        parameters: z.object({
          location: z.string(),
        }),
        generate: async function* ({ location }) {
          yield <WeatherLoading location={location} />;

          const weather = await fetchWeather(location);

          return <WeatherCard weather={weather} />;
        },
      },

      showProducts: {
        description: 'Show product cards',
        parameters: z.object({
          query: z.string(),
        }),
        generate: async function* ({ query }) {
          yield <ProductsLoading />;

          const products = await searchProducts(query);

          return <ProductGrid products={products} />;
        },
      },

      showChart: {
        description: 'Show a data chart',
        parameters: z.object({
          type: z.enum(['bar', 'line', 'pie']),
          data: z.array(z.object({
            label: z.string(),
            value: z.number(),
          })),
        }),
        generate: async ({ type, data }) => {
          return <Chart type={type} data={data} />;
        },
      },
    },
  });

  return result.value;
}

// React components
function WeatherLoading({ location }) {
  return (
    <div className="weather-loading">
      Loading weather for {location}...
    </div>
  );
}

function WeatherCard({ weather }) {
  return (
    <div className="weather-card">
      <h3>{weather.location}</h3>
      <div className="temp">{weather.temperature}°</div>
      <div className="condition">{weather.condition}</div>
    </div>
  );
}

function ProductGrid({ products }) {
  return (
    <div className="product-grid">
      {products.map((product) => (
        <div key={product.id} className="product-card">
          <img src={product.image} alt={product.name} />
          <h4>{product.name}</h4>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}
```

---

## Multi-Provider Support

```javascript
// Easy provider switching
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';
import { anthropic } from '@ai-sdk/anthropic';
import { google } from '@ai-sdk/google';

// Same code, different providers
async function generateWithProvider(provider, prompt) {
  const models = {
    openai: openai('gpt-4o'),
    anthropic: anthropic('claude-sonnet-4-20250514'),
    google: google('gemini-1.5-pro'),
  };

  const { text } = await generateText({
    model: models[provider],
    prompt,
  });

  return text;
}

// Fallback strategy
async function generateWithFallback(prompt) {
  const providers = [
    { name: 'anthropic', model: anthropic('claude-sonnet-4-20250514') },
    { name: 'openai', model: openai('gpt-4o') },
    { name: 'google', model: google('gemini-1.5-pro') },
  ];

  for (const { name, model } of providers) {
    try {
      const { text } = await generateText({ model, prompt });
      console.log(`Success with ${name}`);
      return text;
    } catch (error) {
      console.log(`${name} failed:`, error.message);
      continue;
    }
  }

  throw new Error('All providers failed');
}

// A/B testing providers
async function abTestProviders(prompt, userId) {
  // Simple hash-based assignment
  const bucket = userId.charCodeAt(0) % 2;

  const model = bucket === 0
    ? openai('gpt-4o')
    : anthropic('claude-sonnet-4-20250514');

  const { text, usage } = await generateText({ model, prompt });

  // Log for analysis
  await logExperiment({
    userId,
    bucket,
    provider: bucket === 0 ? 'openai' : 'anthropic',
    usage,
  });

  return text;
}
```

---

## Edge Runtime Support

```javascript
// Works in Edge runtime (Vercel Edge, Cloudflare Workers)
// app/api/chat/route.js
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export const runtime = 'edge';

export async function POST(req) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'),
    messages,
  });

  return result.toDataStreamResponse();
}

// Middleware for auth/rate limiting
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Check rate limit
  const ip = request.ip || 'unknown';
  // ... rate limit logic

  // Check auth
  const token = request.headers.get('authorization');
  // ... auth logic

  return NextResponse.next();
}

export const config = {
  matcher: '/api/chat',
};
```

---

## Practice Exercises

### Exercise 1: Multi-Provider Chat

Build a chat interface that:
- Lets users choose their AI provider
- Streams responses
- Shows token usage

### Exercise 2: Tool-Using Assistant

Create an assistant that can:
- Search a database
- Create/update records
- Perform calculations

### Exercise 3: Generative UI

Build a dashboard that:
- Generates charts from natural language
- Shows dynamic data widgets
- Updates in real-time

---

## Key Takeaways

1. **Unified interface** - Same code works with multiple providers
2. **Built-in streaming** - First-class streaming support
3. **React hooks** - Easy chat UI with useChat/useCompletion
4. **Structured output** - generateObject with Zod schemas
5. **Tool support** - Declarative tool definitions
6. **Edge ready** - Works in serverless/edge environments

---

## What's Next?

Tomorrow, we'll dive into **Prompt Engineering** - learning techniques to get better, more consistent results from AI models through effective prompt design.
