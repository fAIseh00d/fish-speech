# Fish-Speech TTS Server
# Simplified Dockerfile using PyTorch base image

FROM pytorch/pytorch:2.10.0-cuda12.6-cudnn9-runtime

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg \
    libsox-dev \
    portaudio19-dev \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies
COPY pyproject.toml uv.lock README.md ./
COPY . .

# Install fish-speech
RUN pip install --no-cache-dir --break-system-packages -e ".[cu126]"

# Expose API port
EXPOSE 8003

# Environment - paths inside HuggingFace cache
ENV HF_HOME="/root/.cache/huggingface"
ENV LLAMA_CHECKPOINT_PATH="/root/.cache/huggingface/hub/models--fishaudio--s1-mini/snapshots/latest"
ENV DECODER_CHECKPOINT_PATH="/root/.cache/huggingface/hub/models--fishaudio--s1-mini/snapshots/latest/codec.pth"
ENV DECODER_CONFIG_NAME="modded_dac_vq"

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=10 --start-period=180s \
    CMD curl -f http://localhost:8003/v1/health || exit 1

# Run the API server - use shell form to expand ENV variables
CMD python tools/api_server.py \
    --listen 0.0.0.0:8003 \
    --llama-checkpoint-path "$LLAMA_CHECKPOINT_PATH" \
    --decoder-checkpoint-path "$DECODER_CHECKPOINT_PATH" \
    --decoder-config-name "$DECODER_CONFIG_NAME"
