# nexa_ai_flutter

A comprehensive Flutter plugin for the Nexa AI SDK, enabling on-device AI inference for Android applications. Run Large Language Models (LLMs), Vision-Language Models (VLMs), Embeddings, Speech Recognition (ASR), Reranking, and Computer Vision models directly on Android devices with support for NPU, GPU, and CPU inference.

## Features

- **6 Model Types Supported**:
  - 🤖 **LLM (Large Language Models)**: Text generation and chat with streaming support
  - 🖼️ **VLM (Vision-Language Models)**: Multimodal AI with image and audio understanding
  - 🔢 **Embeddings**: Generate vector embeddings for semantic search and RAG
  - 🎤 **ASR (Automatic Speech Recognition)**: Transcribe audio to text
  - 📊 **Reranker**: Improve search relevance by reranking documents
  - 👁️ **Computer Vision**: OCR, object detection, and image classification

- **Built-in Model Downloader**: Download and manage official Nexa AI models with progress tracking
- **Hardware Acceleration**: NPU (Qualcomm Hexagon), GPU (Adreno), and CPU (ARM64-v8a) support
- **Real-time Streaming**: Token-by-token streaming for LLM and VLM models
- **Builder Pattern API**: Clean, fluent API design matching the native SDK
- **Storage Management**: Track model storage, delete models, and manage downloads
- **Comprehensive Error Handling**: Detailed error messages and proper exception handling

## Installation

Add this to your package's `pubspec.yaml` file:

```yaml
dependencies:
  nexa_ai_flutter: ^0.0.1
```

Then run:
```bash
flutter pub get
```

## Platform Requirements

### Android
- **Min SDK**: 27 (Android 8.1)
- **Compile SDK**: 36
- **Kotlin**: 2.1.0+
- **Gradle**: 8.9.1+

### Supported Devices
- **NPU**: Qualcomm Snapdragon 8 Gen 4
- **GPU**: Qualcomm Adreno GPU
- **CPU**: ARM64-v8a

## Getting Started

### 1. Initialize the SDK

```dart
import 'package:nexa_ai_flutter/nexa_ai_flutter.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Initialize the Nexa SDK
  await NexaSdk.getInstance().init();

  runApp(MyApp());
}
```

## Model Management

The plugin includes a built-in model downloader for easy access to official Nexa AI models.

### Downloading Pre-trained Models

#### 1. Get Available Models

```dart
// Get all available models
final models = await ModelDownloader.getAvailableModels();

for (var model in models) {
  print('${model.displayName} - ${model.sizeGb} GB');
  print('Type: ${model.type}');
  print('Features: ${model.features.join(", ")}');
}

// Filter by type
final vlmModels = await ModelDownloader.getModelsByType('multimodal');
```

#### 2. Download a Model with Progress

```dart
ModelDownloader downloader = ModelDownloader();

downloader.downloadModel('OmniNeural-4B').listen(
  (progress) {
    if (progress.status == DownloadStatus.downloading) {
      print('Downloading: ${progress.percentage}%');
      print('Speed: ${progress.speedMBps.toStringAsFixed(2)} MB/s');
      print('${progress.downloadedBytes} / ${progress.totalBytes} bytes');
    } else if (progress.status == DownloadStatus.completed) {
      print('Download completed!');
    } else if (progress.status == DownloadStatus.failed) {
      print('Download failed');
    } else if (progress.status == DownloadStatus.cancelled) {
      print('Download cancelled');
    }
  },
  onError: (error) {
    print('Error: $error');
  },
);
```

#### 3. Cancel Download

```dart
// Cancel an ongoing download
await ModelDownloader.cancelDownload('OmniNeural-4B');
```

#### 4. Check if Model is Downloaded

```dart
bool isDownloaded = await ModelDownloader.isModelDownloaded('OmniNeural-4B');

if (isDownloaded) {
  String? path = await ModelDownloader.getModelPath('OmniNeural-4B');
  print('Model available at: $path');
}
```

#### 5. Use Downloaded Model

```dart
// Get the model path
String? modelPath = await ModelDownloader.getModelPath('OmniNeural-4B');

if (modelPath != null) {
  // Create VLM with the downloaded model
  final vlm = await VlmWrapper.create(
    VlmCreateInput(
      modelName: 'omni-neural',
      modelPath: modelPath,
      config: ModelConfig(maxTokens: 2048),
      pluginId: 'npu',
    ),
  );

  // Use the model...
  // Remember to call vlm.destroy() when done
}
```

#### 6. Manage Storage

```dart
// Get storage information
final storage = await ModelDownloader.getStorageInfo();
print('Total: ${storage.totalSpaceGB.toStringAsFixed(2)} GB');
print('Free: ${storage.freeSpaceGB.toStringAsFixed(2)} GB');
print('Used by models: ${storage.usedByModelsGB.toStringAsFixed(2)} GB');
print('Downloaded models: ${storage.downloadedModels.join(", ")}');

// Delete a model to free space
await ModelDownloader.deleteModel('OmniNeural-4B');

// Get list of all downloaded models
List<String> downloaded = await ModelDownloader.getDownloadedModels();

// Get models directory path
String modelsDir = await ModelDownloader.getModelsDirectory();

// Clean up incomplete downloads
await ModelDownloader.cleanupIncompleteDownloads();
```

### Available Pre-configured Models

The plugin includes 8 pre-configured models from Nexa AI:

| Model ID | Name | Type | Size | Features |
|----------|------|------|------|----------|
| OmniNeural-4B | OmniNeural-4B | VLM | 4 GB | Chat, Vision |
| paddleocr-npu | PaddleOCR NPU | CV | 0.25 GB | OCR |
| parakeet-tdt-0.6b-v3-npu | Parakeet ASR | ASR | 0.6 GB | Speech-to-Text |
| embeddinggemma-300m-npu | Embedding Gemma | Embedder | 0.25 GB | Embeddings |
| jina-v2-rerank-npu | Jina Reranker | Reranker | 1.0 GB | Reranking |
| LFM2-1.2B-npu | LFM2 Chat | LLM | 0.75 GB | Chat |
| SmolVLM-256M-Instruct-f16 | SmolVLM | VLM | 0.48 GB | Chat, Vision (GGUF) |
| LFM2-1.2B-GGUF-GGUF | LFM2 GGUF | LLM | 0.75 GB | Chat (GGUF) |

**Note:** Models are downloaded to `/data/data/{your_package}/files/models/` on Android.

### 2. LLM - Large Language Models

```dart
// Create LLM wrapper
final llm = await LlmWrapper.create(
  LlmCreateInput(
    modelPath: '/path/to/model.gguf',
    config: ModelConfig(
      nCtx: 4096,
      maxTokens: 2048,
    ),
    pluginId: 'cpu_gpu', // or 'npu' for NPU inference
  ),
);

// Apply chat template
final messages = [
  ChatMessage('user', 'What is AI?'),
];
final template = await llm.applyChatTemplate(messages);

// Generate with streaming
llm.generateStream(
  template.formattedText,
  GenerationConfig(maxTokens: 2048),
).listen((result) {
  if (result is LlmStreamToken) {
    print(result.text); // Token-by-token output
  } else if (result is LlmStreamCompleted) {
    print('TTFT: ${result.profile.ttftMs} ms');
    print('Speed: ${result.profile.decodingSpeed} t/s');
  } else if (result is LlmStreamError) {
    print('Error: ${result.message}');
  }
});

// Clean up
await llm.destroy();
```

### 3. VLM - Vision-Language Models

```dart
// Create VLM wrapper
final vlm = await VlmWrapper.create(
  VlmCreateInput(
    modelName: 'omni-neural', // For NPU models
    modelPath: '/path/to/model',
    config: ModelConfig(
      maxTokens: 2048,
      enableThinking: false,
    ),
    pluginId: 'npu',
  ),
);

// Create multimodal message
final message = VlmChatMessage('user', [
  VlmContent('image', '/path/to/image.jpg'),
  VlmContent('text', 'What is in this image?'),
]);

// Apply chat template and inject media paths
final template = await vlm.applyChatTemplate([message]);
final config = vlm.injectMediaPathsToConfig(
  [message],
  GenerationConfig(maxTokens: 2048),
);

// Generate response
vlm.generateStream(template.formattedText, config).listen((result) {
  // Handle streaming results
});

await vlm.destroy();
```

### 4. Embeddings

```dart
// Create embedder
final embedder = await EmbedderWrapper.create(
  EmbedderCreateInput(
    modelName: 'embed-gemma', // For NPU models
    modelPath: '/path/to/embedder',
    config: ModelConfig(),
    pluginId: 'npu',
  ),
);

// Generate embeddings
final texts = ['Hello world', 'AI is awesome'];
final embeddings = await embedder.embed(
  texts,
  EmbeddingConfig(normalize: true),
);

// Calculate dimensions
final dimension = embeddings.length ~/ texts.length;
print('Embedding dimension: $dimension');

await embedder.destroy();
```

### 5. ASR - Automatic Speech Recognition

```dart
// Create ASR wrapper
final asr = await AsrWrapper.create(
  AsrCreateInput(
    modelName: 'parakeet', // For NPU models
    modelPath: '/path/to/asr',
    config: ModelConfig(),
    pluginId: 'npu',
  ),
);

// Transcribe audio
final result = await asr.transcribe(
  AsrTranscribeInput(
    audioPath: '/path/to/audio.wav',
    language: 'en',
  ),
);

print('Transcription: ${result.result.transcript}');

await asr.destroy();
```

### 6. Reranker

```dart
// Create reranker
final reranker = await RerankerWrapper.create(
  RerankerCreateInput(
    modelName: 'jina-rerank', // For NPU models
    modelPath: '/path/to/reranker',
    config: ModelConfig(),
    pluginId: 'npu',
  ),
);

// Rerank documents
final query = 'What is machine learning?';
final documents = [
  'ML is a subset of AI',
  'Weather forecast for tomorrow',
  'Deep learning tutorial',
];

final result = await reranker.rerank(
  query,
  documents,
  RerankConfig(topN: 3),
);

// Display results sorted by relevance
for (var i = 0; i < result.scores.length; i++) {
  print('Score: ${result.scores[i].toStringAsFixed(4)} - ${documents[i]}');
}

await reranker.destroy();
```

### 7. Computer Vision

```dart
// Create CV wrapper
final cv = await CvWrapper.create(
  CVCreateInput(
    modelName: 'paddleocr',
    config: CVModelConfig(
      capabilities: CVCapability.ocr,
      detModelPath: '/path/to/detection',
      recModelPath: '/path/to/recognition',
      charDictPath: '/path/to/dictionary',
    ),
    pluginId: 'npu',
  ),
);

// Perform OCR
final results = await cv.infer('/path/to/image.jpg');

for (var result in results) {
  print('Text: ${result.text}');
  print('Confidence: ${result.confidence}');
  if (result.boundingBox != null) {
    print('Position: (${result.boundingBox!.x}, ${result.boundingBox!.y})');
  }
}

await cv.destroy();
```

## Model Name Mapping (NPU Models)

For NPU models, use these model names and plugin IDs:

| Model Name | Plugin ID | HuggingFace Repository |
|------------|-----------|------------------------|
| omni-neural | npu | NexaAI/OmniNeural-4B-mobile |
| embed-gemma | npu | NexaAI/embeddinggemma-300m-npu-mobile |
| parakeet | npu | NexaAI/parakeet-tdt-0.6b-v3-npu-mobile |
| liquid-v2 | npu | NexaAI/LFM2-1.2B-npu-mobile |
| jina-rerank | npu | NexaAI/jina-v2-rerank-npu-mobile |
| paddleocr | npu | NexaAI/paddleocr-npu-mobile |

For GGUF format models, use `pluginId: "cpu_gpu"` and no model name is required.

## Advanced Usage

### Generation Configuration

```dart
final config = GenerationConfig(
  maxTokens: 2048,
  stopWords: ['<|end|>', '<|stop|>'],
  samplerConfig: SamplerConfig(
    temperature: 0.7,
    topK: 40,
    topP: 0.95,
    repeatPenalty: 1.1,
  ),
);
```

### Conversation Management

```dart
// LLM conversation
final chatList = <ChatMessage>[];
chatList.add(ChatMessage('user', 'Hello!'));

// Apply template and generate
final template = await llm.applyChatTemplate(chatList);
// ... generate response

// Add assistant response to history
chatList.add(ChatMessage('assistant', responseText));

// Continue conversation
chatList.add(ChatMessage('user', 'Tell me more'));

// Reset conversation
await llm.reset();
```

### Stop Generation

```dart
// Stop streaming generation
await llm.stopStream();
// or
await vlm.stopStream();
```

## Error Handling

```dart
try {
  final llm = await LlmWrapper.create(input);
  // Use the model
} catch (e) {
  print('Failed to create model: $e');
}
```

## Performance Tips

1. **Choose the right hardware**: Use NPU for best performance on supported devices
2. **Manage model lifecycle**: Always call `destroy()` when done
3. **Stream for responsiveness**: Use streaming for better user experience
4. **Batch embeddings**: Generate embeddings for multiple texts at once
5. **Reuse models**: Keep models loaded if you need to use them multiple times

## Limitations

- Currently supports Android only
- NPU support requires compatible Qualcomm hardware
- Model files must be downloaded separately and stored locally
- File paths must be absolute paths accessible by the app

## Example App

Check out the `/example` folder for a complete demo app showing all features.

## Support

For issues and questions:
- GitHub Issues: Report a bug or request a feature
- Nexa AI Documentation: https://docs.nexaai.com/

## Acknowledgments

Built on top of the Nexa AI Android SDK (v0.0.10). Special thanks to the Nexa AI team for their excellent work on the underlying SDK.
