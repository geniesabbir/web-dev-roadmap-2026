# Days 6-7: AI-Powered Project - Building an Intelligent Document Assistant

## Introduction

Over these two days, we'll build a production-ready AI-powered document assistant that combines everything we've learned: API integration, streaming, prompt engineering, and RAG. This project demonstrates real-world patterns for building AI applications.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Document Assistant Architecture                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        Frontend (Next.js)                        │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │   │
│  │  │   Upload    │  │    Chat     │  │       Analytics         │  │   │
│  │  │  Documents  │  │  Interface  │  │       Dashboard         │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘  │   │
│  └────────────────────────────┬────────────────────────────────────┘   │
│                               │                                         │
│  ┌────────────────────────────▼────────────────────────────────────┐   │
│  │                        API Routes                                │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │   │
│  │  │   /ingest   │  │   /chat     │  │     /conversations      │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘  │   │
│  └────────────────────────────┬────────────────────────────────────┘   │
│                               │                                         │
│  ┌────────────────────────────▼────────────────────────────────────┐   │
│  │                      Services Layer                              │   │
│  │  ┌───────────┐  ┌────────────┐  ┌──────────┐  ┌──────────────┐  │   │
│  │  │    RAG    │  │   Chat     │  │ Document │  │   Analytics  │  │   │
│  │  │  Pipeline │  │  Service   │  │ Processor│  │   Service    │  │   │
│  │  └───────────┘  └────────────┘  └──────────┘  └──────────────┘  │   │
│  └────────────────────────────┬────────────────────────────────────┘   │
│                               │                                         │
│  ┌────────────────────────────▼────────────────────────────────────┐   │
│  │                      Data Layer                                  │   │
│  │  ┌───────────────────┐  ┌───────────────────────────────────┐   │   │
│  │  │  Vector Database  │  │         PostgreSQL               │   │   │
│  │  │    (Pinecone)     │  │  (Users, Conversations, Docs)    │   │   │
│  │  └───────────────────┘  └───────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

By the end of this project, you will:
- Build a complete AI application from scratch
- Implement document ingestion with progress tracking
- Create a streaming chat interface with citations
- Handle conversation history and context
- Add usage analytics and rate limiting
- Deploy to production with proper error handling

---

## Project Setup

### Initialize the Project

```bash
# Create Next.js project
npx create-next-app@latest doc-assistant --typescript --tailwind --app --src-dir

cd doc-assistant

# Install dependencies
npm install openai @pinecone-database/pinecone pdf-parse mammoth
npm install @tanstack/react-query zustand
npm install lucide-react clsx tailwind-merge
npm install -D prisma @types/pdf-parse
```

### Environment Configuration

```env
# .env.local
OPENAI_API_KEY=sk-...
PINECONE_API_KEY=...
PINECONE_INDEX=doc-assistant

DATABASE_URL=postgresql://...

# Optional: Analytics
POSTHOG_KEY=...
```

### Database Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String         @id @default(cuid())
  email         String         @unique
  name          String?
  documents     Document[]
  conversations Conversation[]
  usage         Usage[]
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
}

model Document {
  id          String   @id @default(cuid())
  name        String
  type        String
  size        Int
  chunkCount  Int
  status      String   @default("processing")
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  metadata    Json     @default("{}")
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([userId])
}

model Conversation {
  id        String    @id @default(cuid())
  title     String?
  userId    String
  user      User      @relation(fields: [userId], references: [id])
  messages  Message[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([userId])
}

model Message {
  id             String       @id @default(cuid())
  role           String
  content        String
  sources        Json?
  conversationId String
  conversation   Conversation @relation(fields: [conversationId], references: [id], onDelete: Cascade)
  createdAt      DateTime     @default(now())

  @@index([conversationId])
}

model Usage {
  id           String   @id @default(cuid())
  userId       String
  user         User     @relation(fields: [userId], references: [id])
  type         String
  inputTokens  Int
  outputTokens Int
  cost         Float
  createdAt    DateTime @default(now())

  @@index([userId, createdAt])
}
```

---

## Day 6: Core Services Implementation

### 1. Document Processing Service

```typescript
// src/lib/services/document-processor.ts
import pdf from 'pdf-parse';
import mammoth from 'mammoth';
import OpenAI from 'openai';
import { Pinecone } from '@pinecone-database/pinecone';

const openai = new OpenAI();
const pinecone = new Pinecone();

interface ProcessingResult {
  chunkCount: number;
  status: 'success' | 'error';
  error?: string;
}

interface Chunk {
  content: string;
  metadata: {
    documentId: string;
    documentName: string;
    chunkIndex: number;
    userId: string;
  };
}

// Extract text from different file types
async function extractText(
  buffer: Buffer,
  mimeType: string
): Promise<string> {
  switch (mimeType) {
    case 'application/pdf': {
      const data = await pdf(buffer);
      return data.text;
    }
    case 'application/vnd.openxmlformats-officedocument.wordprocessingml.document': {
      const result = await mammoth.extractRawText({ buffer });
      return result.value;
    }
    case 'text/plain':
    case 'text/markdown':
      return buffer.toString('utf-8');
    default:
      throw new Error(`Unsupported file type: ${mimeType}`);
  }
}

// Smart chunking that respects document structure
function chunkText(text: string, maxChunkSize = 1000, overlap = 200): string[] {
  const paragraphs = text.split(/\n\n+/);
  const chunks: string[] = [];
  let currentChunk = '';

  for (const paragraph of paragraphs) {
    const trimmed = paragraph.trim();
    if (!trimmed) continue;

    if (currentChunk.length + trimmed.length + 2 <= maxChunkSize) {
      currentChunk += (currentChunk ? '\n\n' : '') + trimmed;
    } else {
      if (currentChunk) {
        chunks.push(currentChunk);
        // Keep overlap from previous chunk
        const words = currentChunk.split(' ');
        const overlapWords = Math.floor(overlap / 5); // Approximate words
        currentChunk = words.slice(-overlapWords).join(' ');
      }

      if (trimmed.length > maxChunkSize) {
        // Split long paragraphs by sentences
        const sentences = trimmed.match(/[^.!?]+[.!?]+/g) || [trimmed];
        for (const sentence of sentences) {
          if (currentChunk.length + sentence.length <= maxChunkSize) {
            currentChunk += (currentChunk ? ' ' : '') + sentence.trim();
          } else {
            if (currentChunk) chunks.push(currentChunk);
            currentChunk = sentence.trim();
          }
        }
      } else {
        currentChunk += (currentChunk ? '\n\n' : '') + trimmed;
      }
    }
  }

  if (currentChunk) {
    chunks.push(currentChunk);
  }

  return chunks;
}

// Generate embeddings in batches
async function generateEmbeddings(texts: string[]): Promise<number[][]> {
  const batchSize = 100;
  const embeddings: number[][] = [];

  for (let i = 0; i < texts.length; i += batchSize) {
    const batch = texts.slice(i, i + batchSize);

    const response = await openai.embeddings.create({
      model: 'text-embedding-3-small',
      input: batch,
    });

    embeddings.push(...response.data.map(d => d.embedding));
  }

  return embeddings;
}

// Main document processing function
export async function processDocument(
  file: Buffer,
  mimeType: string,
  documentId: string,
  documentName: string,
  userId: string,
  onProgress?: (progress: number, status: string) => void
): Promise<ProcessingResult> {
  try {
    // Step 1: Extract text
    onProgress?.(10, 'Extracting text...');
    const text = await extractText(file, mimeType);

    if (!text.trim()) {
      return { chunkCount: 0, status: 'error', error: 'No text content found' };
    }

    // Step 2: Chunk text
    onProgress?.(30, 'Chunking document...');
    const chunks = chunkText(text);

    // Step 3: Generate embeddings
    onProgress?.(50, 'Generating embeddings...');
    const embeddings = await generateEmbeddings(chunks);

    // Step 4: Store in Pinecone
    onProgress?.(80, 'Storing in vector database...');
    const index = pinecone.index(process.env.PINECONE_INDEX!);

    const vectors = chunks.map((content, i) => ({
      id: `${documentId}-${i}`,
      values: embeddings[i],
      metadata: {
        content,
        documentId,
        documentName,
        chunkIndex: i,
        userId,
      },
    }));

    // Upsert in batches
    const batchSize = 100;
    for (let i = 0; i < vectors.length; i += batchSize) {
      await index.upsert(vectors.slice(i, i + batchSize));
    }

    onProgress?.(100, 'Complete');
    return { chunkCount: chunks.length, status: 'success' };
  } catch (error) {
    console.error('Document processing error:', error);
    return {
      chunkCount: 0,
      status: 'error',
      error: error instanceof Error ? error.message : 'Unknown error',
    };
  }
}

// Delete document from vector database
export async function deleteDocument(documentId: string): Promise<void> {
  const index = pinecone.index(process.env.PINECONE_INDEX!);

  // Delete all chunks for this document
  await index.deleteMany({
    filter: { documentId: { $eq: documentId } },
  });
}
```

### 2. RAG Chat Service

```typescript
// src/lib/services/chat-service.ts
import OpenAI from 'openai';
import { Pinecone } from '@pinecone-database/pinecone';

const openai = new OpenAI();
const pinecone = new Pinecone();

interface Message {
  role: 'user' | 'assistant' | 'system';
  content: string;
}

interface Source {
  content: string;
  documentName: string;
  similarity: number;
}

interface ChatResponse {
  content: string;
  sources: Source[];
  usage: {
    inputTokens: number;
    outputTokens: number;
  };
}

// Retrieve relevant context
async function retrieveContext(
  query: string,
  userId: string,
  topK = 5
): Promise<Source[]> {
  // Generate query embedding
  const embeddingResponse = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: query,
  });

  const queryEmbedding = embeddingResponse.data[0].embedding;

  // Search Pinecone
  const index = pinecone.index(process.env.PINECONE_INDEX!);

  const results = await index.query({
    vector: queryEmbedding,
    topK,
    includeMetadata: true,
    filter: { userId: { $eq: userId } },
  });

  return results.matches?.map(match => ({
    content: match.metadata?.content as string,
    documentName: match.metadata?.documentName as string,
    similarity: match.score || 0,
  })) || [];
}

// Generate chat response with RAG
export async function generateChatResponse(
  messages: Message[],
  userId: string
): Promise<ChatResponse> {
  const userMessage = messages[messages.length - 1].content;

  // Retrieve relevant context
  const sources = await retrieveContext(userMessage, userId);

  // Build context string
  const contextText = sources.length > 0
    ? sources.map((s, i) => `[Source ${i + 1}: ${s.documentName}]\n${s.content}`).join('\n\n')
    : 'No relevant documents found.';

  // Build messages with context
  const systemMessage: Message = {
    role: 'system',
    content: `You are a helpful document assistant. Answer questions based on the provided context from the user's documents.

Guidelines:
- If the answer is in the context, provide it with citations like [Source 1]
- If the context doesn't contain the answer, say so clearly
- Be concise but thorough
- If asked about something not in the documents, explain what information is available

Context from user's documents:
${contextText}`,
  };

  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [systemMessage, ...messages],
    temperature: 0.7,
    max_tokens: 1000,
  });

  return {
    content: response.choices[0].message.content || '',
    sources: sources.filter(s => s.similarity > 0.7),
    usage: {
      inputTokens: response.usage?.prompt_tokens || 0,
      outputTokens: response.usage?.completion_tokens || 0,
    },
  };
}

// Streaming version
export async function* streamChatResponse(
  messages: Message[],
  userId: string
): AsyncGenerator<{ type: 'text' | 'sources' | 'done'; data?: string | Source[] }> {
  const userMessage = messages[messages.length - 1].content;

  // Retrieve relevant context
  const sources = await retrieveContext(userMessage, userId);

  const contextText = sources.length > 0
    ? sources.map((s, i) => `[Source ${i + 1}: ${s.documentName}]\n${s.content}`).join('\n\n')
    : 'No relevant documents found.';

  const systemMessage: Message = {
    role: 'system',
    content: `You are a helpful document assistant. Answer questions based on the provided context.

Context:
${contextText}

Cite sources as [Source N] when using information from the context.`,
  };

  const stream = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [systemMessage, ...messages],
    temperature: 0.7,
    stream: true,
  });

  for await (const chunk of stream) {
    const text = chunk.choices[0]?.delta?.content;
    if (text) {
      yield { type: 'text', data: text };
    }
  }

  yield { type: 'sources', data: sources.filter(s => s.similarity > 0.7) };
  yield { type: 'done' };
}

// Generate conversation title from first message
export async function generateConversationTitle(
  firstMessage: string
): Promise<string> {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [
      {
        role: 'system',
        content: 'Generate a short (3-5 words) title for a conversation that starts with this message. Return only the title, no quotes.',
      },
      { role: 'user', content: firstMessage },
    ],
    temperature: 0.7,
    max_tokens: 20,
  });

  return response.choices[0].message.content || 'New Conversation';
}
```

### 3. API Routes

```typescript
// src/app/api/documents/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { processDocument } from '@/lib/services/document-processor';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const formData = await request.formData();
    const file = formData.get('file') as File;

    if (!file) {
      return NextResponse.json({ error: 'No file provided' }, { status: 400 });
    }

    // Create document record
    const document = await prisma.document.create({
      data: {
        name: file.name,
        type: file.type,
        size: file.size,
        chunkCount: 0,
        status: 'processing',
        userId: session.user.id,
      },
    });

    // Process document asynchronously
    const buffer = Buffer.from(await file.arrayBuffer());

    processDocument(
      buffer,
      file.type,
      document.id,
      file.name,
      session.user.id
    ).then(async (result) => {
      await prisma.document.update({
        where: { id: document.id },
        data: {
          chunkCount: result.chunkCount,
          status: result.status,
          metadata: result.error ? { error: result.error } : {},
        },
      });
    });

    return NextResponse.json({
      id: document.id,
      status: 'processing',
    });
  } catch (error) {
    console.error('Upload error:', error);
    return NextResponse.json(
      { error: 'Upload failed' },
      { status: 500 }
    );
  }
}

// src/app/api/documents/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { deleteDocument } from '@/lib/services/document-processor';
import { getServerSession } from 'next-auth';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession();
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const document = await prisma.document.findFirst({
    where: {
      id: params.id,
      userId: session.user.id,
    },
  });

  if (!document) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  return NextResponse.json(document);
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession();
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const document = await prisma.document.findFirst({
    where: {
      id: params.id,
      userId: session.user.id,
    },
  });

  if (!document) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  // Delete from vector database
  await deleteDocument(document.id);

  // Delete from database
  await prisma.document.delete({
    where: { id: document.id },
  });

  return NextResponse.json({ success: true });
}

// src/app/api/chat/route.ts
import { NextRequest } from 'next/server';
import { streamChatResponse } from '@/lib/services/chat-service';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return new Response('Unauthorized', { status: 401 });
    }

    const { messages, conversationId } = await request.json();

    // Create or get conversation
    let conversation;
    if (conversationId) {
      conversation = await prisma.conversation.findFirst({
        where: {
          id: conversationId,
          userId: session.user.id,
        },
      });
    }

    if (!conversation) {
      conversation = await prisma.conversation.create({
        data: {
          userId: session.user.id,
        },
      });
    }

    // Save user message
    const userMessage = messages[messages.length - 1];
    await prisma.message.create({
      data: {
        role: 'user',
        content: userMessage.content,
        conversationId: conversation.id,
      },
    });

    // Stream response
    const encoder = new TextEncoder();
    let fullContent = '';
    let sources: any[] = [];

    const stream = new ReadableStream({
      async start(controller) {
        try {
          for await (const chunk of streamChatResponse(messages, session.user.id)) {
            if (chunk.type === 'text') {
              fullContent += chunk.data;
              controller.enqueue(
                encoder.encode(`data: ${JSON.stringify(chunk)}\n\n`)
              );
            } else if (chunk.type === 'sources') {
              sources = chunk.data as any[];
              controller.enqueue(
                encoder.encode(`data: ${JSON.stringify(chunk)}\n\n`)
              );
            } else if (chunk.type === 'done') {
              // Save assistant message
              await prisma.message.create({
                data: {
                  role: 'assistant',
                  content: fullContent,
                  sources: sources,
                  conversationId: conversation.id,
                },
              });

              controller.enqueue(
                encoder.encode(`data: ${JSON.stringify({ type: 'done', conversationId: conversation.id })}\n\n`)
              );
              controller.close();
            }
          }
        } catch (error) {
          controller.enqueue(
            encoder.encode(`data: ${JSON.stringify({ type: 'error', message: 'Stream error' })}\n\n`)
          );
          controller.close();
        }
      },
    });

    return new Response(stream, {
      headers: {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
      },
    });
  } catch (error) {
    console.error('Chat error:', error);
    return new Response('Internal error', { status: 500 });
  }
}
```

---

## Day 7: Frontend Implementation

### 1. State Management

```typescript
// src/lib/store/chat-store.ts
import { create } from 'zustand';

interface Message {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  sources?: Array<{
    content: string;
    documentName: string;
    similarity: number;
  }>;
  isStreaming?: boolean;
}

interface Conversation {
  id: string;
  title?: string;
  messages: Message[];
  createdAt: Date;
}

interface ChatStore {
  conversations: Conversation[];
  currentConversationId: string | null;
  isLoading: boolean;

  // Actions
  setCurrentConversation: (id: string | null) => void;
  addMessage: (message: Message) => void;
  updateMessage: (id: string, updates: Partial<Message>) => void;
  appendToMessage: (id: string, text: string) => void;
  createConversation: () => string;
  setLoading: (loading: boolean) => void;
  loadConversations: () => Promise<void>;
}

export const useChatStore = create<ChatStore>((set, get) => ({
  conversations: [],
  currentConversationId: null,
  isLoading: false,

  setCurrentConversation: (id) => {
    set({ currentConversationId: id });
  },

  addMessage: (message) => {
    set((state) => {
      const conversations = [...state.conversations];
      const convIndex = conversations.findIndex(
        c => c.id === state.currentConversationId
      );

      if (convIndex >= 0) {
        conversations[convIndex] = {
          ...conversations[convIndex],
          messages: [...conversations[convIndex].messages, message],
        };
      }

      return { conversations };
    });
  },

  updateMessage: (id, updates) => {
    set((state) => {
      const conversations = state.conversations.map(conv => ({
        ...conv,
        messages: conv.messages.map(msg =>
          msg.id === id ? { ...msg, ...updates } : msg
        ),
      }));
      return { conversations };
    });
  },

  appendToMessage: (id, text) => {
    set((state) => {
      const conversations = state.conversations.map(conv => ({
        ...conv,
        messages: conv.messages.map(msg =>
          msg.id === id ? { ...msg, content: msg.content + text } : msg
        ),
      }));
      return { conversations };
    });
  },

  createConversation: () => {
    const id = crypto.randomUUID();
    set((state) => ({
      conversations: [
        {
          id,
          messages: [],
          createdAt: new Date(),
        },
        ...state.conversations,
      ],
      currentConversationId: id,
    }));
    return id;
  },

  setLoading: (loading) => {
    set({ isLoading: loading });
  },

  loadConversations: async () => {
    const response = await fetch('/api/conversations');
    const data = await response.json();
    set({ conversations: data });
  },
}));
```

### 2. Chat Interface Component

```tsx
// src/components/chat/ChatInterface.tsx
'use client';

import { useState, useRef, useEffect, useCallback } from 'react';
import { useChatStore } from '@/lib/store/chat-store';
import { Send, Loader2, FileText } from 'lucide-react';
import { cn } from '@/lib/utils';

export function ChatInterface() {
  const [input, setInput] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const inputRef = useRef<HTMLTextAreaElement>(null);

  const {
    conversations,
    currentConversationId,
    isLoading,
    addMessage,
    appendToMessage,
    updateMessage,
    createConversation,
    setLoading,
  } = useChatStore();

  const currentConversation = conversations.find(
    c => c.id === currentConversationId
  );

  const scrollToBottom = useCallback(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, []);

  useEffect(() => {
    scrollToBottom();
  }, [currentConversation?.messages, scrollToBottom]);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!input.trim() || isLoading) return;

    const userInput = input.trim();
    setInput('');

    // Create conversation if needed
    let convId = currentConversationId;
    if (!convId) {
      convId = createConversation();
    }

    // Add user message
    const userMessageId = crypto.randomUUID();
    addMessage({
      id: userMessageId,
      role: 'user',
      content: userInput,
    });

    // Add placeholder assistant message
    const assistantMessageId = crypto.randomUUID();
    addMessage({
      id: assistantMessageId,
      role: 'assistant',
      content: '',
      isStreaming: true,
    });

    setLoading(true);

    try {
      const messages = [
        ...(currentConversation?.messages || []).map(m => ({
          role: m.role,
          content: m.content,
        })),
        { role: 'user' as const, content: userInput },
      ];

      const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          messages,
          conversationId: convId,
        }),
      });

      const reader = response.body?.getReader();
      const decoder = new TextDecoder();

      while (reader) {
        const { done, value } = await reader.read();
        if (done) break;

        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');

        for (const line of lines) {
          if (line.startsWith('data: ')) {
            try {
              const data = JSON.parse(line.slice(6));

              if (data.type === 'text') {
                appendToMessage(assistantMessageId, data.data);
              } else if (data.type === 'sources') {
                updateMessage(assistantMessageId, { sources: data.data });
              } else if (data.type === 'done') {
                updateMessage(assistantMessageId, { isStreaming: false });
              }
            } catch (e) {
              // Skip invalid JSON
            }
          }
        }
      }
    } catch (error) {
      updateMessage(assistantMessageId, {
        content: 'Sorry, an error occurred. Please try again.',
        isStreaming: false,
      });
    } finally {
      setLoading(false);
    }
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSubmit(e);
    }
  };

  return (
    <div className="flex flex-col h-full">
      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {!currentConversation?.messages.length && (
          <div className="text-center text-gray-500 mt-20">
            <FileText className="w-12 h-12 mx-auto mb-4 opacity-50" />
            <p className="text-lg font-medium">Start a conversation</p>
            <p className="text-sm">
              Ask questions about your uploaded documents
            </p>
          </div>
        )}

        {currentConversation?.messages.map((message) => (
          <MessageBubble key={message.id} message={message} />
        ))}

        <div ref={messagesEndRef} />
      </div>

      {/* Input */}
      <div className="border-t p-4">
        <form onSubmit={handleSubmit} className="flex gap-2">
          <textarea
            ref={inputRef}
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyDown={handleKeyDown}
            placeholder="Ask about your documents..."
            rows={1}
            className="flex-1 resize-none rounded-lg border px-4 py-3 focus:outline-none focus:ring-2 focus:ring-blue-500"
            disabled={isLoading}
          />
          <button
            type="submit"
            disabled={isLoading || !input.trim()}
            className="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {isLoading ? (
              <Loader2 className="w-5 h-5 animate-spin" />
            ) : (
              <Send className="w-5 h-5" />
            )}
          </button>
        </form>
      </div>
    </div>
  );
}

// Message bubble component
function MessageBubble({ message }: { message: any }) {
  const isUser = message.role === 'user';

  return (
    <div className={cn('flex', isUser ? 'justify-end' : 'justify-start')}>
      <div
        className={cn(
          'max-w-[80%] rounded-lg px-4 py-3',
          isUser
            ? 'bg-blue-500 text-white'
            : 'bg-gray-100 text-gray-900'
        )}
      >
        <div className="whitespace-pre-wrap">{message.content}</div>

        {message.isStreaming && (
          <span className="inline-block w-2 h-4 bg-gray-400 animate-pulse ml-1" />
        )}

        {message.sources?.length > 0 && (
          <div className="mt-3 pt-3 border-t border-gray-200">
            <p className="text-xs font-medium mb-2 text-gray-600">Sources:</p>
            <div className="space-y-2">
              {message.sources.map((source: any, i: number) => (
                <div
                  key={i}
                  className="text-xs p-2 bg-white rounded border border-gray-200"
                >
                  <p className="font-medium text-gray-700">
                    {source.documentName}
                  </p>
                  <p className="text-gray-500 mt-1 line-clamp-2">
                    {source.content}
                  </p>
                </div>
              ))}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

### 3. Document Upload Component

```tsx
// src/components/documents/DocumentUpload.tsx
'use client';

import { useState, useCallback } from 'react';
import { useDropzone } from 'react-dropzone';
import { Upload, File, Check, X, Loader2 } from 'lucide-react';
import { cn } from '@/lib/utils';

interface UploadingFile {
  id: string;
  name: string;
  status: 'uploading' | 'processing' | 'complete' | 'error';
  progress: number;
  error?: string;
}

export function DocumentUpload({ onUploadComplete }: { onUploadComplete?: () => void }) {
  const [uploadingFiles, setUploadingFiles] = useState<UploadingFile[]>([]);

  const uploadFile = async (file: File) => {
    const id = crypto.randomUUID();

    setUploadingFiles(prev => [
      ...prev,
      { id, name: file.name, status: 'uploading', progress: 0 },
    ]);

    try {
      const formData = new FormData();
      formData.append('file', file);

      const response = await fetch('/api/documents/upload', {
        method: 'POST',
        body: formData,
      });

      if (!response.ok) {
        throw new Error('Upload failed');
      }

      const data = await response.json();

      // Poll for processing status
      setUploadingFiles(prev =>
        prev.map(f =>
          f.id === id ? { ...f, status: 'processing', progress: 50 } : f
        )
      );

      // Poll until complete
      let attempts = 0;
      while (attempts < 60) { // Max 2 minutes
        await new Promise(resolve => setTimeout(resolve, 2000));

        const statusResponse = await fetch(`/api/documents/${data.id}`);
        const statusData = await statusResponse.json();

        if (statusData.status === 'success') {
          setUploadingFiles(prev =>
            prev.map(f =>
              f.id === id ? { ...f, status: 'complete', progress: 100 } : f
            )
          );
          onUploadComplete?.();
          break;
        } else if (statusData.status === 'error') {
          throw new Error(statusData.metadata?.error || 'Processing failed');
        }

        attempts++;
      }
    } catch (error) {
      setUploadingFiles(prev =>
        prev.map(f =>
          f.id === id
            ? {
                ...f,
                status: 'error',
                error: error instanceof Error ? error.message : 'Upload failed',
              }
            : f
        )
      );
    }
  };

  const onDrop = useCallback((acceptedFiles: File[]) => {
    acceptedFiles.forEach(file => uploadFile(file));
  }, []);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: {
      'application/pdf': ['.pdf'],
      'text/plain': ['.txt'],
      'text/markdown': ['.md'],
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
    },
  });

  const removeFile = (id: string) => {
    setUploadingFiles(prev => prev.filter(f => f.id !== id));
  };

  return (
    <div className="space-y-4">
      <div
        {...getRootProps()}
        className={cn(
          'border-2 border-dashed rounded-lg p-8 text-center cursor-pointer transition-colors',
          isDragActive
            ? 'border-blue-500 bg-blue-50'
            : 'border-gray-300 hover:border-gray-400'
        )}
      >
        <input {...getInputProps()} />
        <Upload className="w-10 h-10 mx-auto mb-4 text-gray-400" />
        {isDragActive ? (
          <p className="text-blue-500">Drop files here...</p>
        ) : (
          <>
            <p className="text-gray-600 mb-1">
              Drag & drop documents here, or click to select
            </p>
            <p className="text-sm text-gray-400">
              Supports PDF, TXT, MD, DOCX
            </p>
          </>
        )}
      </div>

      {uploadingFiles.length > 0 && (
        <div className="space-y-2">
          {uploadingFiles.map(file => (
            <div
              key={file.id}
              className="flex items-center gap-3 p-3 bg-gray-50 rounded-lg"
            >
              <File className="w-5 h-5 text-gray-400" />
              <div className="flex-1 min-w-0">
                <p className="text-sm font-medium truncate">{file.name}</p>
                <div className="flex items-center gap-2 mt-1">
                  {file.status === 'uploading' && (
                    <>
                      <div className="flex-1 h-1 bg-gray-200 rounded-full overflow-hidden">
                        <div
                          className="h-full bg-blue-500 transition-all"
                          style={{ width: `${file.progress}%` }}
                        />
                      </div>
                      <span className="text-xs text-gray-500">Uploading...</span>
                    </>
                  )}
                  {file.status === 'processing' && (
                    <>
                      <Loader2 className="w-3 h-3 animate-spin text-blue-500" />
                      <span className="text-xs text-blue-500">Processing...</span>
                    </>
                  )}
                  {file.status === 'complete' && (
                    <>
                      <Check className="w-3 h-3 text-green-500" />
                      <span className="text-xs text-green-500">Complete</span>
                    </>
                  )}
                  {file.status === 'error' && (
                    <>
                      <X className="w-3 h-3 text-red-500" />
                      <span className="text-xs text-red-500">{file.error}</span>
                    </>
                  )}
                </div>
              </div>
              <button
                onClick={() => removeFile(file.id)}
                className="p-1 hover:bg-gray-200 rounded"
              >
                <X className="w-4 h-4 text-gray-400" />
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### 4. Document List Component

```tsx
// src/components/documents/DocumentList.tsx
'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { File, Trash2, Loader2 } from 'lucide-react';
import { formatDistanceToNow } from 'date-fns';

interface Document {
  id: string;
  name: string;
  type: string;
  size: number;
  chunkCount: number;
  status: string;
  createdAt: string;
}

export function DocumentList() {
  const queryClient = useQueryClient();

  const { data: documents, isLoading } = useQuery<Document[]>({
    queryKey: ['documents'],
    queryFn: async () => {
      const response = await fetch('/api/documents');
      return response.json();
    },
  });

  const deleteMutation = useMutation({
    mutationFn: async (id: string) => {
      await fetch(`/api/documents/${id}`, { method: 'DELETE' });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['documents'] });
    },
  });

  const formatFileSize = (bytes: number) => {
    if (bytes < 1024) return `${bytes} B`;
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
    return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
  };

  if (isLoading) {
    return (
      <div className="flex items-center justify-center py-8">
        <Loader2 className="w-6 h-6 animate-spin text-gray-400" />
      </div>
    );
  }

  if (!documents?.length) {
    return (
      <div className="text-center py-8 text-gray-500">
        <File className="w-10 h-10 mx-auto mb-3 opacity-50" />
        <p>No documents uploaded yet</p>
      </div>
    );
  }

  return (
    <div className="space-y-2">
      {documents.map(doc => (
        <div
          key={doc.id}
          className="flex items-center gap-3 p-3 bg-white border rounded-lg hover:bg-gray-50"
        >
          <File className="w-5 h-5 text-blue-500" />
          <div className="flex-1 min-w-0">
            <p className="font-medium truncate">{doc.name}</p>
            <p className="text-sm text-gray-500">
              {formatFileSize(doc.size)} • {doc.chunkCount} chunks •{' '}
              {formatDistanceToNow(new Date(doc.createdAt), { addSuffix: true })}
            </p>
          </div>
          <div className="flex items-center gap-2">
            {doc.status === 'processing' && (
              <Loader2 className="w-4 h-4 animate-spin text-blue-500" />
            )}
            <button
              onClick={() => deleteMutation.mutate(doc.id)}
              disabled={deleteMutation.isPending}
              className="p-2 text-gray-400 hover:text-red-500 hover:bg-red-50 rounded"
            >
              <Trash2 className="w-4 h-4" />
            </button>
          </div>
        </div>
      ))}
    </div>
  );
}
```

### 5. Main Application Layout

```tsx
// src/app/page.tsx
import { ChatInterface } from '@/components/chat/ChatInterface';
import { DocumentUpload } from '@/components/documents/DocumentUpload';
import { DocumentList } from '@/components/documents/DocumentList';
import { ConversationSidebar } from '@/components/chat/ConversationSidebar';

export default function Home() {
  return (
    <div className="flex h-screen bg-gray-50">
      {/* Sidebar */}
      <aside className="w-64 bg-white border-r flex flex-col">
        <div className="p-4 border-b">
          <h1 className="text-xl font-bold">Doc Assistant</h1>
        </div>
        <div className="flex-1 overflow-y-auto">
          <ConversationSidebar />
        </div>
      </aside>

      {/* Main content */}
      <main className="flex-1 flex">
        {/* Chat area */}
        <div className="flex-1 flex flex-col">
          <ChatInterface />
        </div>

        {/* Documents panel */}
        <aside className="w-80 border-l bg-white flex flex-col">
          <div className="p-4 border-b">
            <h2 className="font-semibold">Documents</h2>
          </div>
          <div className="p-4">
            <DocumentUpload />
          </div>
          <div className="flex-1 overflow-y-auto p-4">
            <DocumentList />
          </div>
        </aside>
      </main>
    </div>
  );
}
```

---

## Deployment Checklist

### Environment Variables

```bash
# Production environment variables
OPENAI_API_KEY=sk-prod-...
PINECONE_API_KEY=...
PINECONE_INDEX=doc-assistant-prod
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=...
NEXTAUTH_URL=https://your-domain.com
```

### Rate Limiting

```typescript
// src/middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
});

export async function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/api/')) {
    const ip = request.ip ?? '127.0.0.1';
    const { success, limit, reset, remaining } = await ratelimit.limit(ip);

    if (!success) {
      return NextResponse.json(
        { error: 'Too many requests' },
        {
          status: 429,
          headers: {
            'X-RateLimit-Limit': limit.toString(),
            'X-RateLimit-Remaining': remaining.toString(),
            'X-RateLimit-Reset': reset.toString(),
          },
        }
      );
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: '/api/:path*',
};
```

### Error Monitoring

```typescript
// src/lib/monitoring.ts
import * as Sentry from '@sentry/nextjs';

export function captureError(error: Error, context?: Record<string, any>) {
  console.error(error);

  Sentry.captureException(error, {
    extra: context,
  });
}

export function trackEvent(name: string, properties?: Record<string, any>) {
  // PostHog or your analytics provider
  if (typeof window !== 'undefined' && (window as any).posthog) {
    (window as any).posthog.capture(name, properties);
  }
}
```

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Document Assistant - Final Architecture              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  FRONTEND                                                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Next.js App Router                                              │   │
│  │  • React Components (Chat, Upload, Documents)                    │   │
│  │  • Zustand State Management                                      │   │
│  │  • TanStack Query for Server State                               │   │
│  │  • Tailwind CSS Styling                                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  API LAYER                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Next.js API Routes                                              │   │
│  │  • /api/documents/* - Upload, List, Delete                       │   │
│  │  • /api/chat - Streaming RAG responses                           │   │
│  │  • /api/conversations/* - History management                     │   │
│  │  • Rate limiting via Upstash                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  SERVICE LAYER                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Document Processor: Extract, chunk, embed, store              │   │
│  │  • Chat Service: RAG retrieval + LLM generation                  │   │
│  │  • Usage Tracker: Token counting, cost calculation               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  DATA LAYER                                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  PostgreSQL (Prisma)     │  Pinecone                             │   │
│  │  • Users                 │  • Document chunks                    │   │
│  │  • Documents metadata    │  • Embeddings                         │   │
│  │  • Conversations         │  • Similarity search                  │   │
│  │  • Messages              │                                       │   │
│  │  • Usage records         │                                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  EXTERNAL SERVICES                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  OpenAI API              │  Monitoring                           │   │
│  │  • text-embedding-3-small│  • Sentry (errors)                    │   │
│  │  • gpt-4o (chat)         │  • PostHog (analytics)                │   │
│  │  • gpt-4o-mini (titles)  │  • Upstash (rate limiting)            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    AI Application Best Practices                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. ARCHITECTURE                                                        │
│     • Separate concerns: processing, retrieval, generation              │
│     • Use streaming for better UX                                       │
│     • Implement proper error handling at every layer                    │
│                                                                         │
│  2. DOCUMENT PROCESSING                                                 │
│     • Async processing with status polling                              │
│     • Smart chunking respects document structure                        │
│     • Batch operations for efficiency                                   │
│                                                                         │
│  3. RAG IMPLEMENTATION                                                  │
│     • Filter by user for multi-tenancy                                  │
│     • Include source citations for transparency                         │
│     • Threshold filtering for relevance                                 │
│                                                                         │
│  4. USER EXPERIENCE                                                     │
│     • Streaming responses feel faster                                   │
│     • Show processing status and progress                               │
│     • Display sources to build trust                                    │
│                                                                         │
│  5. PRODUCTION READINESS                                                │
│     • Rate limiting protects resources                                  │
│     • Error monitoring catches issues early                             │
│     • Usage tracking enables cost management                            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Extensions and Improvements

### Ideas for Further Development

1. **Multi-model support**: Add Anthropic Claude as an alternative
2. **Collaborative features**: Share documents and conversations
3. **Advanced retrieval**: Implement hybrid search and re-ranking
4. **Document versioning**: Track document updates over time
5. **Export functionality**: Export conversations as PDF/Markdown
6. **Mobile app**: React Native version for mobile access
7. **Browser extension**: Query documents from any webpage
8. **Slack/Teams integration**: Access documents from chat apps

---

## Congratulations!

You've built a production-ready AI-powered document assistant! This project demonstrates:

- Modern React patterns with Next.js App Router
- Streaming AI responses for excellent UX
- RAG implementation with vector search
- Proper state management and data fetching
- Production considerations like rate limiting and monitoring

Use this as a foundation for building more sophisticated AI applications. The patterns you've learned apply to chatbots, search engines, content generation tools, and more.

---

## Additional Resources

- [Vercel AI SDK Documentation](https://sdk.vercel.ai/docs)
- [Pinecone Best Practices](https://docs.pinecone.io/guides/get-started/overview)
- [OpenAI Production Guide](https://platform.openai.com/docs/guides/production-best-practices)
- [Next.js Documentation](https://nextjs.org/docs)
- [Prisma Documentation](https://www.prisma.io/docs)
