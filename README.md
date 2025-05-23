# Video Translation API Quick Start Guide

Welcome to the Video Translation API! This guide will help you get started with translating videos into different languages using our powerful API.

## Overview

The Video Translation API allows you to:
- Translate content into multiple target languages
- Apply lip-sync to match the translated audio
- Retrieve translated videos with subtitles

## Prerequisites

Before you begin, make sure you have:
- An API key from Perso.ai
- A server where your video/audio files are hosted and downloadable
- Python 3.6+ installed (for the examples)

## Installation

Install the required Python packages:

```bash
pip install requests
```

## Authentication

All API requests require authentication using your API key. Include it in the request headers:

```python
headers = {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "PersoLive-APIKey": "your-api-key-here"
}
```

## Quick Start

Here's a simple example to translate a video from Korean to English:

```python
import requests
import json
import time

# Configuration
input_file_url = "https://your-server.com/video.mp4"
source_language = "ko"
target_language = "en"
api_key = "your-api-key-here"

headers = {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "PersoLive-APIKey": api_key,
}

# Step 1: Create a project
url = "https://live-api.perso.ai/api/video_translator/v2/project/"
payload = json.dumps({
    "input_file_name": "video.mp4",
    "input_file_url": input_file_url,
    "source_language": source_language,
})

response = requests.post(url, headers=headers, data=payload)
project_id = response.json()["project_id"]
print(f"Project created: {project_id}")

# Step 2: Create an export
url = "https://live-api.perso.ai/api/video_translator/v2/export/"
payload = json.dumps({
    "export_type": "INITIAL_EXPORT",
    "priority": 0,
    "server_label": "prod",
    "project": project_id,
    "target_language": target_language,
    "lipsync": False,
    "watermark": True,
})

response = requests.post(url, headers=headers, data=payload)
export_id = response.json()["projectexport_id"]
print(f"Export created: {export_id}")

# Step 3: Check export status
while True:
    time.sleep(5)
    response = requests.get(
        f"https://live-api.perso.ai/api/video_translator/v2/export/{export_id}/",
        headers=headers
    )
    
    data = response.json()
    print(f"Status: {data['status']} - {data['status_detail']}")
    
    if data["status"] == "COMPLETED":
        print(f"Translation completed!")
        print(f"Video URL: {data['video_output_video_without_lipsync']}")
        break
```

## API Reference

### 1. Create Project

Creates a new translation project for your video or audio file.

**Endpoint:** `POST /api/video_translator/v2/project/`

**Request Body:**
```json
{
    "input_file_type": "VIDEO",  // "VIDEO" or "AUDIO"
    "input_file_url": "https://example.com/video.mp4",
    "input_file_name": "video.mp4",  // Must be .mp4 or .webm
    "source_language": "ko"
}
```

**Response:**
```json
{
    "project_id": "proj_abc123",
    // ... other project details
}
```

### 2. Create Export

Initiates the translation process for a specific target language.

**Endpoint:** `POST /api/video_translator/v2/export/`

**Request Body:**
```json
{
    "project": "proj_abc123",
    "server_label": "prod",
    "priority": 0,
    "target_language": "en",
    "export_type": "INITIAL_EXPORT",
    "lipsync": false,
    "watermark": true
}
```

**Response:**
```json
{
    "projectexport_id": "exp_xyz789",
    // ... other export details
}
```

### 3. Check Export Status

Retrieves the current status of your translation export.

**Endpoint:** `GET /api/video_translator/v2/export/{export_id}/`

**Response:**
```json
{
    "status": "COMPLETED",
    "status_detail": "Translation finished successfully",
    "video_output_video_without_lipsync": "https://...",
    "video_output_video_with_lipsync": "https://...",  // If lipsync=true
    "audio_subtitle_original": "https://...",
    "audio_subtitle_translated": "https://...",
    // ... other output files
}
```

## Advanced Features

### Modifying Translations

You can edit the translated text before generating the final video:

```python
# Get project details
url = f"https://live-api.perso.ai/api/video_translator/v2/project/{project_id}/"
response = requests.get(url, headers=headers)
scripts = response.json()["scripts"]

# Update a translation
script_id = scripts[0]["projectscript_id"]
url = f"https://live-api.perso.ai/api/video_translator/v2/script/{script_id}/"
payload = json.dumps({
    "text_translated": "Your improved translation here"
})
response = requests.patch(url, headers=headers, data=payload)

# Generate audio for the updated translation
url = f"https://live-api.perso.ai/api/video_translator/v2/script/{script_id}/generate_audio/"
response = requests.post(url, headers=headers)

# Create a new export with the updated translation
url = "https://live-api.perso.ai/api/video_translator/v2/export/"
payload = json.dumps({
    "export_type": "PROOFREAD_EXPORT",
    "server_label": "prod",
    "priority": 0,
    "project": project_id,
    "target_language": target_language,
    "lipsync": False,
    "watermark": True,
})
response = requests.post(url, headers=headers, data=payload)
```

### Lip Sync

Enable lip synchronization for more natural-looking videos:

```python
payload = json.dumps({
    # ... other parameters
    "lipsync": True,
})
```

When lip sync is enabled, the API will generate a video where the speaker's lip movements match the translated audio.

## Status Values

The export status can be one of:
- `PENDING`: Export is queued
- `PROCESSING`: Translation in progress
- `COMPLETED`: Translation successful
- `FAILED`: Translation failed (check `failure_reason`)

## Error Handling

Always check the response status code and handle errors appropriately:

```python
response = requests.post(url, headers=headers, data=payload)

if response.status_code == 200:
    data = response.json()
    # Process successful response
elif response.status_code == 401:
    print("Authentication failed. Check your API key.")
elif response.status_code == 400:
    print("Bad request. Check your input parameters.")
else:
    print(f"Error: {response.status_code} - {response.text}")
```

## Rate Limits

Please be aware of rate limits when using the API. If you encounter rate limit errors, implement exponential backoff in your retry logic.

## Supported Languages

The API supports a wide range of source and target languages. Common language codes include:
- `en` - English
- `ko` - Korean
- `ja` - Japanese
- `zh` - Chinese
- `es` - Spanish
- `fr` - French
- `de` - German
- `ta` - Tamil
- `hi` - Hindi

For a complete list of supported languages, please refer to our API documentation.

## Best Practices

1. **File Hosting**: Ensure your video files are hosted on a reliable server with good download speeds
2. **File Formats**: Use MP4 or WebM formats for best compatibility
3. **Polling**: When checking export status, poll every 5-10 seconds to avoid overwhelming the API
4. **Error Handling**: Always implement proper error handling and retry logic
5. **Translation Review**: Consider reviewing and editing translations before final export for best quality

## Next Steps

- Explore the [full API documentation](https://live-api.perso.ai/api/schema/swagger-ui/)
- Try translating videos with different target languages
- Experiment with lip sync for more engaging videos
- Integrate the API into your application

## Support

If you have any questions or need assistance, please contact our support team or visit our documentation portal.

Happy translating! 🎬🌍
