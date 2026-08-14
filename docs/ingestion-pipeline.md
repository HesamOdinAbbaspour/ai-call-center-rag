# Ingestion Pipeline

> Status: ✅ Implemented

## Overview

This document describes the ingestion pipeline responsible for collecting new call recordings, detecting unique files, processing audio locally, generating normalized transcripts and embeddings, and preparing the resulting data for ingestion into the RAG dataset.

The pipeline is orchestrated by n8n and integrates Issabel, MySQL, MinIO, FFmpeg, Faster-Whisper, and BGE-M3.

## Trigger

The ingestion workflow is triggered based on the factory-defined event mechanism.

Once triggered, the workflow starts by acquiring call recordings and their associated metadata from the Issabel system.

## 1. Issabel Audio & Metadata Acquisition

The workflow reads call recordings from the Issabel system along with their available call metadata.

The metadata is collected together with the audio file information and is preserved throughout the ingestion process.

At this stage, the system does not perform speaker diarization or identify which participant said each part of the conversation.

## 2. MySQL Uniqueness Detection

The workflow queries a MySQL database hosted on the n8n server to determine whether each discovered call recording has already been processed.

The database is used as the processing state store for call recordings.

Each incoming recording is checked against the existing records.

- If the recording already exists, it is treated as a previously processed file and is excluded from further ingestion.
- If the recording is not found, it is classified as a unique file and is passed to the next stage of the workflow.

Only unique recordings are allowed to enter the processing loop.

## 3. Unique File Processing Loop

For each recording identified as unique, the workflow enters a processing loop.

Each unique recording is processed independently through the downstream ingestion stages.

At the beginning of the loop, the unique audio file is stored in MinIO before the local speech-to-text and data-processing stages are executed.

The loop continues through transcription, text processing, metadata generation, embedding, validation, dataset ingestion, database status update, and final MinIO storage.

## 4. MinIO Storage

MinIO is used as the local object storage layer for the ingestion pipeline.

When a recording is identified as unique, the original audio file is stored in MinIO at the beginning of its processing cycle.

MinIO is also used at the end of the pipeline to store the processed RAG-related output using a structure aligned with the RAG dataset.

This provides a persistent local storage layer for both the original audio assets and the processed knowledge-ingestion artifacts.

## 5. Speech-to-Text

The speech-to-text stage is handled locally using Faster-Whisper.

The transcription pipeline was optimized for a resource-constrained deployment environment where processing long recordings with larger Whisper models could lead to instability and process crashes.

Long recordings are divided into smaller audio chunks before transcription. Chunk durations of approximately 30–40 seconds were selected as a practical balance between processing stability, resource usage, and transcription quality.

Audio processing is primarily performed on GPU to achieve better inference performance. When GPU resources become insufficient or transcription fails, processing can fall back to CPU execution to maintain pipeline continuity.

The final configuration has been able to process recordings of up to approximately 10 minutes with improved stability compared with processing long recordings as a single input.

In testing scenarios involving Persian literary text, transcription accuracy reached approximately 95% or higher.

## 6. Text Cleaning & Normalization

The transcription output generated from individual audio chunks is cleaned and normalized before being used in the RAG ingestion pipeline.

This stage prepares the chunk-level transcription output for consistent downstream processing while preserving the semantic content of the original transcription.

The normalized output is then passed to the metadata and transcript generation stage.

## 7. Metadata + Transcript Generation

After transcription and text normalization, the workflow combines the available call metadata with the processed transcript.

A text-based artifact is generated containing the call metadata together with the corresponding transcript content.

This artifact provides a structured representation of the processed call and is used to prepare the chunked content and associated metadata for the RAG ingestion process.

## 8. BGE-M3 Embedding

The normalized transcript chunks are passed to the locally deployed BGE-M3 embedding model.

BGE-M3 converts each processed chunk into a vector representation as part of the RAG ingestion workflow.

The embedding operation is executed locally on the RAG infrastructure and does not depend on an external embedding API.

The generated embeddings are then validated before the processed data is written into the RAG dataset.

## 9. Embedding Validation

After the embedding step, the workflow verifies that the embedding operation has completed successfully before continuing to the RAG dataset ingestion stage.

Only successfully processed embedding results are allowed to proceed.

If the embedding operation is not completed successfully, the workflow does not finalize the ingestion of the corresponding data.

## 10. RAG Dataset Ingestion

After successful embedding validation, the processed transcript chunks are automatically written to the RAG dataset.

Each chunk is ingested together with its associated processed content and metadata required by the RAG ingestion pipeline.

The ingestion process is completed automatically as part of the n8n workflow without requiring manual intervention.

At this stage, the data is prepared for the downstream retrieval layer, while the final vector storage and retrieval mechanism using Qdrant is still planned.

## 11. MySQL Processing Status Update

After the successful completion of the ingestion process, the workflow updates the corresponding call recording record in MySQL.

The database record is used to persist the processing state of the recording and prevent the same file from being processed again as a new input.

The status update is performed as part of the automated n8n workflow after the required ingestion stages have completed successfully.

## 12. MinIO Final Storage

After successful ingestion and processing, the workflow stores the resulting RAG-related artifacts in MinIO using a structure aligned with the RAG dataset.

This final storage step provides a persistent local copy of the processed output and preserves the relationship between the original call recording, its metadata, transcript, and processed RAG artifacts.

