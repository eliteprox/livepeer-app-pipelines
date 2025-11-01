# ComfyStream integration with Livepeer BYOC streaming

ComfyUI streaming processor using the pre-built `livepeer/comfyui-base` image for real-time video/audio processing with ComfyUI workflows.

## Features

- Real-time video and audio processing with ComfyUI workflows
- WebRTC streaming support
- Adaptive frame skipping
- Loading overlay during warmup
- Text output support (for workflows that produce text)
- Automatic orchestrator registration

## Setup

1. Copy `.env.worker` to `.env` and update variables:
   - `ORCH_URL`: Orchestrator URL (e.g., `https://dev.eliteencoder.net:9995`)
   - `ORCH_SECRET`: Orchestrator secret
   - `CAPABILITY_NAME`: Name for the capability (default: `comfystream`)
   - `CAPABILITY_URL`: URL where the runner is accessible (e.g., `http://dev.eliteencoder.net:8000`)
   - `CAPABILITY_CAPACITY`: Maximum concurrent streams (default: 1)
   - `CAPABILITY_PRICE_PER_UNIT`: Price per unit (default: 0)
   - `PORT`: Port to expose (default: 8000)

2. Ensure ComfyUI model directories exist:
```bash
mkdir -p ~/models/ComfyUI--models ~/models/ComfyUI--output
```

## Launch 

```bash
# Start the ComfyStream service
docker compose up -d

# View logs
docker compose logs -f

# Stop the service
docker compose down
```

## Using a Different Image

To use a different version of the ComfyUI base image, update the `image` field in `docker-compose.yml`:

```yaml
services:
  comfystream:
    image: livepeer/comfyui-base:latest  # or any other tag
```

## Configuration

All configuration is done via environment variables in `.env`:

- `ORCH_URL`: Orchestrator URL for registration
- `ORCH_SECRET`: Secret for orchestrator authentication
- `CAPABILITY_NAME`: Unique name for this capability
- `CAPABILITY_DESCRIPTION`: Human-readable description
- `CAPABILITY_URL`: Public URL where this service is accessible
- `CAPABILITY_PRICE_PER_UNIT`: Pricing for the service
- `CAPABILITY_PRICE_SCALING`: Price scaling factor
- `CAPABILITY_CAPACITY`: Maximum concurrent streams

## Volume Mounts

The service mounts two directories:
- `~/models/ComfyUI--models`: ComfyUI models (checkpoints, LoRAs, etc.)
- `~/models/ComfyUI--output`: Output directory for generated files

You can change these paths in `docker-compose.yml` if needed.

## API Endpoints

The service exposes the same endpoints as the standard ComfyStream server:

### Warmup
`POST /api/stream/warmup`

Warm up the pipeline with optional parameters:
```json
{
  "prompts": { /* ComfyUI workflow */ },
  "width": 512,
  "height": 512
}
```

### Update Parameters
`POST /api/stream/params`

Update processing parameters during streaming:
```json
{
  "prompts": { /* ComfyUI workflow */ },
  "width": 512,
  "height": 512
}
```

## Troubleshooting

### GPU not detected
Ensure NVIDIA Container Toolkit is installed:
```bash
docker run --rm --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi
```

### Network issues
Make sure the `byoc-stream` network exists:
```bash
docker network create byoc-stream
```

### Port conflicts
Change the port mapping in `.env`:
```bash
PORT=8001
```
