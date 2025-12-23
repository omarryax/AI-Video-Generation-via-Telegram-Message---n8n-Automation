# Telegram → 4 Video Generators Workflow

## 📋 Project Overview

This project contains an **n8n automation workflow** that automatically generates videos from Telegram messages using AI-powered video generation services. The workflow receives text prompts or images from Telegram, splits them into 4 sequential video scenes using GPT-4, generates videos using SORA models, concatenates them into a single video, and sends the result back to Telegram.

### Key Features

- 🤖 **AI-Powered Prompt Engineering**: Uses GPT-4 to intelligently split prompts into 4 sequential video scenes
- 🎬 **Multiple Video Generation**: Supports both image-to-video and text-to-video generation using SORA models
- 📱 **Telegram Integration**: Seamless integration with Telegram for receiving inputs and delivering outputs
- 🎞️ **Video Concatenation**: Automatically stitches 4 video scenes into a single cohesive video using FFmpeg
- ⚡ **Parallel Processing**: Generates multiple videos simultaneously for faster processing
- 🔄 **Status Monitoring**: Polls video generation status and handles retries automatically

---

## 🗂️ Project Files

- **`Telegram → 4 Video Generators copy (3).json`** - Main n8n workflow file (ready to import)
- **`README.md`** - This documentation file

---

## 🔄 Workflow Process

### High-Level Flow

```
1. Telegram Message Received (Text or Image)
   ↓
2. AI Transform (Clean prompt, remove line breaks)
   ↓
3. GPT-4 Prompt Engineering (Split into 4 sequential scenes)
   ↓
4. Parallel Video Generation (4 videos using SORA models)
   ├─→ Scene 1 (Image-to-Video or Text-to-Video)
   ├─→ Scene 2 (Image-to-Video or Text-to-Video)
   ├─→ Scene 3 (Image-to-Video or Text-to-Video)
   └─→ Scene 4 (Image-to-Video or Text-to-Video)
   ↓
5. Status Polling & Video Download
   ↓
6. Video Concatenation (FFmpeg)
   ↓
7. Send Final Video to Telegram
```

### Detailed Steps

1. **Telegram Trigger**
   - Listens for new messages in Telegram
   - Downloads images automatically if present
   - Extracts text from message or caption

2. **Input Processing**
   - **AI Transform**: Cleans the prompt by removing line breaks
   - **Conditional Logic**: Checks if message contains an image (photo) or is text-only

3. **Prompt Engineering**
   - **GPT-4 API Call**: Splits the input prompt into exactly 4 sequential video prompts
   - Ensures visual continuity, consistent style, and natural transitions
   - Returns structured JSON with 4 scene prompts

4. **Video Generation**
   - **Image Path**: If image exists → Generates 4 image-to-video scenes
   - **Text Path**: If no image → Generates 4 text-to-video scenes
   - Uses SORA models:
     - `openai/sora-2/image-to-video-developer`
     - `openai/sora-2/text-to-video-developer`
   - All 4 videos generated in parallel for efficiency

5. **Status Monitoring**
   - Polls video generation status every 60 seconds
   - Filters for "created" status initially
   - Waits for "completed" status
   - Handles failed generations gracefully

6. **Video Download**
   - Downloads completed videos from API
   - Saves to `/home/node/.n8n-files/` directory
   - Files named: `video0.mp4`, `video1.mp4`, `video2.mp4`, `video3.mp4`

7. **Video Concatenation**
   - Uses FFmpeg to concatenate all 4 videos
   - Creates `videos.txt` with file list
   - Generates final `output.mp4`

8. **Telegram Delivery**
   - Reads the concatenated video file
   - Sends video back to Telegram with model name as caption

---

## 🚀 Setup Instructions

### Prerequisites

1. **n8n Installation**
   - Install n8n (self-hosted or cloud)
   - Ensure n8n version 1.0+ is installed
   - FFmpeg must be installed on the n8n server for video concatenation

2. **Required Services & API Keys**
   - **Telegram Bot Token**: Create a bot via [@BotFather](https://t.me/botfather)
   - **Video Generation API**: API key for SORA video generation service
   - **OpenAI/GPT-4 API**: API key for prompt engineering (or compatible service)
   - **File System Access**: Write permissions to `/home/node/.n8n-files/` directory

### Installation Steps

1. **Import Workflow**
   ```
   1. Open n8n
   2. Go to Workflows → Import from File
   3. Select "Telegram → 4 Video Generators copy (3).json"
   4. Click Import
   ```

2. **Configure Telegram Credentials**
   - Go to Credentials → Add Credential
   - Select "Telegram"
   - Enter your Bot Token from BotFather
   - Update the `chatId` in "Send Videos to Telegram1" node with your Telegram chat ID

3. **Configure API Endpoints**
   
   Update all HTTP Request nodes with your API endpoints:
   
   - **Video Generation API**:
     - Update URL in all "SORA POST" nodes (currently set to placeholder)
     - Add your API key in Authorization header: `Bearer YOUR_API_KEY_HERE`
     - Example: `https://api.atlascloud.ai/api/v1/model/generateVideo` (or your service URL)
   
   - **GPT-4 API**:
     - Update URL in "HTTP Request" node (for prompt engineering)
     - Add your API key in Authorization header
     - Ensure it supports GPT-4 or compatible model

4. **Configure File System**
   - Ensure `/home/node/.n8n-files/` directory exists
   - Grant write permissions to n8n user
   - Verify FFmpeg is installed: `ffmpeg -version`

5. **Update Node Credentials**
   - Click on "Telegram Trigger1" node
   - Select your Telegram credential
   - Click on "Send Videos to Telegram1" node
   - Select your Telegram credential
   - Update `chatId` parameter with your Telegram chat ID

6. **Test the Workflow**
   - Use "Test workflow" button in n8n
   - Send a test message to your Telegram bot
   - Check execution logs for errors
   - Verify video generation and delivery

---

## 📊 Workflow Structure

### Node Types Used

#### Integration Nodes

- **Telegram** (`n8n-nodes-base.telegramTrigger`, `n8n-nodes-base.telegram`)
  - Trigger: Receives new messages
  - Action: Sends videos back to Telegram

- **HTTP Request** (`n8n-nodes-base.httpRequest`)
  - Video generation API calls (SORA POST nodes)
  - GPT-4 API calls for prompt engineering
  - Video download requests

#### AI & Processing Nodes

- **AI Transform** (`n8n-nodes-base.aiTransform`)
  - Cleans and processes prompts
  - Removes line breaks from text

- **Set** (`n8n-nodes-base.set`)
  - Parses GPT-4 response into structured JSON
  - Extracts scene prompts

#### Control Flow Nodes

- **If** (`n8n-nodes-base.if`)
  - Checks if message contains photo
  - Routes to image-to-video or text-to-video path
  - Validates video generation status

- **Filter** (`n8n-nodes-base.filter`)
  - Filters for "created" status
  - Filters for "completed" status

- **Wait** (`n8n-nodes-base.wait`)
  - Waits 60 seconds between status checks
  - Allows video generation to complete

- **Merge** (`n8n-nodes-base.merge`)
  - Combines image and prompt data

#### File Operations

- **Extract from File** (`n8n-nodes-base.extractFromFile`)
  - Extracts image data from Telegram message

- **Read/Write Files** (`n8n-nodes-base.readWriteFile`)
  - Writes downloaded videos to disk
  - Reads concatenated video file

- **Execute Command** (`n8n-nodes-base.executeCommand`)
  - Runs FFmpeg to concatenate videos

### Node Count

- **Total Nodes**: ~30+ nodes
- **Trigger Nodes**: 1 (Telegram)
- **HTTP Request Nodes**: 8+ (Video generation + GPT-4)
- **Control Nodes**: Multiple (If, Filter, Wait, Merge)
- **File Nodes**: Multiple (Read/Write, Extract, Execute)

---

## ⚙️ Configuration Details

### Key Parameters

#### Telegram Trigger
- **Event**: `message`
- **Download**: Enabled (for images)
- **Updates**: `["message"]`

#### Video Generation API
- **Model Options**:
  - `openai/sora-2/image-to-video-developer` (for image inputs)
  - `openai/sora-2/text-to-video-developer` (for text inputs)
- **Duration**: 1 second per scene
- **Size**: `720*1280` (portrait orientation)
- **Retry**: Enabled (2 max tries)

#### GPT-4 Prompt Engineering
- **Model**: `openai/gpt-4.1-nano-developer` (or compatible)
- **Task**: Split prompt into 4 sequential video scenes
- **Output Format**: JSON with scenes array
- **Max Tokens**: 1500
- **Temperature**: 1

#### Video Concatenation
- **Command**: FFmpeg concat demuxer
- **Input Files**: `video0.mp4`, `video1.mp4`, `video2.mp4`, `video3.mp4`
- **Output File**: `output.mp4`
- **Method**: Stream copy (no re-encoding for speed)

---

## ⚠️ Important Notes

### Manual Configuration Required

1. **API Endpoints**
   - All HTTP Request nodes currently have placeholder URLs
   - Update with your actual video generation API endpoint
   - Update GPT-4 API endpoint if using different service

2. **API Keys**
   - Replace `YOUR_API_KEY_HERE` in all Authorization headers
   - Ensure API keys have proper permissions
   - Keep API keys secure (use n8n credentials when possible)

3. **Telegram Configuration**
   - Update `chatId` in "Send Videos to Telegram1" node
   - Ensure bot has permission to send videos
   - Test bot token is valid

4. **File System**
   - Verify `/home/node/.n8n-files/` directory exists
   - Ensure n8n has write permissions
   - Check disk space for video storage

5. **FFmpeg Installation**
   - FFmpeg must be installed on n8n server
   - Verify with: `ffmpeg -version`
   - Ensure FFmpeg is in system PATH

6. **Video Generation Service**
   - Confirm your API supports the SORA models used
   - Verify API response format matches expected structure
   - Check rate limits and quotas

### Workflow Behavior

- **Parallel Processing**: All 4 videos are generated simultaneously
- **Status Polling**: Checks status every 60 seconds
- **Error Handling**: Retries failed requests (2 max tries)
- **Video Quality**: Output is 720x1280 (portrait) for mobile viewing
- **Processing Time**: Depends on video generation API speed (typically 1-5 minutes per video)

---

## 🔍 Troubleshooting

### Common Issues

1. **Telegram Bot Not Responding**
   - **Problem**: Bot doesn't receive messages
   - **Solution**: 
     - Verify bot token is correct
     - Check webhook is properly configured
     - Ensure bot is started in n8n

2. **Video Generation Fails**
   - **Problem**: API returns errors
   - **Solution**:
     - Verify API endpoint URL is correct
     - Check API key is valid and has credits
     - Review API response format
     - Check rate limits

3. **FFmpeg Command Fails**
   - **Problem**: Video concatenation fails
   - **Solution**:
     - Verify FFmpeg is installed
     - Check file paths are correct
     - Ensure all 4 videos downloaded successfully
     - Verify file permissions

4. **Videos Not Downloading**
   - **Problem**: Download requests fail
   - **Solution**:
     - Check video generation status is "completed"
     - Verify download URL is accessible
     - Check network connectivity
     - Review authorization headers

5. **GPT-4 Prompt Engineering Fails**
   - **Problem**: Scene prompts not generated correctly
   - **Solution**:
     - Verify GPT-4 API endpoint and key
     - Check prompt format matches expected structure
     - Review API response parsing
     - Adjust temperature if needed

6. **File System Errors**
   - **Problem**: Cannot write/read files
   - **Solution**:
     - Check directory exists: `/home/node/.n8n-files/`
     - Verify write permissions
     - Check disk space
     - Review n8n user permissions

---

## 📝 Usage Examples

### Example 1: Text-to-Video

1. Send a text message to your Telegram bot:
   ```
   A beautiful sunset over the ocean, waves gently crashing on the shore, 
   seagulls flying in the distance, and a sailboat on the horizon
   ```

2. Workflow will:
   - Split into 4 sequential scenes
   - Generate 4 text-to-video clips
   - Concatenate into one video
   - Send back to Telegram

### Example 2: Image-to-Video

1. Send an image with caption to your Telegram bot:
   ```
   Transform this image into a dynamic scene with movement and life
   ```

2. Workflow will:
   - Extract the image
   - Use image as base for 4 video scenes
   - Generate 4 image-to-video clips
   - Concatenate into one video
   - Send back to Telegram

---

## 🔧 Customization

### Adjust Video Parameters

- **Duration**: Change `duration` parameter in SORA POST nodes (currently 1 second)
- **Size**: Modify `size` parameter (currently `720*1280`)
- **Model**: Switch between different SORA models if available

### Modify Scene Count

- Update GPT-4 prompt to generate different number of scenes
- Adjust FFmpeg command to concatenate different number of videos
- Update file naming pattern

### Add Error Notifications

- Add Slack/Discord webhook nodes for error notifications
- Send alerts when video generation fails
- Notify on successful completion

---

## 📚 Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [SORA Model Information](https://openai.com/sora)
- [OpenAI API Documentation](https://platform.openai.com/docs)

---

## 📄 License

This workflow is provided as-is for personal or commercial use. Ensure compliance with:
- Video generation API terms of service
- OpenAI API usage policies
- Telegram Bot API terms
- Data privacy regulations

---

## 👤 Support

For issues or questions:

1. Check n8n execution logs for detailed error messages
2. Review node-specific documentation
3. Verify all API services are operational
4. Test individual nodes in isolation
5. Consult n8n community forums: [community.n8n.io](https://community.n8n.io)

---

## 🔄 Version History

- **v1.0** (Current)
  - Initial workflow implementation
  - Supports 4 video scene generation
  - Image-to-video and text-to-video modes
  - FFmpeg video concatenation
  - Telegram integration

---

**Last Updated**: January 2025  
**Workflow Version**: 1.0  
**n8n Compatibility**: n8n 1.0+  
**Required Services**: Telegram Bot, Video Generation API, GPT-4 API, FFmpeg
