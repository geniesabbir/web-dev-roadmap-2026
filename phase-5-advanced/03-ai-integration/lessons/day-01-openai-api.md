# Day 1: OpenAI API - Building AI-Powered Applications

## Introduction

OpenAI's API provides access to powerful language models like GPT-4 that can understand and generate human-like text. As a web developer, integrating AI capabilities can transform your applications with intelligent chatbots, content generation, code assistance, and more. Today, you'll learn how to use the OpenAI API effectively in your applications.

## Learning Objectives

By the end of this lesson, you will be able to:
- Set up and authenticate with the OpenAI API
- Make chat completions requests
- Handle streaming responses
- Implement function calling
- Build a basic AI chatbot

---

## Getting Started

### Setup

```bash
# Install the OpenAI SDK
npm install openai

# Store your API key securely
# .env.local
OPENAI_API_KEY=sk-...
```

### Basic Configuration

```javascript
// lib/openai.js
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export default openai;

// For edge/browser environments (be careful with API keys!)
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  dangerouslyAllowBrowser: true // Only for demos!
});
```

---

## Chat Completions API

### Basic Chat Request

```javascript
// Basic completion
import openai from '@/lib/openai';

async function chat(message) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: 'You are a helpful assistant.'
      },
      {
        role: 'user',
        content: message
      }
    ],
  });

  return response.choices[0].message.content;
}

// Usage
const answer = await chat('What is the capital of France?');
console.log(answer); // "The capital of France is Paris."
```

### Message Roles

```javascript
// Understanding message roles
const messages = [
  {
    role: 'system',
    // System message: Sets the AI's behavior and personality
    content: `You are a senior software engineer who explains concepts clearly.
              Always provide code examples when relevant.
              Be concise but thorough.`
  },
  {
    role: 'user',
    // User message: The human's input
    content: 'How do I implement a debounce function?'
  },
  {
    role: 'assistant',
    // Assistant message: Previous AI responses (for context)
    content: 'Here is a debounce implementation...'
  },
  {
    role: 'user',
    // Follow-up question
    content: 'Can you also show the TypeScript version?'
  }
];
```

### Conversation History

```javascript
// Managing conversation context
class Conversation {
  constructor(systemPrompt) {
    this.messages = [
      { role: 'system', content: systemPrompt }
    ];
  }

  async send(userMessage) {
    // Add user message
    this.messages.push({
      role: 'user',
      content: userMessage
    });

    // Get AI response
    const response = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages: this.messages,
    });

    const assistantMessage = response.choices[0].message;

    // Add assistant response to history
    this.messages.push(assistantMessage);

    return assistantMessage.content;
  }

  // Limit context window to prevent token overflow
  trimHistory(maxMessages = 20) {
    if (this.messages.length > maxMessages) {
      // Keep system message + last N messages
      const systemMessage = this.messages[0];
      const recentMessages = this.messages.slice(-maxMessages + 1);
      this.messages = [systemMessage, ...recentMessages];
    }
  }

  clear() {
    const systemMessage = this.messages[0];
    this.messages = [systemMessage];
  }
}

// Usage
const chat = new Conversation('You are a helpful coding assistant.');

await chat.send('How do I create a React component?');
await chat.send('Can you add TypeScript to that?');
await chat.send('How about adding props?');
```

---

## Model Parameters

```javascript
// Fine-tuning model behavior
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [...],

  // Temperature: 0-2, controls randomness
  // Lower = more focused/deterministic
  // Higher = more creative/random
  temperature: 0.7,

  // Top P: Alternative to temperature (nucleus sampling)
  // 0.1 = only consider top 10% probability tokens
  top_p: 1,

  // Max tokens in response
  max_tokens: 1000,

  // Frequency penalty: -2 to 2
  // Positive = reduce repetition of tokens
  frequency_penalty: 0,

  // Presence penalty: -2 to 2
  // Positive = encourage new topics
  presence_penalty: 0,

  // Stop sequences
  stop: ['\n\n', 'END'],

  // Number of completions to generate
  n: 1,
});

// Use cases for different temperatures:
// 0.0-0.3: Code generation, factual Q&A, data extraction
// 0.4-0.7: General conversation, balanced responses
// 0.8-1.2: Creative writing, brainstorming
// 1.3+: Highly creative/experimental outputs
```

---

## Streaming Responses

### Server-Side Streaming

```javascript
// Stream response for better UX
async function streamChat(message) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' },
      { role: 'user', content: message }
    ],
    stream: true,
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || '';
    process.stdout.write(content);
  }
}

// Next.js API route with streaming
// app/api/chat/route.js
import { OpenAIStream, StreamingTextResponse } from 'ai';
import OpenAI from 'openai';

const openai = new OpenAI();

export async function POST(req) {
  const { messages } = await req.json();

  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    stream: true,
  });

  // Convert to streaming response
  const stream = OpenAIStream(response);
  return new StreamingTextResponse(stream);
}

// Manual streaming without 'ai' package
export async function POST(req) {
  const { messages } = await req.json();

  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    stream: true,
  });

  // Create a readable stream
  const encoder = new TextEncoder();
  const readable = new ReadableStream({
    async start(controller) {
      for await (const chunk of response) {
        const text = chunk.choices[0]?.delta?.content || '';
        controller.enqueue(encoder.encode(text));
      }
      controller.close();
    },
  });

  return new Response(readable, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

### Client-Side Streaming

```jsx
// React component with streaming
import { useState } from 'react';

function ChatComponent() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    if (!input.trim()) return;

    const userMessage = { role: 'user', content: input };
    setMessages(prev => [...prev, userMessage]);
    setInput('');
    setIsLoading(true);

    // Add placeholder for assistant message
    setMessages(prev => [...prev, { role: 'assistant', content: '' }]);

    try {
      const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          messages: [...messages, userMessage]
        }),
      });

      const reader = response.body.getReader();
      const decoder = new TextDecoder();

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const text = decoder.decode(value);

        // Update the last message (assistant)
        setMessages(prev => {
          const newMessages = [...prev];
          newMessages[newMessages.length - 1] = {
            role: 'assistant',
            content: newMessages[newMessages.length - 1].content + text
          };
          return newMessages;
        });
      }
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setIsLoading(false);
    }
  }

  return (
    <div className="chat-container">
      <div className="messages">
        {messages.map((msg, i) => (
          <div key={i} className={`message ${msg.role}`}>
            {msg.content}
          </div>
        ))}
      </div>

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type a message..."
          disabled={isLoading}
        />
        <button type="submit" disabled={isLoading}>
          Send
        </button>
      </form>
    </div>
  );
}
```

---

## Function Calling

### Defining Functions

```javascript
// Function calling allows the AI to trigger your code
const tools = [
  {
    type: 'function',
    function: {
      name: 'get_weather',
      description: 'Get the current weather in a given location',
      parameters: {
        type: 'object',
        properties: {
          location: {
            type: 'string',
            description: 'The city and state, e.g. San Francisco, CA'
          },
          unit: {
            type: 'string',
            enum: ['celsius', 'fahrenheit'],
            description: 'Temperature unit'
          }
        },
        required: ['location']
      }
    }
  },
  {
    type: 'function',
    function: {
      name: 'search_products',
      description: 'Search for products in the catalog',
      parameters: {
        type: 'object',
        properties: {
          query: {
            type: 'string',
            description: 'Search query'
          },
          category: {
            type: 'string',
            description: 'Product category'
          },
          max_price: {
            type: 'number',
            description: 'Maximum price filter'
          }
        },
        required: ['query']
      }
    }
  }
];
```

### Handling Function Calls

```javascript
// Complete function calling flow
async function chatWithFunctions(userMessage) {
  const messages = [
    { role: 'user', content: userMessage }
  ];

  // First API call
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    tools,
    tool_choice: 'auto', // Let AI decide when to use tools
  });

  const assistantMessage = response.choices[0].message;

  // Check if AI wants to call a function
  if (assistantMessage.tool_calls) {
    messages.push(assistantMessage);

    // Execute each function call
    for (const toolCall of assistantMessage.tool_calls) {
      const functionName = toolCall.function.name;
      const functionArgs = JSON.parse(toolCall.function.arguments);

      // Execute the actual function
      const functionResult = await executeFunction(functionName, functionArgs);

      // Add function result to messages
      messages.push({
        role: 'tool',
        tool_call_id: toolCall.id,
        content: JSON.stringify(functionResult)
      });
    }

    // Second API call with function results
    const secondResponse = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages,
    });

    return secondResponse.choices[0].message.content;
  }

  return assistantMessage.content;
}

// Function implementations
async function executeFunction(name, args) {
  switch (name) {
    case 'get_weather':
      return await getWeather(args.location, args.unit);

    case 'search_products':
      return await searchProducts(args.query, args.category, args.max_price);

    default:
      throw new Error(`Unknown function: ${name}`);
  }
}

async function getWeather(location, unit = 'celsius') {
  // Call weather API
  const response = await fetch(
    `https://api.weather.com/v1/current?location=${encodeURIComponent(location)}`
  );
  return response.json();
}

async function searchProducts(query, category, maxPrice) {
  // Search your product database
  return prisma.product.findMany({
    where: {
      name: { contains: query },
      category: category || undefined,
      price: maxPrice ? { lte: maxPrice } : undefined
    },
    take: 5
  });
}

// Usage
const response = await chatWithFunctions(
  "What's the weather like in Tokyo and find me some umbrellas under $30"
);
```

---

## Structured Outputs (JSON Mode)

```javascript
// Force JSON output format
async function extractData(text) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: `Extract the following information from the text and return as JSON:
                  - name: string
                  - email: string
                  - phone: string
                  - company: string
                  Return null for any field not found.`
      },
      {
        role: 'user',
        content: text
      }
    ],
    response_format: { type: 'json_object' },
  });

  return JSON.parse(response.choices[0].message.content);
}

// Usage
const data = await extractData(`
  Hi, I'm John Smith from Acme Corp.
  You can reach me at john@acme.com or call 555-1234.
`);
// { name: "John Smith", email: "john@acme.com", phone: "555-1234", company: "Acme Corp" }

// With JSON Schema validation
async function analyzeReview(review) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: `Analyze the product review and extract:
                  {
                    "sentiment": "positive" | "negative" | "neutral",
                    "rating": 1-5,
                    "pros": string[],
                    "cons": string[],
                    "summary": string (max 100 chars)
                  }`
      },
      { role: 'user', content: review }
    ],
    response_format: { type: 'json_object' },
  });

  return JSON.parse(response.choices[0].message.content);
}
```

---

## Error Handling

```javascript
// Robust error handling
import OpenAI from 'openai';

async function safeChatCompletion(messages, retries = 3) {
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      const response = await openai.chat.completions.create({
        model: 'gpt-4o',
        messages,
      });

      return response.choices[0].message.content;

    } catch (error) {
      // Rate limit error - wait and retry
      if (error instanceof OpenAI.RateLimitError) {
        const waitTime = Math.pow(2, attempt) * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await new Promise(r => setTimeout(r, waitTime));
        continue;
      }

      // API error (500s)
      if (error instanceof OpenAI.APIError) {
        if (error.status >= 500 && attempt < retries - 1) {
          console.log(`API error. Retrying...`);
          await new Promise(r => setTimeout(r, 1000));
          continue;
        }
      }

      // Authentication error
      if (error instanceof OpenAI.AuthenticationError) {
        throw new Error('Invalid API key');
      }

      // Bad request
      if (error instanceof OpenAI.BadRequestError) {
        throw new Error(`Invalid request: ${error.message}`);
      }

      // Context length exceeded
      if (error.code === 'context_length_exceeded') {
        throw new Error('Message too long. Please shorten your input.');
      }

      throw error;
    }
  }

  throw new Error('Max retries exceeded');
}

// API route with error handling
export async function POST(req) {
  try {
    const { messages } = await req.json();

    const response = await safeChatCompletion(messages);

    return Response.json({ content: response });

  } catch (error) {
    console.error('Chat error:', error);

    return Response.json(
      { error: error.message || 'An error occurred' },
      { status: error.status || 500 }
    );
  }
}
```

---

## Cost Management

```javascript
// Track token usage and costs
const MODEL_COSTS = {
  'gpt-4o': { input: 0.0025, output: 0.01 },      // per 1K tokens
  'gpt-4o-mini': { input: 0.00015, output: 0.0006 },
  'gpt-4-turbo': { input: 0.01, output: 0.03 },
};

async function chatWithTracking(messages, model = 'gpt-4o') {
  const response = await openai.chat.completions.create({
    model,
    messages,
  });

  const usage = response.usage;
  const costs = MODEL_COSTS[model];

  const inputCost = (usage.prompt_tokens / 1000) * costs.input;
  const outputCost = (usage.completion_tokens / 1000) * costs.output;
  const totalCost = inputCost + outputCost;

  console.log(`Tokens: ${usage.prompt_tokens} in, ${usage.completion_tokens} out`);
  console.log(`Cost: $${totalCost.toFixed(6)}`);

  // Store for analytics
  await trackUsage({
    model,
    promptTokens: usage.prompt_tokens,
    completionTokens: usage.completion_tokens,
    cost: totalCost,
    timestamp: new Date()
  });

  return response.choices[0].message.content;
}

// Token estimation (approximate)
function estimateTokens(text) {
  // Rough estimate: ~4 characters per token for English
  return Math.ceil(text.length / 4);
}

// Check before sending to avoid surprises
function validateMessageLength(messages, maxTokens = 8000) {
  const totalText = messages.map(m => m.content).join(' ');
  const estimatedTokens = estimateTokens(totalText);

  if (estimatedTokens > maxTokens) {
    throw new Error(`Message too long: ~${estimatedTokens} tokens (max: ${maxTokens})`);
  }

  return estimatedTokens;
}
```

---

## Building a Chatbot API

```javascript
// Complete chatbot API with all features
// app/api/chat/route.js
import OpenAI from 'openai';
import { prisma } from '@/lib/prisma';

const openai = new OpenAI();

const SYSTEM_PROMPT = `You are a helpful customer support assistant for TechStore.
You help customers with:
- Product information and recommendations
- Order status and tracking
- Returns and refunds
- Technical support

Be friendly, concise, and helpful. If you don't know something, say so.`;

export async function POST(req) {
  try {
    const { messages, conversationId, userId } = await req.json();

    // Validate
    if (!messages || !Array.isArray(messages)) {
      return Response.json(
        { error: 'Messages array required' },
        { status: 400 }
      );
    }

    // Build messages with system prompt
    const fullMessages = [
      { role: 'system', content: SYSTEM_PROMPT },
      ...messages.slice(-20) // Keep last 20 messages for context
    ];

    // Create completion with streaming
    const stream = await openai.chat.completions.create({
      model: 'gpt-4o-mini', // Use cheaper model for chat
      messages: fullMessages,
      stream: true,
      max_tokens: 500,
      temperature: 0.7,
    });

    // Stream response
    const encoder = new TextEncoder();
    let fullResponse = '';

    const readable = new ReadableStream({
      async start(controller) {
        for await (const chunk of stream) {
          const text = chunk.choices[0]?.delta?.content || '';
          fullResponse += text;
          controller.enqueue(encoder.encode(text));
        }

        // Save conversation after streaming completes
        if (conversationId && userId) {
          await saveMessage(conversationId, userId, messages[messages.length - 1].content, fullResponse);
        }

        controller.close();
      },
    });

    return new Response(readable, {
      headers: {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
      },
    });

  } catch (error) {
    console.error('Chat error:', error);
    return Response.json(
      { error: 'Failed to process message' },
      { status: 500 }
    );
  }
}

async function saveMessage(conversationId, userId, userMessage, assistantMessage) {
  await prisma.message.createMany({
    data: [
      {
        conversationId,
        role: 'user',
        content: userMessage,
      },
      {
        conversationId,
        role: 'assistant',
        content: assistantMessage,
      }
    ]
  });
}
```

---

## Practice Exercises

### Exercise 1: Simple Q&A Bot

Build a Q&A bot that:
- Answers questions about your product/service
- Maintains conversation context
- Handles errors gracefully

### Exercise 2: Content Generator

Create a content generation tool:
- Generate blog post outlines
- Create social media posts
- Output in structured JSON format

### Exercise 3: Function Calling

Implement an assistant that can:
- Search your database
- Create new records
- Perform calculations

---

## Key Takeaways

1. **Use the right model** - GPT-4o-mini for simple tasks, GPT-4o for complex ones
2. **Stream for UX** - Streaming makes responses feel faster
3. **Manage context** - Trim conversation history to control costs
4. **Handle errors** - Implement retries and graceful degradation
5. **Track costs** - Monitor token usage to avoid surprises
6. **Secure API keys** - Never expose keys to the client

---

## What's Next?

Tomorrow, we'll explore the **Anthropic API** - learning how to use Claude for different use cases and comparing it with OpenAI's offerings.
