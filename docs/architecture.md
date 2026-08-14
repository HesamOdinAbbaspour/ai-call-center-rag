# Architecture

## Objective

Build a local, event-driven and agentic AI pipeline for ingesting call-center recordings, processing speech-to-text data, generating embeddings, and preparing structured knowledge for a RAG-based system.

## Pipeline

Issabel
↓
MySQL Metadata & Uniqueness Check
↓
n8n Agentic Workflow
↓
MinIO
↓
FFmpeg / Audio Preprocessing
↓
Audio Chunking
↓
Faster-Whisper
↓
Text Cleaning & Normalization
↓
Metadata + Transcript Generation
↓
BGE-M3 Embedding
↓
Embedding Validation
↓
RAG Dataset
↓
MySQL Processing Status Update

### Planned Retrieval Layer

RAG Dataset
↓
Qdrant
↓
Qwen3-4B-Instruct

## Components

### Issabel
Source system for call recordings and associated call metadata.

### MySQL
Runs on the n8n server and is used to identify unique call recordings and track processing status.

### n8n
Agentic workflow orchestration layer responsible for coordinating the ingestion and processing pipeline, including uniqueness checks, looping over new recordings, model execution, validation, and status updates.

### MinIO
Object storage used to store unique call recordings and processed RAG-related assets.

### FFmpeg
Used during audio preprocessing to reduce file size and standardize audio input before transcription.

### Audio Chunking
Long recordings are divided into smaller segments to improve transcription stability and reduce peak resource usage.

### Faster-Whisper
Local speech-to-text engine used to transcribe the audio chunks.

### Text Cleaning & Normalization
Processes the transcription output before it is converted into the final structured text representation used by the RAG ingestion pipeline.

### BGE-M3
Locally deployed embedding model used to convert normalized transcript chunks into vector representations.

### RAG Dataset
Receives processed chunks and their embeddings after successful embedding validation.

### Qwen3-4B-Instruct
Locally deployed language model intended for the downstream RAG generation layer.

### Qdrant
Planned vector database for the retrieval layer.
