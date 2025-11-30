# AI Integration

**Duration:** 2 weeks

## Learning Objectives

By the end of this section, you will:
- Integrate LLM APIs into applications
- Build AI-powered features
- Understand prompt engineering basics
- Create RAG (Retrieval Augmented Generation) systems

---

## Week 1: LLM APIs

### OpenAI Integration

```bash
npm install openai
```

```typescript
// src/lib/openai.ts
import OpenAI from 'openai'

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
})

// Basic completion
export async function generateText(prompt: string) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' },
      { role: 'user', content: prompt },
    ],
    max_tokens: 1000,
    temperature: 0.7,
  })

  return response.choices[0].message.content
}

// Streaming response
export async function* streamText(prompt: string) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [{ role: 'user', content: prompt }],
    stream: true,
  })

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content
    if (content) yield content
  }
}
```

### Anthropic (Claude) Integration

```bash
npm install @anthropic-ai/sdk
```

```typescript
// src/lib/anthropic.ts
import Anthropic from '@anthropic-ai/sdk'

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
})

export async function generateWithClaude(prompt: string) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [{ role: 'user', content: prompt }],
  })

  return response.content[0].type === 'text'
    ? response.content[0].text
    : ''
}
```

### Vercel AI SDK

```bash
npm install ai openai
```

```typescript
// app/api/chat/route.ts
import { OpenAIStream, StreamingTextResponse } from 'ai'
import OpenAI from 'openai'

const openai = new OpenAI()

export async function POST(req: Request) {
  const { messages } = await req.json()

  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    stream: true,
  })

  const stream = OpenAIStream(response)
  return new StreamingTextResponse(stream)
}

// Frontend hook
'use client'
import { useChat } from 'ai/react'

export function ChatComponent() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat()

  return (
    <div>
      {messages.map((m) => (
        <div key={m.id}>
          <strong>{m.role}:</strong> {m.content}
        </div>
      ))}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} />
        <button disabled={isLoading}>Send</button>
      </form>
    </div>
  )
}
```

---

## Week 2: Building AI Features

### Prompt Engineering

```typescript
// Structured prompts
const systemPrompt = `You are a customer support agent for an e-commerce store.
Your responsibilities:
1. Answer questions about orders and products
2. Help with returns and refunds
3. Be polite and professional

Guidelines:
- Keep responses concise (under 100 words)
- If you don't know, say so
- Never make up information about orders`

// Few-shot prompting
const fewShotPrompt = `
Classify the sentiment of these reviews:

Review: "This product is amazing, best purchase ever!"
Sentiment: Positive

Review: "Terrible quality, broke after one day"
Sentiment: Negative

Review: "It's okay, nothing special"
Sentiment: Neutral

Review: "${userReview}"
Sentiment:`

// JSON output
const jsonPrompt = `
Extract product information from this text and return as JSON:
{name, price, description, category}

Text: "${productDescription}"

JSON:`
```

### AI-Powered Features

```typescript
// Content summarization
export async function summarizeArticle(content: string) {
  return generateText(`
    Summarize this article in 3 bullet points:

    ${content}

    Summary:
  `)
}

// Smart search
export async function enhanceSearchQuery(query: string) {
  const response = await generateText(`
    Given this search query: "${query}"

    Generate 3 related search terms that might help find relevant results.
    Return as JSON array: ["term1", "term2", "term3"]
  `)

  return JSON.parse(response)
}

// Content moderation
export async function moderateContent(text: string) {
  const response = await openai.moderations.create({ input: text })
  return response.results[0]
}
```

### RAG (Retrieval Augmented Generation)

```bash
npm install @pinecone-database/pinecone
```

```typescript
// Vector database setup
import { Pinecone } from '@pinecone-database/pinecone'

const pinecone = new Pinecone()
const index = pinecone.index('knowledge-base')

// Generate embeddings
async function getEmbedding(text: string) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: text,
  })
  return response.data[0].embedding
}

// Index documents
async function indexDocument(id: string, content: string) {
  const embedding = await getEmbedding(content)

  await index.upsert([{
    id,
    values: embedding,
    metadata: { content },
  }])
}

// RAG query
async function ragQuery(question: string) {
  // 1. Get question embedding
  const questionEmbedding = await getEmbedding(question)

  // 2. Find relevant documents
  const results = await index.query({
    vector: questionEmbedding,
    topK: 3,
    includeMetadata: true,
  })

  // 3. Build context from results
  const context = results.matches
    .map((m) => m.metadata?.content)
    .join('\n\n')

  // 4. Generate answer with context
  const answer = await generateText(`
    Answer this question based on the following context.
    If the context doesn't contain the answer, say "I don't have information about that."

    Context:
    ${context}

    Question: ${question}

    Answer:
  `)

  return answer
}
```

### AI Chatbot with Memory

```typescript
// Conversation with history
interface Message {
  role: 'user' | 'assistant' | 'system'
  content: string
}

class ChatBot {
  private messages: Message[] = []

  constructor(systemPrompt: string) {
    this.messages = [{ role: 'system', content: systemPrompt }]
  }

  async chat(userMessage: string): Promise<string> {
    this.messages.push({ role: 'user', content: userMessage })

    const response = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages: this.messages,
    })

    const assistantMessage = response.choices[0].message.content!
    this.messages.push({ role: 'assistant', content: assistantMessage })

    // Keep last 20 messages to manage context window
    if (this.messages.length > 21) {
      this.messages = [
        this.messages[0], // Keep system prompt
        ...this.messages.slice(-20),
      ]
    }

    return assistantMessage
  }
}
```

---

## Best Practices

### Error Handling
```typescript
async function safeGenerate(prompt: string) {
  try {
    return await generateText(prompt)
  } catch (error) {
    if (error instanceof OpenAI.RateLimitError) {
      // Retry with exponential backoff
      await sleep(1000)
      return safeGenerate(prompt)
    }
    throw error
  }
}
```

### Cost Management
```typescript
// Track token usage
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages,
})

console.log('Tokens used:', response.usage?.total_tokens)

// Use cheaper models for simple tasks
const model = taskComplexity === 'simple' ? 'gpt-4o-mini' : 'gpt-4o'
```

### Rate Limiting
```typescript
import { RateLimiter } from 'limiter'

const limiter = new RateLimiter({
  tokensPerInterval: 60,
  interval: 'minute',
})

async function rateLimitedGenerate(prompt: string) {
  await limiter.removeTokens(1)
  return generateText(prompt)
}
```

---

## Project Ideas

1. **AI Customer Support** - Chatbot that answers FAQs
2. **Content Generator** - Blog post or product description writer
3. **Code Assistant** - Help with code explanations
4. **Smart Search** - Search with natural language queries
5. **Document Q&A** - Upload PDFs and ask questions

---

## Resources

- [OpenAI Documentation](https://platform.openai.com/docs)
- [Anthropic Documentation](https://docs.anthropic.com/)
- [Vercel AI SDK](https://sdk.vercel.ai/docs)
- [Pinecone Documentation](https://docs.pinecone.io/)

---

**Next:** [Job Preparation](../04-job-prep/README.md)
