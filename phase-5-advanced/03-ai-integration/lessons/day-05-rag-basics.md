# Day 5: RAG (Retrieval-Augmented Generation) Basics

## Introduction

Retrieval-Augmented Generation (RAG) is a technique that enhances AI responses by giving models access to external knowledge. Instead of relying solely on training data, RAG retrieves relevant information from your own documents, databases, or knowledge bases to generate more accurate, up-to-date, and contextually relevant responses.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         RAG Architecture                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    ┌──────────┐      ┌─────────────────┐      ┌──────────────────┐     │
│    │  User    │      │    Embedding    │      │   Vector Store   │     │
│    │  Query   │─────▶│     Model       │─────▶│   (Similarity    │     │
│    └──────────┘      └─────────────────┘      │    Search)       │     │
│                                               └────────┬─────────┘     │
│                                                        │               │
│                                                        ▼               │
│    ┌──────────┐      ┌─────────────────┐      ┌──────────────────┐     │
│    │ Response │◀─────│      LLM        │◀─────│   Retrieved      │     │
│    │          │      │   Generation    │      │   Context        │     │
│    └──────────┘      └─────────────────┘      └──────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

By the end of this lesson, you will:
- Understand vector embeddings and semantic search
- Set up a vector database for storing embeddings
- Implement document chunking strategies
- Build a complete RAG pipeline
- Handle different document types
- Optimize retrieval quality

---

## 1. Understanding Embeddings

### What Are Embeddings?

Embeddings are numerical representations (vectors) of text that capture semantic meaning. Similar texts have similar embeddings, enabling semantic search.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Embedding Space Visualization                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                          "programming"                                  │
│                               ●                                         │
│                              /                                          │
│                  "coding"   /    "JavaScript"                           │
│                      ●─────●─────────●                                  │
│                            \          \                                 │
│                             \          "TypeScript"                     │
│                              \              ●                           │
│                               \                                         │
│                                "software"                               │
│                                    ●                                    │
│                                                                         │
│                                                                         │
│       "cooking"                         "music"                         │
│           ●                                 ●                           │
│          /                                 /                            │
│    "recipe"                         "guitar"                            │
│        ●                                ●                               │
│                                                                         │
│   Similar concepts cluster together in embedding space                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Generating Embeddings

```javascript
// lib/embeddings.js
import OpenAI from 'openai';

const openai = new OpenAI();

// Generate embedding for a single text
export async function generateEmbedding(text) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: text,
  });

  return response.data[0].embedding;
}

// Generate embeddings for multiple texts (batch)
export async function generateEmbeddings(texts) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: texts,
  });

  return response.data.map(item => item.embedding);
}

// Calculate cosine similarity between two vectors
export function cosineSimilarity(a, b) {
  let dotProduct = 0;
  let normA = 0;
  let normB = 0;

  for (let i = 0; i < a.length; i++) {
    dotProduct += a[i] * b[i];
    normA += a[i] * a[i];
    normB += b[i] * b[i];
  }

  return dotProduct / (Math.sqrt(normA) * Math.sqrt(normB));
}

// Example: Find semantic similarity
async function demonstrateSimilarity() {
  const texts = [
    "How do I reset my password?",
    "I forgot my login credentials",
    "What's the weather today?",
  ];

  const embeddings = await generateEmbeddings(texts);

  // Compare first two (should be similar)
  const sim1 = cosineSimilarity(embeddings[0], embeddings[1]);
  console.log(`"password reset" vs "forgot credentials": ${sim1.toFixed(3)}`);
  // Output: ~0.85 (high similarity)

  // Compare first and third (should be different)
  const sim2 = cosineSimilarity(embeddings[0], embeddings[2]);
  console.log(`"password reset" vs "weather": ${sim2.toFixed(3)}`);
  // Output: ~0.15 (low similarity)
}
```

### Embedding Models Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Embedding Models Comparison                          │
├──────────────────────┬─────────────┬─────────────┬─────────────────────┤
│ Model                │ Dimensions  │ Cost/1M     │ Best For            │
├──────────────────────┼─────────────┼─────────────┼─────────────────────┤
│ text-embedding-3-    │ 1536        │ $0.02       │ General purpose,    │
│ small                │             │             │ cost-effective      │
├──────────────────────┼─────────────┼─────────────┼─────────────────────┤
│ text-embedding-3-    │ 3072        │ $0.13       │ High accuracy,      │
│ large                │             │             │ complex queries     │
├──────────────────────┼─────────────┼─────────────┼─────────────────────┤
│ voyage-3             │ 1024        │ $0.06       │ Code, multilingual  │
├──────────────────────┼─────────────┼─────────────┼─────────────────────┤
│ nomic-embed-text     │ 768         │ Free (local)│ Privacy-sensitive   │
└──────────────────────┴─────────────┴─────────────┴─────────────────────┘
```

---

## 2. Vector Databases

### Setting Up Pinecone

```javascript
// lib/vectordb/pinecone.js
import { Pinecone } from '@pinecone-database/pinecone';

const pinecone = new Pinecone({
  apiKey: process.env.PINECONE_API_KEY,
});

const INDEX_NAME = 'knowledge-base';

// Initialize index (run once)
export async function initializeIndex() {
  const existingIndexes = await pinecone.listIndexes();

  if (!existingIndexes.indexes?.find(i => i.name === INDEX_NAME)) {
    await pinecone.createIndex({
      name: INDEX_NAME,
      dimension: 1536, // Match embedding model dimensions
      metric: 'cosine',
      spec: {
        serverless: {
          cloud: 'aws',
          region: 'us-east-1',
        },
      },
    });

    console.log('Index created, waiting for it to be ready...');
    await new Promise(resolve => setTimeout(resolve, 60000));
  }

  return pinecone.index(INDEX_NAME);
}

// Get index reference
export function getIndex() {
  return pinecone.index(INDEX_NAME);
}

// Upsert vectors with metadata
export async function upsertVectors(vectors) {
  const index = getIndex();

  // Pinecone allows batches of 100
  const batchSize = 100;
  for (let i = 0; i < vectors.length; i += batchSize) {
    const batch = vectors.slice(i, i + batchSize);
    await index.upsert(batch);
    console.log(`Upserted batch ${Math.floor(i / batchSize) + 1}`);
  }
}

// Query similar vectors
export async function querySimilar(embedding, topK = 5, filter = {}) {
  const index = getIndex();

  const results = await index.query({
    vector: embedding,
    topK,
    includeMetadata: true,
    filter,
  });

  return results.matches;
}

// Delete vectors by ID
export async function deleteVectors(ids) {
  const index = getIndex();
  await index.deleteMany(ids);
}

// Delete all vectors in namespace
export async function clearNamespace(namespace) {
  const index = getIndex();
  await index.namespace(namespace).deleteAll();
}
```

### Using Supabase with pgvector

```javascript
// lib/vectordb/supabase.js
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_SERVICE_KEY
);

// SQL to create the table (run in Supabase SQL editor)
/*
-- Enable pgvector extension
create extension if not exists vector;

-- Create documents table
create table documents (
  id uuid primary key default gen_random_uuid(),
  content text not null,
  metadata jsonb default '{}',
  embedding vector(1536),
  created_at timestamptz default now()
);

-- Create index for similarity search
create index on documents
using ivfflat (embedding vector_cosine_ops)
with (lists = 100);

-- Create similarity search function
create or replace function match_documents(
  query_embedding vector(1536),
  match_threshold float default 0.7,
  match_count int default 5
)
returns table (
  id uuid,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    documents.id,
    documents.content,
    documents.metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where 1 - (documents.embedding <=> query_embedding) > match_threshold
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
*/

// Insert document with embedding
export async function insertDocument(content, embedding, metadata = {}) {
  const { data, error } = await supabase
    .from('documents')
    .insert({
      content,
      embedding,
      metadata,
    })
    .select()
    .single();

  if (error) throw error;
  return data;
}

// Batch insert documents
export async function insertDocuments(documents) {
  const { data, error } = await supabase
    .from('documents')
    .insert(documents)
    .select();

  if (error) throw error;
  return data;
}

// Search similar documents
export async function searchDocuments(embedding, options = {}) {
  const { threshold = 0.7, limit = 5 } = options;

  const { data, error } = await supabase.rpc('match_documents', {
    query_embedding: embedding,
    match_threshold: threshold,
    match_count: limit,
  });

  if (error) throw error;
  return data;
}

// Search with metadata filter
export async function searchWithFilter(embedding, filter, options = {}) {
  const { threshold = 0.7, limit = 5 } = options;

  // Build query with filter
  let query = supabase
    .from('documents')
    .select('id, content, metadata')
    .limit(limit);

  // Apply metadata filters
  for (const [key, value] of Object.entries(filter)) {
    query = query.eq(`metadata->>${key}`, value);
  }

  const { data, error } = await query;
  if (error) throw error;

  // Calculate similarity manually for filtered results
  // (pgvector function doesn't support additional filters easily)
  return data;
}
```

### Architecture Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Vector Database Options                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐     │
│  │    Pinecone     │    │    Supabase     │    │   Chroma/Weaviate│     │
│  │                 │    │   (pgvector)    │    │                 │     │
│  │  • Managed      │    │  • SQL + Vector │    │  • Self-hosted  │     │
│  │  • Serverless   │    │  • All-in-one   │    │  • Open source  │     │
│  │  • Auto-scaling │    │  • Transactions │    │  • Full control │     │
│  │  • High perf    │    │  • Lower cost   │    │  • Privacy      │     │
│  │                 │    │                 │    │                 │     │
│  │  Best for:      │    │  Best for:      │    │  Best for:      │     │
│  │  Production at  │    │  Apps already   │    │  Privacy-first, │     │
│  │  scale          │    │  using Supabase │    │  cost-sensitive │     │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Document Processing

### Text Chunking Strategies

```javascript
// lib/chunking.js

// Simple character-based chunking
export function chunkByCharacters(text, chunkSize = 1000, overlap = 200) {
  const chunks = [];
  let start = 0;

  while (start < text.length) {
    const end = Math.min(start + chunkSize, text.length);
    chunks.push(text.slice(start, end));
    start = end - overlap;
  }

  return chunks;
}

// Sentence-based chunking (better semantic boundaries)
export function chunkBySentences(text, maxChunkSize = 1000, overlap = 1) {
  // Split into sentences
  const sentences = text.match(/[^.!?]+[.!?]+/g) || [text];
  const chunks = [];
  let currentChunk = [];
  let currentSize = 0;

  for (const sentence of sentences) {
    const sentenceSize = sentence.length;

    if (currentSize + sentenceSize > maxChunkSize && currentChunk.length > 0) {
      chunks.push(currentChunk.join(' ').trim());

      // Keep overlap sentences
      const overlapSentences = currentChunk.slice(-overlap);
      currentChunk = overlapSentences;
      currentSize = overlapSentences.join(' ').length;
    }

    currentChunk.push(sentence.trim());
    currentSize += sentenceSize;
  }

  if (currentChunk.length > 0) {
    chunks.push(currentChunk.join(' ').trim());
  }

  return chunks;
}

// Recursive chunking (respects document structure)
export function chunkRecursively(text, maxChunkSize = 1000) {
  const separators = ['\n\n', '\n', '. ', ' ', ''];

  function split(text, separatorIndex) {
    if (text.length <= maxChunkSize) {
      return [text];
    }

    const separator = separators[separatorIndex];

    if (separatorIndex === separators.length - 1) {
      // Last resort: character split
      return chunkByCharacters(text, maxChunkSize, 0);
    }

    const parts = text.split(separator);
    const chunks = [];
    let currentChunk = '';

    for (const part of parts) {
      const potential = currentChunk
        ? currentChunk + separator + part
        : part;

      if (potential.length <= maxChunkSize) {
        currentChunk = potential;
      } else {
        if (currentChunk) {
          chunks.push(currentChunk);
        }

        if (part.length > maxChunkSize) {
          // Recursively split with next separator
          chunks.push(...split(part, separatorIndex + 1));
          currentChunk = '';
        } else {
          currentChunk = part;
        }
      }
    }

    if (currentChunk) {
      chunks.push(currentChunk);
    }

    return chunks;
  }

  return split(text, 0);
}

// Semantic chunking (uses embeddings to find natural breaks)
export async function chunkSemantically(text, generateEmbedding, threshold = 0.5) {
  const sentences = text.match(/[^.!?]+[.!?]+/g) || [text];

  if (sentences.length <= 1) {
    return [text];
  }

  // Generate embeddings for each sentence
  const embeddings = await Promise.all(
    sentences.map(s => generateEmbedding(s))
  );

  // Find semantic breakpoints
  const breakpoints = [0];

  for (let i = 1; i < embeddings.length; i++) {
    const similarity = cosineSimilarity(embeddings[i - 1], embeddings[i]);

    if (similarity < threshold) {
      breakpoints.push(i);
    }
  }

  breakpoints.push(sentences.length);

  // Create chunks from breakpoints
  const chunks = [];
  for (let i = 0; i < breakpoints.length - 1; i++) {
    const chunk = sentences
      .slice(breakpoints[i], breakpoints[i + 1])
      .join(' ')
      .trim();
    chunks.push(chunk);
  }

  return chunks;
}
```

### Processing Different Document Types

```javascript
// lib/document-processors.js
import pdf from 'pdf-parse';
import mammoth from 'mammoth';
import { marked } from 'marked';
import * as cheerio from 'cheerio';

// PDF processing
export async function processPDF(buffer) {
  const data = await pdf(buffer);

  return {
    text: data.text,
    metadata: {
      pages: data.numpages,
      info: data.info,
    },
  };
}

// Word document processing
export async function processDocx(buffer) {
  const result = await mammoth.extractRawText({ buffer });

  return {
    text: result.value,
    metadata: {
      messages: result.messages,
    },
  };
}

// Markdown processing
export function processMarkdown(content) {
  // Convert to HTML then extract text
  const html = marked(content);
  const $ = cheerio.load(html);

  // Extract text while preserving structure
  const text = $('body').text();

  // Extract headers for metadata
  const headers = [];
  $('h1, h2, h3').each((_, el) => {
    headers.push($(el).text());
  });

  return {
    text,
    metadata: {
      headers,
    },
  };
}

// HTML processing
export function processHTML(html) {
  const $ = cheerio.load(html);

  // Remove scripts and styles
  $('script, style, nav, footer, header').remove();

  // Get main content
  const main = $('main, article, .content').first();
  const text = (main.length ? main : $('body')).text();

  // Clean up whitespace
  const cleaned = text.replace(/\s+/g, ' ').trim();

  return {
    text: cleaned,
    metadata: {
      title: $('title').text(),
    },
  };
}

// Universal document processor
export async function processDocument(file, mimeType) {
  const buffer = Buffer.isBuffer(file) ? file : Buffer.from(await file.arrayBuffer());

  switch (mimeType) {
    case 'application/pdf':
      return processPDF(buffer);

    case 'application/vnd.openxmlformats-officedocument.wordprocessingml.document':
      return processDocx(buffer);

    case 'text/markdown':
      return processMarkdown(buffer.toString());

    case 'text/html':
      return processHTML(buffer.toString());

    case 'text/plain':
      return { text: buffer.toString(), metadata: {} };

    default:
      throw new Error(`Unsupported document type: ${mimeType}`);
  }
}
```

---

## 4. Building a RAG Pipeline

### Complete RAG Implementation

```javascript
// lib/rag/pipeline.js
import { generateEmbedding, generateEmbeddings } from '../embeddings';
import { searchDocuments, insertDocuments } from '../vectordb/supabase';
import { chunkRecursively } from '../chunking';
import { processDocument } from '../document-processors';
import OpenAI from 'openai';

const openai = new OpenAI();

// Ingest a document into the knowledge base
export async function ingestDocument(file, mimeType, metadata = {}) {
  console.log('Processing document...');
  const { text, metadata: docMetadata } = await processDocument(file, mimeType);

  console.log('Chunking document...');
  const chunks = chunkRecursively(text, 1000);
  console.log(`Created ${chunks.length} chunks`);

  console.log('Generating embeddings...');
  const embeddings = await generateEmbeddings(chunks);

  console.log('Storing in vector database...');
  const documents = chunks.map((content, i) => ({
    content,
    embedding: embeddings[i],
    metadata: {
      ...metadata,
      ...docMetadata,
      chunkIndex: i,
      totalChunks: chunks.length,
    },
  }));

  const result = await insertDocuments(documents);
  console.log(`Stored ${result.length} chunks`);

  return result;
}

// Retrieve relevant context for a query
export async function retrieveContext(query, options = {}) {
  const { topK = 5, threshold = 0.7 } = options;

  // Generate query embedding
  const queryEmbedding = await generateEmbedding(query);

  // Search for similar documents
  const results = await searchDocuments(queryEmbedding, {
    threshold,
    limit: topK,
  });

  return results;
}

// Generate response with RAG
export async function generateRAGResponse(query, options = {}) {
  const {
    topK = 5,
    threshold = 0.7,
    model = 'gpt-4o',
    systemPrompt = 'You are a helpful assistant that answers questions based on the provided context.',
  } = options;

  // Retrieve relevant context
  const context = await retrieveContext(query, { topK, threshold });

  if (context.length === 0) {
    return {
      answer: "I couldn't find relevant information to answer your question.",
      sources: [],
      context: [],
    };
  }

  // Format context for the prompt
  const contextText = context
    .map((doc, i) => `[${i + 1}] ${doc.content}`)
    .join('\n\n');

  // Generate response
  const response = await openai.chat.completions.create({
    model,
    messages: [
      {
        role: 'system',
        content: `${systemPrompt}

Use the following context to answer questions. If the answer cannot be found in the context, say so.

Context:
${contextText}`,
      },
      {
        role: 'user',
        content: query,
      },
    ],
    temperature: 0.7,
  });

  return {
    answer: response.choices[0].message.content,
    sources: context.map(doc => ({
      content: doc.content.substring(0, 200) + '...',
      similarity: doc.similarity,
      metadata: doc.metadata,
    })),
    context,
  };
}
```

### RAG Pipeline Visualization

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Complete RAG Pipeline                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  INGESTION PHASE                                                        │
│  ───────────────                                                        │
│                                                                         │
│  ┌──────────┐   ┌─────────────┐   ┌─────────────┐   ┌──────────────┐   │
│  │ Document │──▶│   Process   │──▶│    Chunk    │──▶│   Generate   │   │
│  │  Upload  │   │   (Parse)   │   │    Text     │   │  Embeddings  │   │
│  └──────────┘   └─────────────┘   └─────────────┘   └──────┬───────┘   │
│                                                             │           │
│                                                             ▼           │
│                                                    ┌──────────────┐    │
│                                                    │    Store in  │    │
│                                                    │  Vector DB   │    │
│                                                    └──────────────┘    │
│                                                                         │
│  RETRIEVAL PHASE                                                        │
│  ────────────────                                                       │
│                                                                         │
│  ┌──────────┐   ┌─────────────┐   ┌─────────────┐   ┌──────────────┐   │
│  │  User    │──▶│   Generate  │──▶│  Similarity │──▶│   Retrieve   │   │
│  │  Query   │   │  Embedding  │   │   Search    │   │   Top-K      │   │
│  └──────────┘   └─────────────┘   └─────────────┘   └──────┬───────┘   │
│                                                             │           │
│  GENERATION PHASE                                           │           │
│  ─────────────────                                          ▼           │
│                                                    ┌──────────────┐    │
│  ┌──────────┐   ┌─────────────┐                    │   Combine    │    │
│  │ Response │◀──│     LLM     │◀───────────────────│   Query +    │    │
│  │          │   │  Generation │                    │   Context    │    │
│  └──────────┘   └─────────────┘                    └──────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Streaming RAG Responses

```javascript
// lib/rag/streaming.js
import { generateEmbedding } from '../embeddings';
import { searchDocuments } from '../vectordb/supabase';
import OpenAI from 'openai';

const openai = new OpenAI();

export async function* streamRAGResponse(query, options = {}) {
  const { topK = 5, threshold = 0.7, model = 'gpt-4o' } = options;

  // Retrieve context (yield progress updates)
  yield { type: 'status', message: 'Searching knowledge base...' };

  const queryEmbedding = await generateEmbedding(query);
  const context = await searchDocuments(queryEmbedding, { threshold, limit: topK });

  yield { type: 'context', sources: context.length };

  if (context.length === 0) {
    yield { type: 'content', text: "I couldn't find relevant information." };
    yield { type: 'done' };
    return;
  }

  // Build prompt
  const contextText = context
    .map((doc, i) => `[${i + 1}] ${doc.content}`)
    .join('\n\n');

  yield { type: 'status', message: 'Generating response...' };

  // Stream generation
  const stream = await openai.chat.completions.create({
    model,
    messages: [
      {
        role: 'system',
        content: `Answer based on this context:\n\n${contextText}`,
      },
      { role: 'user', content: query },
    ],
    stream: true,
  });

  for await (const chunk of stream) {
    const text = chunk.choices[0]?.delta?.content;
    if (text) {
      yield { type: 'content', text };
    }
  }

  yield { type: 'sources', data: context };
  yield { type: 'done' };
}

// API route using streaming
// app/api/rag/route.js
import { streamRAGResponse } from '@/lib/rag/streaming';

export async function POST(request) {
  const { query } = await request.json();

  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      try {
        for await (const event of streamRAGResponse(query)) {
          const data = JSON.stringify(event) + '\n';
          controller.enqueue(encoder.encode(data));
        }
      } catch (error) {
        controller.enqueue(
          encoder.encode(JSON.stringify({ type: 'error', message: error.message }))
        );
      } finally {
        controller.close();
      }
    },
  });

  return new Response(stream, {
    headers: { 'Content-Type': 'text/event-stream' },
  });
}
```

---

## 5. Advanced RAG Techniques

### Hybrid Search (Vector + Keyword)

```javascript
// lib/rag/hybrid-search.js
import { generateEmbedding } from '../embeddings';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_SERVICE_KEY
);

// Combine vector similarity with full-text search
export async function hybridSearch(query, options = {}) {
  const {
    vectorWeight = 0.7,
    keywordWeight = 0.3,
    limit = 10,
  } = options;

  // Generate embedding for vector search
  const embedding = await generateEmbedding(query);

  // Perform both searches in parallel
  const [vectorResults, keywordResults] = await Promise.all([
    // Vector similarity search
    supabase.rpc('match_documents', {
      query_embedding: embedding,
      match_threshold: 0.5,
      match_count: limit * 2,
    }),
    // Full-text search
    supabase
      .from('documents')
      .select('id, content, metadata')
      .textSearch('content', query, { type: 'websearch' })
      .limit(limit * 2),
  ]);

  // Combine and score results
  const scoreMap = new Map();

  // Add vector scores
  for (const doc of vectorResults.data || []) {
    scoreMap.set(doc.id, {
      ...doc,
      vectorScore: doc.similarity,
      keywordScore: 0,
    });
  }

  // Add keyword scores (normalize position to 0-1)
  const keywordDocs = keywordResults.data || [];
  for (let i = 0; i < keywordDocs.length; i++) {
    const doc = keywordDocs[i];
    const keywordScore = 1 - (i / keywordDocs.length);

    if (scoreMap.has(doc.id)) {
      scoreMap.get(doc.id).keywordScore = keywordScore;
    } else {
      scoreMap.set(doc.id, {
        ...doc,
        vectorScore: 0,
        keywordScore,
      });
    }
  }

  // Calculate combined scores
  const results = Array.from(scoreMap.values())
    .map(doc => ({
      ...doc,
      combinedScore:
        doc.vectorScore * vectorWeight +
        doc.keywordScore * keywordWeight,
    }))
    .sort((a, b) => b.combinedScore - a.combinedScore)
    .slice(0, limit);

  return results;
}
```

### Query Expansion

```javascript
// lib/rag/query-expansion.js
import OpenAI from 'openai';

const openai = new OpenAI();

// Generate multiple search queries from user question
export async function expandQuery(query) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [
      {
        role: 'system',
        content: `Generate 3 alternative search queries for the given question.
Return only the queries, one per line.
Make queries diverse - use synonyms, different phrasings, related concepts.`,
      },
      {
        role: 'user',
        content: query,
      },
    ],
    temperature: 0.7,
  });

  const expandedQueries = response.choices[0].message.content
    .split('\n')
    .map(q => q.trim())
    .filter(q => q.length > 0);

  // Include original query
  return [query, ...expandedQueries];
}

// Search with expanded queries
export async function searchWithExpansion(query, searchFn, options = {}) {
  const { limit = 5 } = options;

  // Expand the query
  const queries = await expandQuery(query);
  console.log('Expanded queries:', queries);

  // Search with all queries
  const allResults = await Promise.all(
    queries.map(q => searchFn(q, { limit: limit * 2 }))
  );

  // Deduplicate and combine scores
  const resultMap = new Map();

  for (const results of allResults) {
    for (const doc of results) {
      if (resultMap.has(doc.id)) {
        // Keep the higher similarity score
        const existing = resultMap.get(doc.id);
        if (doc.similarity > existing.similarity) {
          resultMap.set(doc.id, doc);
        }
      } else {
        resultMap.set(doc.id, doc);
      }
    }
  }

  // Sort by similarity and limit
  return Array.from(resultMap.values())
    .sort((a, b) => b.similarity - a.similarity)
    .slice(0, limit);
}
```

### Re-ranking Retrieved Results

```javascript
// lib/rag/reranking.js
import OpenAI from 'openai';

const openai = new OpenAI();

// Use LLM to re-rank retrieved documents
export async function rerankDocuments(query, documents, topK = 5) {
  if (documents.length === 0) return [];
  if (documents.length <= topK) return documents;

  const prompt = `Given a query and a list of documents, rank them by relevance.
Return the indices of the top ${topK} most relevant documents in order.

Query: ${query}

Documents:
${documents.map((doc, i) => `[${i}] ${doc.content.substring(0, 500)}`).join('\n\n')}

Return only the indices as comma-separated numbers (e.g., "2,0,4,1,3").`;

  const response = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [
      { role: 'user', content: prompt },
    ],
    temperature: 0,
  });

  const indices = response.choices[0].message.content
    .split(',')
    .map(s => parseInt(s.trim()))
    .filter(i => !isNaN(i) && i >= 0 && i < documents.length);

  return indices.slice(0, topK).map(i => documents[i]);
}

// Context compression - extract only relevant parts
export async function compressContext(query, documents) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [
      {
        role: 'system',
        content: `Extract only the sentences from the documents that are relevant to answering the query. Return the relevant excerpts, preserving important context.`,
      },
      {
        role: 'user',
        content: `Query: ${query}

Documents:
${documents.map(d => d.content).join('\n\n---\n\n')}`,
      },
    ],
  });

  return response.choices[0].message.content;
}
```

### RAG Architecture Patterns

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Advanced RAG Patterns                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. NAIVE RAG (Simple)                                                  │
│     Query → Embed → Search → Generate                                   │
│     ✓ Simple  ✗ Limited accuracy                                        │
│                                                                         │
│  2. ADVANCED RAG (Production)                                           │
│     Query → Expand → Hybrid Search → Re-rank → Compress → Generate     │
│     ✓ Better accuracy  ✗ More latency                                   │
│                                                                         │
│  3. MODULAR RAG (Flexible)                                              │
│     ┌─────────────────────────────────────────────────────┐             │
│     │  Query Analysis → Route to appropriate pipeline     │             │
│     │                                                     │             │
│     │  ┌─────────┐  ┌─────────┐  ┌─────────┐            │             │
│     │  │ Simple  │  │ Multi-  │  │ Agent   │            │             │
│     │  │ Lookup  │  │ Hop     │  │ Based   │            │             │
│     │  └─────────┘  └─────────┘  └─────────┘            │             │
│     └─────────────────────────────────────────────────────┘             │
│                                                                         │
│  4. AGENTIC RAG (Dynamic)                                               │
│     Agent decides when/what to retrieve during generation               │
│     ✓ Most flexible  ✗ Complex, expensive                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Building a RAG API

### Complete Next.js Implementation

```javascript
// app/api/knowledge/ingest/route.js
import { ingestDocument } from '@/lib/rag/pipeline';
import { NextResponse } from 'next/server';

export async function POST(request) {
  try {
    const formData = await request.formData();
    const file = formData.get('file');
    const metadata = JSON.parse(formData.get('metadata') || '{}');

    if (!file) {
      return NextResponse.json(
        { error: 'No file provided' },
        { status: 400 }
      );
    }

    const buffer = Buffer.from(await file.arrayBuffer());
    const result = await ingestDocument(buffer, file.type, {
      ...metadata,
      filename: file.name,
      uploadedAt: new Date().toISOString(),
    });

    return NextResponse.json({
      success: true,
      chunksCreated: result.length,
    });
  } catch (error) {
    console.error('Ingestion error:', error);
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}

// app/api/knowledge/query/route.js
import { generateRAGResponse } from '@/lib/rag/pipeline';
import { NextResponse } from 'next/server';

export async function POST(request) {
  try {
    const { query, options = {} } = await request.json();

    if (!query) {
      return NextResponse.json(
        { error: 'Query is required' },
        { status: 400 }
      );
    }

    const result = await generateRAGResponse(query, options);

    return NextResponse.json(result);
  } catch (error) {
    console.error('Query error:', error);
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### React Chat Interface with RAG

```jsx
// components/RAGChat.jsx
'use client';

import { useState, useRef, useEffect } from 'react';

export function RAGChat() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [sources, setSources] = useState([]);
  const messagesEndRef = useRef(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  async function handleSubmit(e) {
    e.preventDefault();
    if (!input.trim() || isLoading) return;

    const userMessage = input.trim();
    setInput('');
    setMessages(prev => [...prev, { role: 'user', content: userMessage }]);
    setIsLoading(true);
    setSources([]);

    try {
      const response = await fetch('/api/knowledge/query', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ query: userMessage }),
      });

      const data = await response.json();

      if (data.error) {
        throw new Error(data.error);
      }

      setMessages(prev => [
        ...prev,
        { role: 'assistant', content: data.answer },
      ]);
      setSources(data.sources || []);
    } catch (error) {
      setMessages(prev => [
        ...prev,
        { role: 'assistant', content: `Error: ${error.message}` },
      ]);
    } finally {
      setIsLoading(false);
    }
  }

  return (
    <div className="flex flex-col h-[600px] border rounded-lg">
      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((msg, i) => (
          <div
            key={i}
            className={`flex ${
              msg.role === 'user' ? 'justify-end' : 'justify-start'
            }`}
          >
            <div
              className={`max-w-[80%] p-3 rounded-lg ${
                msg.role === 'user'
                  ? 'bg-blue-500 text-white'
                  : 'bg-gray-100 text-gray-900'
              }`}
            >
              {msg.content}
            </div>
          </div>
        ))}

        {isLoading && (
          <div className="flex justify-start">
            <div className="bg-gray-100 p-3 rounded-lg">
              <div className="flex space-x-2">
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" />
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce delay-100" />
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce delay-200" />
              </div>
            </div>
          </div>
        )}

        <div ref={messagesEndRef} />
      </div>

      {/* Sources */}
      {sources.length > 0 && (
        <div className="border-t p-3 bg-gray-50">
          <p className="text-sm font-medium text-gray-700 mb-2">Sources:</p>
          <div className="space-y-2">
            {sources.map((source, i) => (
              <div
                key={i}
                className="text-xs p-2 bg-white rounded border"
              >
                <p className="text-gray-600">{source.content}</p>
                <p className="text-gray-400 mt-1">
                  Similarity: {(source.similarity * 100).toFixed(1)}%
                </p>
              </div>
            ))}
          </div>
        </div>
      )}

      {/* Input */}
      <form onSubmit={handleSubmit} className="border-t p-4">
        <div className="flex space-x-2">
          <input
            type="text"
            value={input}
            onChange={(e) => setInput(e.target.value)}
            placeholder="Ask a question about your documents..."
            className="flex-1 px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
            disabled={isLoading}
          />
          <button
            type="submit"
            disabled={isLoading || !input.trim()}
            className="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 disabled:opacity-50"
          >
            Send
          </button>
        </div>
      </form>
    </div>
  );
}
```

### Document Upload Component

```jsx
// components/DocumentUpload.jsx
'use client';

import { useState, useCallback } from 'react';
import { useDropzone } from 'react-dropzone';

export function DocumentUpload({ onUploadComplete }) {
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(null);

  const onDrop = useCallback(async (acceptedFiles) => {
    if (acceptedFiles.length === 0) return;

    setUploading(true);
    setProgress({ current: 0, total: acceptedFiles.length });

    for (let i = 0; i < acceptedFiles.length; i++) {
      const file = acceptedFiles[i];
      setProgress({ current: i + 1, total: acceptedFiles.length, filename: file.name });

      const formData = new FormData();
      formData.append('file', file);
      formData.append('metadata', JSON.stringify({
        filename: file.name,
        size: file.size,
      }));

      try {
        const response = await fetch('/api/knowledge/ingest', {
          method: 'POST',
          body: formData,
        });

        const data = await response.json();

        if (data.error) {
          console.error(`Error uploading ${file.name}:`, data.error);
        }
      } catch (error) {
        console.error(`Error uploading ${file.name}:`, error);
      }
    }

    setUploading(false);
    setProgress(null);
    onUploadComplete?.();
  }, [onUploadComplete]);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: {
      'application/pdf': ['.pdf'],
      'text/plain': ['.txt'],
      'text/markdown': ['.md'],
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
    },
    disabled: uploading,
  });

  return (
    <div
      {...getRootProps()}
      className={`
        border-2 border-dashed rounded-lg p-8 text-center cursor-pointer
        transition-colors
        ${isDragActive ? 'border-blue-500 bg-blue-50' : 'border-gray-300 hover:border-gray-400'}
        ${uploading ? 'opacity-50 cursor-not-allowed' : ''}
      `}
    >
      <input {...getInputProps()} />

      {uploading ? (
        <div>
          <div className="animate-spin w-8 h-8 border-2 border-blue-500 border-t-transparent rounded-full mx-auto mb-4" />
          <p className="text-gray-600">
            Uploading {progress?.filename} ({progress?.current}/{progress?.total})
          </p>
        </div>
      ) : isDragActive ? (
        <p className="text-blue-500">Drop files here...</p>
      ) : (
        <div>
          <p className="text-gray-600 mb-2">
            Drag & drop documents here, or click to select
          </p>
          <p className="text-sm text-gray-400">
            Supports PDF, TXT, MD, DOCX
          </p>
        </div>
      )}
    </div>
  );
}
```

---

## 7. RAG Best Practices

### Evaluation Metrics

```javascript
// lib/rag/evaluation.js

// Evaluate retrieval quality
export function evaluateRetrieval(retrieved, relevant) {
  const retrievedIds = new Set(retrieved.map(d => d.id));
  const relevantIds = new Set(relevant.map(d => d.id));

  // Precision: What fraction of retrieved docs are relevant?
  const truePositives = [...retrievedIds].filter(id => relevantIds.has(id)).length;
  const precision = truePositives / retrievedIds.size;

  // Recall: What fraction of relevant docs were retrieved?
  const recall = truePositives / relevantIds.size;

  // F1 Score
  const f1 = precision + recall > 0
    ? (2 * precision * recall) / (precision + recall)
    : 0;

  // Mean Reciprocal Rank (MRR)
  let mrr = 0;
  for (let i = 0; i < retrieved.length; i++) {
    if (relevantIds.has(retrieved[i].id)) {
      mrr = 1 / (i + 1);
      break;
    }
  }

  return { precision, recall, f1, mrr };
}

// Evaluate answer quality using LLM
export async function evaluateAnswer(query, answer, context, openai) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [
      {
        role: 'system',
        content: `Evaluate the answer quality on these criteria (1-5 scale):
1. Relevance: Does it answer the question?
2. Faithfulness: Is it supported by the context?
3. Completeness: Does it cover all aspects?
4. Coherence: Is it well-structured?

Return JSON: { relevance: N, faithfulness: N, completeness: N, coherence: N }`,
      },
      {
        role: 'user',
        content: `Query: ${query}

Context: ${context}

Answer: ${answer}`,
      },
    ],
    response_format: { type: 'json_object' },
  });

  return JSON.parse(response.choices[0].message.content);
}
```

### Performance Optimization

```javascript
// lib/rag/optimization.js

// Cache embeddings for common queries
import { LRUCache } from 'lru-cache';

const embeddingCache = new LRUCache({
  max: 1000,
  ttl: 1000 * 60 * 60, // 1 hour
});

export async function getCachedEmbedding(text, generateFn) {
  const cached = embeddingCache.get(text);
  if (cached) return cached;

  const embedding = await generateFn(text);
  embeddingCache.set(text, embedding);
  return embedding;
}

// Batch embedding generation
export async function batchGenerateEmbeddings(texts, openai, batchSize = 100) {
  const results = [];

  for (let i = 0; i < texts.length; i += batchSize) {
    const batch = texts.slice(i, i + batchSize);

    const response = await openai.embeddings.create({
      model: 'text-embedding-3-small',
      input: batch,
    });

    results.push(...response.data.map(d => d.embedding));

    // Rate limiting
    if (i + batchSize < texts.length) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }

  return results;
}

// Parallel retrieval from multiple sources
export async function multiSourceRetrieval(query, sources) {
  const results = await Promise.all(
    sources.map(source => source.search(query).catch(() => []))
  );

  // Merge and deduplicate
  const seen = new Set();
  const merged = [];

  for (const sourceResults of results) {
    for (const doc of sourceResults) {
      if (!seen.has(doc.id)) {
        seen.add(doc.id);
        merged.push(doc);
      }
    }
  }

  return merged.sort((a, b) => b.similarity - a.similarity);
}
```

---

## Practice Exercises

### Exercise 1: Build a Documentation Search
Create a RAG system that indexes and searches your project's documentation.

Requirements:
- Ingest markdown files from a docs folder
- Implement search with source highlighting
- Add filters by section/category

### Exercise 2: Multi-Modal RAG
Extend RAG to handle images and tables.

Features:
- Extract text from images using OCR
- Parse tables from PDFs
- Generate descriptions for images

### Exercise 3: Conversational RAG
Build a chatbot that maintains context across turns.

Features:
- Track conversation history
- Reference previous answers
- Handle follow-up questions

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        RAG Key Concepts                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. EMBEDDINGS                                                          │
│     • Vector representations of text                                    │
│     • Enable semantic similarity search                                 │
│     • Choose model based on accuracy/cost needs                         │
│                                                                         │
│  2. CHUNKING                                                            │
│     • Break documents into searchable pieces                            │
│     • Respect semantic boundaries                                       │
│     • Balance size vs. context                                          │
│                                                                         │
│  3. RETRIEVAL                                                           │
│     • Vector search for semantic similarity                             │
│     • Hybrid search combines keywords + vectors                         │
│     • Re-ranking improves relevance                                     │
│                                                                         │
│  4. GENERATION                                                          │
│     • Augment LLM with retrieved context                                │
│     • Cite sources for transparency                                     │
│     • Handle missing information gracefully                             │
│                                                                         │
│  5. OPTIMIZATION                                                        │
│     • Cache frequently used embeddings                                  │
│     • Batch operations where possible                                   │
│     • Evaluate and iterate on quality                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Additional Resources

- [Pinecone Documentation](https://docs.pinecone.io/)
- [Supabase Vector Guide](https://supabase.com/docs/guides/ai)
- [LangChain RAG Guide](https://python.langchain.com/docs/tutorials/rag/)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [RAG Survey Paper](https://arxiv.org/abs/2312.10997)

---

Tomorrow, we'll build a complete **AI-Powered Project** - combining everything we've learned to create a production-ready AI application.
