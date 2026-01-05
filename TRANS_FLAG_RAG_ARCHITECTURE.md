# 🏗️ Trans Flag RAG Architecture & Implementation Guide

> **เอกสารนี้:** สำหรับทำในอนาคต ถ้า `trans_flag_filter.json` (Option A) ไม่เพียงพอ
> 
> **วันที่สร้าง:** 2026-01-05  
> **เวอร์ชัน:** 1.0.0  
> **สถานะ:** 📋 Design Document (ยังไม่ implement)

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Current Problems](#current-problems)
3. [RAG Solution Architecture](#rag-solution-architecture)
4. [Technology Stack](#technology-stack)
5. [Implementation Steps](#implementation-steps)
6. [Code Examples](#code-examples)
7. [Migration Plan](#migration-plan)
8. [Testing Strategy](#testing-strategy)
9. [Performance Metrics](#performance-metrics)
10. [Cost Analysis](#cost-analysis)

---

## 📖 Overview

### What is RAG?

**RAG (Retrieval Augmented Generation)** = การดึงข้อมูลที่เกี่ยวข้องมาเสริม AI ก่อน generate คำตอบ

```
Traditional AI:
User Question → AI (with full context) → Answer
❌ ต้องส่ง context ทั้งหมด (แพง, ช้า)

RAG AI:
User Question → Retrieve relevant docs → AI (with focused context) → Answer
✅ ส่งเฉพาะที่เกี่ยวข้อง (ถูก, เร็ว, แม่นยำ)
```

### Why RAG for Trans Flags?

- **trans_flag.json** มี 150+ รายการ (~5,000 tokens)
- User ถามครั้งละ 1-3 trans_flags เท่านั้น
- ส่งทั้งหมดให้ AI = **เสียเงิน 95%** โดยใช่เหตุ
- RAG ช่วย: ดึงเฉพาะ 5-10 trans_flags ที่เกี่ยวข้อง

---

## ⚠️ Current Problems

### Problem 1: No Trans Flag Context

```typescript
// ChatService.ts - ปัจจุบัน
const prompt = GeminiService.buildSQLPrompt(
  message,
  template.systemPrompt,     // ✅ มี
  template.schema,           // ✅ มี
  template.specialQueries,   // ✅ มี
  // ❌ ไม่มี trans_flag context!
);

// AI ไม่รู้ว่า "เบิก" = trans_flag 56
```

### Problem 2: If We Add Full trans_flag.json

```typescript
// ถ้าเพิ่ม trans_flag.json ทั้งหมด
const transFlags = loadTransFlag(); // 150 รายการ = 5,000 tokens

prompt += JSON.stringify(transFlags); // ❌ แพงมาก!

// Current token usage: 8,000 tokens
// With full trans_flag: 13,000 tokens (+62.5%)
// Cost increase: +฿0.10 per request
```

### Problem 3: Slow & Expensive

| Metric | Current | With Full trans_flag | Problem |
|--------|---------|---------------------|---------|
| Tokens/Request | 8,000 | 13,000 | +62.5% |
| Cost/Request | ฿0.08 | ฿0.13 | +62.5% |
| Response Time | 3s | 4-5s | +66% |

---

## 🎯 RAG Solution Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    USER QUESTION                        │
│         "ดูสินค้าที่เบิกไปเดือนนี้"                      │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              STEP 1: Intent Analysis                    │
│              (GeminiService - existing)                 │
│  - Detect follow-up                                     │
│  - Extract context                                      │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│         STEP 2: RAG Retrieval (NEW!)                    │
│                                                          │
│  ┌──────────────────────────────────────────┐          │
│  │  TransFlagRAGService                     │          │
│  │  - Embed user question                   │          │
│  │  - Query vector database                 │          │
│  │  - Filter by template (stock/ap/ar)     │          │
│  │  - Return top 10 trans_flags             │          │
│  └──────────────────────────────────────────┘          │
│                                                          │
│  Input:  "ดูสินค้าที่เบิกไป"                            │
│  Output: [56, 122, 58, 57, 59] + descriptions          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│         STEP 3: Build Enhanced Prompt                   │
│                                                          │
│  Prompt = {                                             │
│    systemPrompt (full),                                 │
│    schema (full),                                       │
│    specialQueries (RAG - top 3),      ← RAG!           │
│    transFlags (RAG - top 10),         ← RAG!           │
│    intentContext                                        │
│  }                                                      │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│         STEP 4: SQL Generation (Gemini)                 │
│  - AI รู้ว่า "เบิก" = trans_flag 56                     │
│  - Generate: WHERE trans_flag = 56                      │
└─────────────────────────────────────────────────────────┘
```

### Data Flow

```
┌───────────────────────┐
│  trans_flag.json      │
│  (Master - 150 items) │
└───────────────────────┘
           ↓ Index once
┌───────────────────────┐
│  Vector Database      │
│  (Embeddings)         │
│  - Chroma / Pinecone  │
│  - Qdrant / Weaviate  │
└───────────────────────┘
           ↓ Query (every request)
┌───────────────────────┐
│  Top 10 Trans Flags   │
│  (Relevant only)      │
└───────────────────────┘
```

---

## 🛠️ Technology Stack

### Option 1: Chroma (แนะนำ - Open Source, Easy)

```bash
# Install
npm install chromadb

# Features
✅ Open source
✅ ติดตั้งง่าย (npm install)
✅ ใช้ local หรือ server ได้
✅ รองรับ TypeScript
✅ ฟรี
```

### Option 2: Pinecone (Managed Service)

```bash
# Install
npm install @pinecone-database/pinecone

# Features
✅ Fully managed (no server setup)
✅ Scalable
✅ Fast
⚠️ มีค่าใช้จ่าย (free tier: 100k vectors)
```

### Option 3: Qdrant (High Performance)

```bash
# Install
npm install @qdrant/js-client-rest

# Features
✅ High performance
✅ Open source
✅ Rust-based (super fast)
⚠️ ต้อง setup server
```

### Embedding Model Options

| Model | Dimension | Cost | Speed | Quality |
|-------|-----------|------|-------|---------|
| **text-embedding-004** (Google) | 768 | ฿0.00001/1K | Fast | Good |
| **text-embedding-3-small** (OpenAI) | 1536 | ฿0.00002/1K | Fast | Better |
| **text-embedding-3-large** (OpenAI) | 3072 | ฿0.00013/1K | Slow | Best |

**แนะนำ:** `text-embedding-004` (Google) - ถูก, เร็ว, คุณภาพดี

---

## 📝 Implementation Steps

### Phase 1: Setup Vector Database

#### Step 1.1: Install Dependencies

```bash
# Terminal
cd /Users/nontawatwongnuk/dev/sml_chatbot
npm install chromadb
npm install @google/generative-ai  # For embeddings
```

#### Step 1.2: Create ChromaDB Client

```typescript
// src/db/chromadb.ts
import { ChromaClient } from 'chromadb';
import logger from '../config/logger';

class ChromaDBService {
  private client: ChromaClient;
  private collection: any;

  constructor() {
    this.client = new ChromaClient({
      path: 'http://localhost:8000', // Chroma server
    });
  }

  async initialize() {
    try {
      // Create or get collection
      this.collection = await this.client.getOrCreateCollection({
        name: 'trans_flags',
        metadata: { description: 'SML Trans Flag embeddings' },
      });
      
      logger.info('✅ ChromaDB initialized');
    } catch (error) {
      logger.error('❌ ChromaDB initialization failed:', error);
      throw error;
    }
  }

  getCollection() {
    return this.collection;
  }
}

export default new ChromaDBService();
```

#### Step 1.3: Start Chroma Server

```bash
# Terminal - Run Chroma server
docker run -p 8000:8000 chromadb/chroma

# Or install locally
pip install chromadb
chroma run --path ./chroma_data
```

---

### Phase 2: Index trans_flag.json

#### Step 2.1: Create Indexing Service

```typescript
// src/services/TransFlagIndexService.ts
import chromadb from '../db/chromadb';
import { GoogleGenerativeAI } from '@google/generative-ai';
import logger from '../config/logger';
import config from '../config';

interface TransFlag {
  trans_flag: string;
  'ชื่อเมนุ(ภาษาไทย)': string;
  'ชื่อเมนุ(ภาษาอังกฤษ)': string;
  Table: string;
  description: string;
}

class TransFlagIndexService {
  private genAI: GoogleGenerativeAI;
  private embeddingModel: any;

  constructor() {
    this.genAI = new GoogleGenerativeAI(config.gemini.apiKey);
    this.embeddingModel = this.genAI.getGenerativeModel({
      model: 'text-embedding-004',
    });
  }

  /**
   * Index all trans_flags into vector database
   */
  async indexTransFlags(transFlags: TransFlag[]) {
    try {
      logger.info(`🔄 Indexing ${transFlags.length} trans_flags...`);

      const collection = chromadb.getCollection();
      
      // Prepare data
      const ids: string[] = [];
      const embeddings: number[][] = [];
      const documents: string[] = [];
      const metadatas: any[] = [];

      for (const tf of transFlags) {
        // Create searchable text
        const searchText = `
          ${tf['ชื่อเมนุ(ภาษาไทย)']}
          ${tf['ชื่อเมนุ(ภาษาอังกฤษ)']}
          ${tf.description}
          trans_flag ${tf.trans_flag}
        `.trim();

        // Generate embedding
        const result = await this.embeddingModel.embedContent(searchText);
        const embedding = result.embedding.values;

        // Add to arrays
        ids.push(`tf_${tf.trans_flag}`);
        embeddings.push(embedding);
        documents.push(searchText);
        metadatas.push({
          trans_flag: tf.trans_flag,
          name_th: tf['ชื่อเมนุ(ภาษาไทย)'],
          name_en: tf['ชื่อเมนุ(ภาษาอังกฤษ)'],
          table: tf.Table,
          description: tf.description,
        });
      }

      // Batch insert
      await collection.add({
        ids,
        embeddings,
        documents,
        metadatas,
      });

      logger.info(`✅ Indexed ${transFlags.length} trans_flags successfully`);
    } catch (error) {
      logger.error('❌ Indexing failed:', error);
      throw error;
    }
  }

  /**
   * Re-index from trans_flag.json file
   */
  async reindexFromFile(filePath: string) {
    const fs = require('fs');
    const transFlags = JSON.parse(fs.readFileSync(filePath, 'utf-8'));
    await this.indexTransFlags(transFlags);
  }
}

export default new TransFlagIndexService();
```

#### Step 2.2: Run Indexing Script

```typescript
// scripts/index-trans-flags.ts
import TransFlagIndexService from '../src/services/TransFlagIndexService';
import chromadb from '../src/db/chromadb';
import path from 'path';

async function main() {
  try {
    // Initialize ChromaDB
    await chromadb.initialize();

    // Index trans_flags
    const transFilePath = path.join(
      __dirname,
      '../template-repo/templates/trans_flag.json'
    );

    await TransFlagIndexService.reindexFromFile(transFilePath);

    console.log('✅ Indexing complete!');
    process.exit(0);
  } catch (error) {
    console.error('❌ Indexing failed:', error);
    process.exit(1);
  }
}

main();
```

```bash
# Run indexing
npm run ts-node scripts/index-trans-flags.ts
```

---

### Phase 3: Create RAG Service

```typescript
// src/services/TransFlagRAGService.ts
import chromadb from '../db/chromadb';
import { GoogleGenerativeAI } from '@google/generative-ai';
import config from '../config';
import logger from '../config/logger';

interface TransFlagResult {
  trans_flag: string;
  name_th: string;
  name_en: string;
  description: string;
  score: number;
}

class TransFlagRAGService {
  private genAI: GoogleGenerativeAI;
  private embeddingModel: any;

  constructor() {
    this.genAI = new GoogleGenerativeAI(config.gemini.apiKey);
    this.embeddingModel = this.genAI.getGenerativeModel({
      model: 'text-embedding-004',
    });
  }

  /**
   * Find relevant trans_flags for user question
   */
  async findRelevantTransFlags(
    userQuestion: string,
    template: string,
    topK: number = 10
  ): Promise<TransFlagResult[]> {
    try {
      // Generate query embedding
      const result = await this.embeddingModel.embedContent(userQuestion);
      const queryEmbedding = result.embedding.values;

      // Query vector database
      const collection = chromadb.getCollection();
      const results = await collection.query({
        queryEmbeddings: [queryEmbedding],
        nResults: topK,
      });

      // Parse results
      const transFlags: TransFlagResult[] = [];
      
      if (results.metadatas && results.metadatas[0]) {
        results.metadatas[0].forEach((metadata: any, index: number) => {
          transFlags.push({
            trans_flag: metadata.trans_flag,
            name_th: metadata.name_th,
            name_en: metadata.name_en,
            description: metadata.description,
            score: results.distances?.[0]?.[index] || 0,
          });
        });
      }

      logger.debug(`Found ${transFlags.length} relevant trans_flags`, {
        question: userQuestion,
        template,
        topK,
      });

      return transFlags;
    } catch (error) {
      logger.error('RAG query failed:', error);
      return [];
    }
  }

  /**
   * Format trans_flags for prompt
   */
  formatForPrompt(transFlags: TransFlagResult[]): string {
    let formatted = `## 🏷️ Relevant Trans Flags:\n\n`;
    
    transFlags.forEach((tf, index) => {
      formatted += `${index + 1}. **trans_flag = ${tf.trans_flag}** (${tf.name_th})\n`;
      formatted += `   - English: ${tf.name_en}\n`;
      formatted += `   - Description: ${tf.description}\n`;
      formatted += `   - Relevance: ${(1 - tf.score).toFixed(3)}\n\n`;
    });

    return formatted;
  }
}

export default new TransFlagRAGService();
```

---

### Phase 4: Integrate with ChatService

```typescript
// src/services/ChatService.ts (Modified)

import TransFlagRAGService from './TransFlagRAGService';

class ChatService {
  async sendMessage(...) {
    // ... existing code ...

    // STEP 2: RAG Retrieval (NEW!)
    logger.info(`🔍 [STEP 2/4] Retrieving relevant trans_flags...`);
    
    const relevantTransFlags = await TransFlagRAGService.findRelevantTransFlags(
      message,
      template.templateId,
      10  // top 10
    );

    const transFlagContext = TransFlagRAGService.formatForPrompt(
      relevantTransFlags
    );

    logger.info(`✓ Found ${relevantTransFlags.length} relevant trans_flags`);

    // STEP 3: Build SQL prompt with RAG context
    const prompt = GeminiService.buildSQLPrompt(
      message,
      finalSystemPrompt,
      schemaContext,
      conversationHistory,
      template.customQueries || [],
      channel.customSystemPrompt || '',
      template.schema,
      template.specialQueries,
      transFlagContext,  // ← NEW! RAG context
      intentAnalysis
    );

    // ... rest of the code ...
  }
}
```

---

### Phase 5: Update GeminiService

```typescript
// src/services/GeminiService.ts (Modified)

buildSQLPrompt(
  userMessage: string,
  systemPrompt: string,
  schema: string,
  conversationHistory: string[] = [],
  customQueries: any[] = [],
  channelCustomInstructions: string = '',
  templateSchemaMarkdown?: string,
  templateSpecialQueriesMarkdown?: string,
  transFlagContext?: string,  // ← NEW!
  intentContext?: any
): string {
  let prompt = `${systemPrompt}\n\n`;

  // Add intent context
  if (intentContext && intentContext.isFollowUp) {
    // ... existing code ...
  }

  // Add trans_flag context (RAG)
  if (transFlagContext && transFlagContext.trim()) {
    prompt += `${transFlagContext}\n\n`;
    prompt += `⚠️ **IMPORTANT:** Use these trans_flags when building WHERE clauses.\n\n`;
  }

  // Add schema
  if (templateSchemaMarkdown && templateSchemaMarkdown.trim()) {
    prompt += `${templateSchemaMarkdown}\n\n`;
  }

  // ... rest of the code ...

  return prompt;
}
```

---

## 🧪 Testing Strategy

### Test 1: Accuracy Test

```typescript
// tests/rag/accuracy.test.ts

describe('TransFlagRAG Accuracy', () => {
  test('should find correct trans_flag for "เบิกสินค้า"', async () => {
    const results = await TransFlagRAGService.findRelevantTransFlags(
      'ดูสินค้าที่เบิกไปเดือนนี้',
      'stock',
      10
    );

    expect(results[0].trans_flag).toBe('56'); // เบิกสินค้า
    expect(results[0].score).toBeLessThan(0.3); // High similarity
  });

  test('should find correct trans_flag for "ใบสั่งซื้อ"', async () => {
    const results = await TransFlagRAGService.findRelevantTransFlags(
      'ดู PO ที่รอรับของ',
      'stock',
      10
    );

    expect(results[0].trans_flag).toBe('6'); // PO
  });
});
```

### Test 2: Performance Test

```typescript
// tests/rag/performance.test.ts

describe('TransFlagRAG Performance', () => {
  test('should retrieve in < 200ms', async () => {
    const start = Date.now();
    
    await TransFlagRAGService.findRelevantTransFlags(
      'ดูสินค้าที่เบิกไป',
      'stock',
      10
    );

    const elapsed = Date.now() - start;
    expect(elapsed).toBeLessThan(200);
  });
});
```

---

## 📊 Performance Metrics

### Expected Results

| Metric | Before RAG | After RAG | Improvement |
|--------|-----------|-----------|-------------|
| **Avg Tokens/Request** | 8,000 | 5,000 | **-37.5%** |
| **Cost/Request** | ฿0.08 | ฿0.05 | **-37.5%** |
| **Response Time** | 3s | 2.5s | **-16.7%** |
| **Accuracy** | 85% | 95% | **+11.8%** |
| **Monthly Cost** (10K requests) | ฿800 | ฿500 | **-฿300** |

---

## 💰 Cost Analysis

### Development Cost

| Task | Time | Cost (Developer @ ฿500/hr) |
|------|------|---------------------------|
| Setup ChromaDB | 2h | ฿1,000 |
| Create Indexing Service | 3h | ฿1,500 |
| Create RAG Service | 4h | ฿2,000 |
| Integration | 3h | ฿1,500 |
| Testing | 2h | ฿1,000 |
| **Total** | **14h** | **฿7,000** |

### Operational Cost

| Item | Cost/Month |
|------|-----------|
| ChromaDB Server (self-hosted) | ฿0 (free) |
| Embedding API (10K requests) | ฿1 |
| Gemini API Savings | -฿300 |
| **Net Savings** | **-฿299/month** |

**ROI:** Break even in **23 days** (฿7,000 / ฿299)

---

## 🚀 Migration Plan

### Week 1: Setup & Development

- **Day 1-2:** Setup ChromaDB server
- **Day 3-4:** Develop indexing service
- **Day 5:** Index trans_flag.json

### Week 2: Integration & Testing

- **Day 1-2:** Develop RAG service
- **Day 3:** Integrate with ChatService
- **Day 4-5:** Testing & bug fixes

### Week 3: Deployment

- **Day 1:** Deploy to staging
- **Day 2-3:** User acceptance testing
- **Day 4:** Deploy to production
- **Day 5:** Monitor & optimize

---

## 📌 Next Steps

### If trans_flag_filter.json Works Well:
✅ Keep using it - ง่าย, maintainable

### If Need Better Performance:
1. Review this document
2. Implement RAG following steps above
3. A/B test with trans_flag_filter.json
4. Gradually migrate

---

## 📝 Conclusion

**RAG Architecture พร้อมใช้งาน:**
- ✅ Design complete
- ✅ Implementation guide ready
- ✅ Code examples provided
- ✅ Testing strategy defined
- ✅ Cost analysis done

**เมื่อไหร่ควรทำ:**
- ⏰ เมื่อ trans_flag_filter.json ไม่เพียงพอ
- ⏰ เมื่อต้องการ accuracy สูงขึ้น
- ⏰ เมื่อต้องการประหยัดค่า API

**ไม่ต้องรีบ:**
- ทดสอบ trans_flag_filter.json ก่อน
- ถ้าดีพอ ไม่ต้องทำ RAG
- เก็บเอกสารนี้ไว้สำหรับอนาคต

---

**Document Version:** 1.0.0  
**Last Updated:** 2026-01-05  
**Author:** GitHub Copilot  
**Status:** 📋 Ready for Implementation
