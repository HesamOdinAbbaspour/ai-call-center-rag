# Architecture

## Objective

Build a local AI-powered call center knowledge system.

## Pipeline

Isabel
↓
MinIO
↓
n8n
↓
Whisper
↓
Chunking
↓
Embedding
↓
Qdrant
↓
Qwen3

## Components

### Isabel
Source of call recordings.

### MinIO
Object storage for audio files.

### n8n
Workflow orchestration.

### Whisper
Speech-to-text transcription.

### Qdrant
Vector database.

### Qwen3
Local language model.