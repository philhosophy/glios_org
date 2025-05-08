---
layout: post
title: "Introducing Bo's Three-Lane Architecture for Real-Time AI Companions"
description: "How we're solving the latency challenge in AI companions to enable truly natural conversations"
date: 2025-05-10 15:01:35 +0300
image: '/images/bo.png'
tags: [technology, ai, architecture]
---

In building Bo, our compassionate AI companion, we quickly learned that creating a genuinely engaging conversational experience requires more than just intelligent responses - it demands *responsiveness*. Users perceive even small delays as unnatural, breaking the illusion of a fluid conversation and eroding trust.

Our solution? A novel **three-lane architecture** designed to balance the need for near-instantaneous responses with the computational demands of deep reasoning and knowledge assimilation. This approach allows Bo to maintain natural conversation flow while continuously learning and evolving in the background.

## The Latency Challenge

Our early experiments with standard language model architectures revealed a fundamental problem: median voice-to-voice response times often exceeded 3 seconds, creating awkward pauses that felt distinctly mechanical. Human conversations, by contrast, feature average response latencies around 200-500ms. This gap needed to be bridged.

## The Three Lanes of Bo's Mind

Our architecture divides Bo's cognitive processes into three parallel pathways:

### 1. The Fast Lane (Cache Hits)

The Fast Lane is optimized for sub-500ms voice-to-voice responses. When a user speaks, Bo first checks if the utterance (or something semantically similar) has been encountered before. If so, Bo can respond almost immediately using cached responses.

By leveraging Redis with RedisVL for vector similarity search, we can match incoming utterances against previously seen patterns and respond in microseconds. This handles a surprising number of interactions - our testing shows approximately 35-40% of casual conversation follows predictable patterns that can be effectively cached.

### 2. The Read-Through Lane (Cache Miss, KG Hit)

When Bo encounters a novel utterance, but possesses relevant context in its short-term knowledge graph, the Read-Through Lane activates. This leverages a Neo4j-based knowledge graph with Graphiti for quick context retrieval.

This lane handles novel questions about previously discussed topics, allowing Bo to maintain coherent conversations about users' interests, past discussions, and shared information without relying on the slower LLM-based reasoning. The target latency here is sub-1-second, still maintaining conversational flow.

### 3. The Background Reasoning & Learning Lane

The third lane operates asynchronously, processing conversations to enrich Bo's knowledge graph, update caches, and enable continuous learning. When deeper reasoning is required, this lane engages larger language models like o4-mini to analyze the full conversation context and update Bo's understanding.

Critically, this lane doesn't block the conversation. If Bo needs time to "think," it can use natural-sounding fillers ("Hmm, let me think about that...") while the reasoning process completes in the background.

## Bringing It All Together

The magic happens in the Decision Orchestrator, which routes conversations through the appropriate lanes based on complexity and context. It can also interrupt background reasoning if the user speaks again, ensuring Bo remains responsive rather than finishing an obsolete thought.

```mermaid
graph TD
    A[User Voice Input] --> B{LiveKit SFU};
    B --> C[STT Engine (Deepgram)];
    C -- Transcript Stream --> D{Decision Orchestrator / Planner};

    subgraph RealtimePath [Real-Time Interaction Path]
        D -- Cache Lookup Key --> E[Redis SemanticCache];
        E -- Cache Hit (Cached 'speak' Text) --> F[TTS Engine (GPT-4o Realtime)];
        E -- Cache Miss --> G[Knowledge Graph Read];
        G -- KG Context --> D;
        D -- "Speak" Text --> F;
        F -- Synthesized Speech --> B;
        B --> H[User Hears Bo];
    end

    subgraph BackgroundProcessingPath [Background Reasoning & Learning Path]
        D -- Transcript + Context --> I[Reasoning LLM (o4-mini)];
        I -- Structured JSON --> D;
        I -- Data for KG --> J[Knowledge Graph Write/Update];
        J -- Updated KG State --> K[Cache Update];
        I -- Data for Long-Term Storage --> L[Deep RAG (LightRAG)];
    end

    D -- "Thinking" Filler Request --> F;
```

## Results: A More Human Conversation

With this architecture, we've achieved remarkable results:
- **Cache Hit Scenarios**: ~450ms end-to-end latency
- **Read-Through Scenarios**: ~750-900ms latency
- **Background Processing**: Asynchronous, with natural-sounding fillers

The result is a conversation that *feels* natural, even when Bo is processing complex queries. Users don't perceive the technical complexity - they simply experience a responsive companion who listens and responds with apparent understanding and minimal delay.

## Beyond Voice: The Full Experience

Combined with our Unity-based frontend featuring Mixamo animations and OVRLipSync for realistic mouth movements, the three-lane architecture creates an illusion of presence that early testers have described as "uncannily natural" and "genuinely engaging."

This is just the beginning of our journey to create Bo, a compassionate AI companion designed for authentic human connection. We're excited to share more technical insights as we progress.

*For a deeper dive into Bo's architecture, check out our [whitepaper](/whitepaper).* 