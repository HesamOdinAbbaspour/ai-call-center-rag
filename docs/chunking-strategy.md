# Audio Chunking Strategy

## Overview

This document explains the strategy used for processing long audio recordings inside our local AI-powered call center RAG pipeline.

The main objective was to achieve stable, high-quality speech-to-text transcription on limited hardware while minimizing system crashes and maintaining acceptable transcription accuracy.


## Initial Problem

Initially, the system attempted to transcribe complete audio recordings using Whisper Large-v3.

Although the model produced excellent transcription quality, recordings longer than approximately 7–8 minutes frequently caused instability due to hardware limitations.

Common issues included:

- GPU memory exhaustion
- Whisper process crashes
- Extremely long processing times
- Pipeline interruption


## System Constraints

The deployment environment was limited by available GPU resources.

Because the system needed to process continuous incoming call recordings, stability was prioritized over maximum inference speed.

The solution therefore focused on reducing peak GPU memory usage while preserving transcription quality.


## Why We Introduced Chunking

Instead of sending entire recordings to Whisper, each audio file was divided into smaller segments before transcription.

This approach significantly reduced GPU memory consumption and prevented failures caused by processing long recordings in a single inference session.


## Chunk Size Experiments

Several chunk durations were evaluated.

Very small chunks increased processing overhead and negatively affected sentence continuity.

Large chunks consumed excessive GPU memory and increased the probability of crashes.

After multiple experiments, chunk durations between 30 and 40 seconds provided the best balance between:

- Stability
- Processing speed
- GPU utilization
- Transcription quality


## Audio Preprocessing

Before transcription, every recording was preprocessed using FFmpeg.

The preprocessing stage reduced unnecessary bitrate and standardized the audio format before entering the transcription pipeline.

This reduced overall processing requirements without noticeably affecting transcription accuracy.


## GPU / CPU Fallback Strategy

Even after chunking, Whisper Large-v3 occasionally became unstable when GPU resources were exhausted.

Instead of terminating the transcription process, a fallback mechanism was introduced.

If GPU inference failed, the remaining chunks were automatically processed on CPU.

Although CPU inference was slower, this strategy dramatically improved pipeline reliability and enabled successful processing of recordings up to approximately ten minutes in length.


## Results

The final pipeline achieved:

- Stable transcription of recordings up to approximately 10 minutes
- Significantly fewer system crashes
- Reliable GPU utilization
- Automatic recovery using CPU fallback
- Persian transcription accuracy above 95% for clear speech


## Lessons Learned

Chunking should not be viewed only as an optimization technique.

For resource-constrained environments, it becomes an essential architectural decision.

System stability often provides greater practical value than maximizing raw inference speed.

Graceful degradation using CPU fallback can significantly improve production reliability.


## Future Improvements

Possible future improvements include:

- Dynamic chunk sizing based on silence detection
- Parallel chunk transcription
- Voice activity detection before chunk generation
- Smarter GPU scheduling
- Multi-GPU support

