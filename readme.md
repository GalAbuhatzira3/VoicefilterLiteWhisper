# VoiceFilter-Lite with Whisper (Target Speaker ASR)

This project implements a reproduction and adaptation of the VoiceFilter-Lite model for target-speaker speech enhancement, integrated with OpenAI Whisper for automatic speech recognition (ASR).

The system enhances speech from a target speaker in a multi-speaker mixture using a speaker embedding (d-vector), and improves transcription accuracy without retraining the ASR model.

## Key Features
- Speaker-conditioned speech enhancement (VoiceFilter-Lite)
- Integration with frozen Whisper (small.en)
- Training on Libri2Mix-style two-speaker mixtures
- Combined loss:
  - Asymmetric feature reconstruction loss
  - Frame-level overlap classification loss
- Adaptive suppression at inference time
- Full training pipeline:
  - Adam optimizer + warm-up + cosine decay
  - Gradient clipping
  - Checkpointing & TensorBoard logging

## Results
- Improved WER compared to noisy mixtures
- Strong degradation under wrong-speaker conditioning (validating speaker-awareness)
- Remaining gap to clean condition highlights limitations of frontend-only approach

## Tech Stack
- Python, PyTorch
- Whisper (ASR)
- SpeechBrain (speaker embeddings)
- Libri2Mix dataset