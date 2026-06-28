# Image Processing Queue

A containerized, asynchronous image processing pipeline built with FastAPI, RabbitMQ, MinIO, and Python workers. Users upload an image through a REST API, the file is stored in MinIO, a task is queued in RabbitMQ, and a worker compresses the image and stores the processed version in a separate MinIO bucket.

## Overview

This project demonstrates a practical message-queue-based architecture for image processing without blocking the main application flow. It is ideal for use cases such as:

- Uploading and storing images asynchronously
- Offloading heavy image transformation tasks to background workers
- Building a scalable and modular processing pipeline with Docker

## Features

- FastAPI-based upload endpoint
- Asynchronous task processing with RabbitMQ
- Object storage with MinIO
- Background worker for image compression
- Docker Compose support for quick local deployment
- Automatic bucket creation for original and processed images

## Architecture

```text
Client --> FastAPI API --> MinIO (original-images) --> RabbitMQ --> Worker --> MinIO (processed-images)
```

### Components

- FastAPI API: Accepts image uploads and publishes processing jobs
- RabbitMQ: Delivers tasks to the worker queue
- MinIO: Stores original and processed images
- Worker: Downloads the image, compresses it, and uploads the output

## Tech Stack

- Python 3.11
- FastAPI
- Uvicorn
- RabbitMQ
- MinIO
- Pillow
- Docker Compose

## Project Structure

```text
.
├── api/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── worker/
│   ├── worker.py
│   ├── Dockerfile
│   └── requirements.txt
├── docker-compose.yml
└── README.md
```

## Prerequisites

Make sure you have Docker and Docker Compose installed on your machine.

- Docker: https://docs.docker.com/get-docker/
- Docker Compose: https://docs.docker.com/compose/

## Quick Start

1. Clone the repository

```bash
git clone https://github.com/your-username/Image-Processing-Queue.git
cd Image-Processing-Queue
```

2. Start the services

```bash
docker compose up --build
```

3. Access the services

- API documentation: http://localhost:8000/docs
- MinIO Console: http://localhost:9001
- RabbitMQ Management UI: http://localhost:15672

### Default credentials

- MinIO:
  - Username: minioadmin
  - Password: minioadmin123
- RabbitMQ:
  - Username: user
  - Password: pass

## API Usage

The API exposes a single upload endpoint:

```http
POST /upload
Content-Type: multipart/form-data
```

### Request example

```bash
curl -X POST "http://localhost:8000/upload" \
  -F "file=@/path/to/your-image.jpg"
```

### Response example

```json
{
  "message": "فایل دریافت شد و برای پردازش در صف قرار گرفت",
  "object_name": "1712345678_my-image.jpg"
}
```

The uploaded image is stored in the MinIO bucket named `original-images`, and the processed output will appear in the `processed-images` bucket as a file named `compressed_<original_name>`.

## Environment Variables

The services are configured via environment variables in Docker Compose. The most important ones are:

| Variable | Description | Default |
|---|---|---|
| MINIO_ENDPOINT | MinIO service address | minio:9000 |
| MINIO_ACCESS_KEY | MinIO access key | minioadmin |
| MINIO_SECRET_KEY | MinIO secret key | minioadmin123 |
| MINIO_BUCKET_ORIGINAL | Bucket for uploaded images | original-images |
| MINIO_BUCKET_PROCESSED | Bucket for processed images | processed-images |
| RABBITMQ_HOST | RabbitMQ host | rabbitmq |
| RABBITMQ_PORT | RabbitMQ port | 5672 |
| RABBITMQ_QUEUE | Queue used by the worker | image_tasks |

## How It Works

1. A client uploads an image to the FastAPI endpoint.
2. The API validates the file type and stores it in MinIO.
3. A message containing the object details is published to RabbitMQ.
4. The worker consumes the message, downloads the image from MinIO, compresses it using Pillow, and uploads the processed result back to MinIO.

## Stopping the Services

```bash
docker compose down
```

To remove the persistent MinIO data volume as well:

```bash
docker compose down -v
```

## Notes

- The current worker implementation compresses images to JPEG format with optimized quality settings.
- The API automatically creates the required MinIO buckets on startup.
- OpenAPI documentation is available at http://localhost:8000/docs when the API container is running.

