---
title: "Agentic Vision: Building Visual Intelligence with Amazon Bedrock and MCP Servers"
created: 2026-07-24
updated: 2026-09-18
type: entity
tags: [aws, bedrock, computer-vision, mcp, strands-agents, ai-agents, s3, opensearch, rekognition]
sources: [raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agentic Vision: Building Visual Intelligence with Amazon Bedrock and MCP Servers

## Overview

AWS blog post (2026-07-15) by Kiowa Jackson, Jundong Qiao, Justin Kuskowski, and Nick Biso demonstrating how to converge Computer Vision, Strands Agents, and the Model Context Protocol (MCP) into a unified pipeline. The architecture bridges perception, decision-making, and action through standardized interfaces — allowing AI systems to see, understand, and respond in a coordinated way. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md]

## Architecture

The solution uses a centralized IAM role as a security gateway, with Amazon S3 for object storage, Amazon OpenSearch for search capabilities, Amazon Bedrock for generative AI models, and Amazon Rekognition for image analysis. A Streamlit chat UI provides the user interface with media upload (images/videos up to 200 MB) and model selection (Claude 4 Sonnet, Claude 3.7 Sonnet). ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md]

## Two MCP Servers

### CV Server
Provides a unified interface for image and video analysis consolidating three Amazon AI services:
- **describe_image** — uses Claude model in Bedrock for image analysis with specific monitoring instructions; retrieves images from S3 and processes through Claude's multimodal capabilities
- **analyze_video** — uses Amazon Nova video analysis to process video content according to specific instructions
- **detect_labels** — integrates with Amazon Rekognition for label detection and image property analysis, providing bounding box information for spatial localization
- **crop_bounding_box** — uses Rekognition's object detection to identify key elements and provide precise bounding box coordinates for intelligent cropping
- **remove_background** — uses the rembg library for background removal without complex ML setup

### OpenSearch Server
Provides a unified interface for image ingestion and retrieval:
- **generate_image_description** — analyzes images using Bedrock Claude models and generates natural language descriptions
- **generate_multimodal_embedding** — uses Amazon Titan multimodal models to create high-dimensional vector embeddings capturing visual and textual information
- **ingest_image_to_opensearch** — end-to-end pipeline for processing and storing images in OpenSearch with metadata
- **query_images_by_text** — supports natural language search across image collections using multimodal embeddings
- **query_images_by_image** — image-based similarity search
- **bulk_ingest_images** — batch processing for large-scale ingestion

## Use Cases

1. **Infrastructure-less Computer Vision Pipeline** — perform bounding box generation, image descriptions, and video analysis without dedicated servers; pay-per-use model with rapid deployment
2. **Intelligent Image Cataloging with Embeddings** — embedding-based similarity algorithms for semantic understanding of visual content; transcends keyword-based search limitations
3. **Visual Memory Database for Contextual Reasoning** — combines CV pipelines with embedding-based similarity; processes scenes to extract objects and bounding boxes, generates embeddings, and stores with temporal/spatial metadata for multi-camera contextual reasoning ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md]

## 深度分析

### Tool-Mediated Vision Beats Prompt-Stuffed Pixels

The decisive choice is what the agent is *not* given: raw pixels never enter its context. Every visual operation is a tool call — `describe_image` fetches from S3 and drives a Bedrock Claude multimodal call inside the tool boundary, `analyze_video` delegates to Amazon Nova, `detect_labels` hands pixels to Rekognition. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:97-98,176,204] Uploads run to 200 MB, far beyond any context window, yet only *text* returns to the loop; tools pass original objects at native resolution, expose confidence and label-cap knobs so identical calls yield identical shapes, and leave every hop as a named tool over a named S3 key. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:30,204,85]

### Semantic Index vs. Specialist CV Service: A Real Division of Labour

The two servers occupy different regimes. Rekognition is a closed-set, stateless classifier: it answers "is this a cat, and where?" with boxes and scores, dominating the easy end of its taxonomy (Cat, Kitten, Grass at 99.99%) while hedging at the hard end — "Manx" at 77.85%, honest uncertainty that must not be laundered into fact. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:204,222-231] OpenSearch is the inverse: Titan multimodal embeddings put images and text in one shared vector space, making the collection searchable by meaning rather than keyword — but the result is a ranked guess, as the owl-to-owl match at 0.65 similarity shows. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:246-253,266] Use the specialist when output must be defensible; use the index when a triage shortlist is the answer.

### MCP as the Boundary That Makes the Orchestrator Disposable

MCP's contribution is less the tools than the fact that the boundary is *declared*, replacing a bespoke connection for every model-and-data-source pairing, so with stable tool contracts the caller becomes interchangeable — the Streamlit UI is one client, the Inline Agent orchestrator another, and the same servers serve both cataloging and visual-memory workloads. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:24,28,293-301] The reusable asset is the tool surface, which is the composition argument for [[entities/mcp-protocol.md|MCP]]; CV covers fresh perception, search covers accumulated perception, and an orchestrator such as [[entities/strands-agents.md|Strands Agents]] only chooses which to call. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:95,244]

### Agentic Loops Over Visual Data, Not Single-Shot Captioning

One captioning call is one-shot: pixels in, prose out. The blog demonstrates a loop instead, and the system prompt makes it legible — acknowledge, execute tools in sequence, render through UI tools, summarize — under explicit multi-step workflows and cross-call variable-passing rules. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:35-87] Because description, labels, and a crop can all be requested for one artifact, the agent reconciles independent views before answering, and the prompt delegates explicitly — the seam at which a lone CV agent becomes one specialist among several. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:184]

### Failure Modes and Cost Tradeoffs

**Embedding versus on-the-fly analysis:** `ingest_image_to_opensearch` bundles description, embedding, and indexing, with `bulk_ingest_images` for batches — upfront cost, cheap repeat queries; on-demand `describe_image` is the inverse, needing no setup but repeating work and leaving nothing searchable. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:248,270,97-98] **Borderline labels:** 77.85% and 99.99% are the same kind of output with very different warrant, so an explicit actionable-threshold policy is mandatory. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:204,222-231] **Image-borne text:** descriptions and OCR-derived strings re-enter the loop as instruction-shaped content marked as neither image-sourced nor user-said; the single-IAM-role design handles credential sprawl but is silent on this channel. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:20,97]

### Why the Pattern Generalizes Beyond Images

Strip the product names and a recipe for multimodal [[concepts/rag-retrieval-augmented-generation.md|RAG]] remains: an object store for artifacts, specialist analyzers converting them into text and vectors, a semantic index over those vectors, and an agent deciding which analyzer to invoke and when to cite. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:20,314-316] The visual-memory case adds the payoff: with temporal and spatial metadata attached, the index becomes an event log rather than a document set. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:301] The closing claim — standardization, accessibility, integration — is that the boundary is the product, joining [[entities/multimodal-agentic-frameworks-survey.md|multimodal agentic frameworks]] and [[entities/agentic-ai-data-mesh-aws-s3-vectors-mcp.md|S3-plus-vectors data meshes]]. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:315-316]

## 实践启示

1. **Keep pixels out of the prompt and put them behind tools** that own the fetch, the model call, and the return shape — then media size stops constraining context. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:97-98,30]
2. **Choose the instrument by the output's burden of proof**: actionable calls on a thresholded specialist CV service, open-vocabulary discovery on a multimodal embedding index, with confidence returned as a number rather than prose. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:204,253,266,85]
3. **Let reuse decide embed-once versus analyze-on-demand.** Ingestion front-loads cost and pays off on a repeatedly-queried corpus; ad-hoc analysis suits one-off questions. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:248,270,97-98]
4. **Treat image-borne text as untrusted, and budget for the slow path**, adding a boundary beyond the central [[entities/amazon-bedrock.md|Bedrock]]/IAM credential model and pricing Nova video as the latency outlier. ^[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md:20,97,176]

## Source

> [AWS Machine Learning Blog](https://aws.amazon.com/blogs/machine-learning/agentic-vision-building-visual-intelligence-with-amazon-bedrock-and-mcp-servers)

---
## 关联
→ [[raw/articles/agentic-vision-building-visual-intelligence-with-amazon-bedr.md|原文存档]]
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

