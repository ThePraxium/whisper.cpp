# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

whisper.cpp is a high-performance C/C++ implementation of OpenAI's Whisper automatic speech recognition (ASR) model. It's designed for efficient inference without dependencies and supports multiple platforms and hardware accelerators.

## Key Architecture Components

### Core Library Structure
- **`include/whisper.h`** - Main C API header with complete interface definitions
- **`src/whisper.cpp`** - Core implementation of the Whisper model inference engine
- **`src/whisper-arch.h`** - Architecture-specific optimizations and configurations
- **`ggml/`** - Machine learning library submodule providing tensor operations and hardware acceleration

### Hardware Acceleration Backends
- **CoreML** (`src/coreml/`) - Apple Neural Engine acceleration for macOS/iOS
- **OpenVINO** (`src/openvino/`) - Intel optimization toolkit integration
- **CUDA/MUSA** (via ggml) - NVIDIA and Moore Threads GPU support
- **Metal** (via ggml) - Apple GPU acceleration
- **Vulkan** (via ggml) - Cross-platform GPU compute

### Language Bindings
- **Go** (`bindings/go/`) - Complete Go wrapper with examples
- **JavaScript/WebAssembly** (`bindings/javascript/`) - Browser and Node.js support
- **Java** (`bindings/java/`) - JNI bindings for Java applications
- **Ruby** (`bindings/ruby/`) - Native Ruby extension

## Build System Commands

### Quick Build (CMake)
```bash
# Standard build
cmake -B build
cmake --build build --config Release

# With CUDA support
cmake -B build -DGGML_CUDA=1
cmake --build build --config Release

# With Core ML support (macOS)
cmake -B build -DWHISPER_COREML=1
cmake --build build --config Release
```

### Using Makefile (Convenience)
```bash
# Build project
make build

# Download and test specific model
make base.en    # Downloads model and runs on sample audio
make small      # Downloads small model
make large-v3   # Downloads large v3 model

# Download audio samples for testing
make samples
```

### Model Management
```bash
# Download specific model
./models/download-ggml-model.sh base.en

# Convert custom models
python models/convert-pt-to-ggml.py

# Quantize existing models
./build/bin/quantize models/ggml-base.en.bin models/ggml-base.en-q5_0.bin q5_0
```

## Testing and Development

### Running Tests
```bash
# Build and run tests
cmake -B build -DWHISPER_BUILD_TESTS=ON
cmake --build build --config Release
cd build && ctest
```

### Example Applications
- **`./build/bin/whisper-cli`** - Main command-line transcription tool
- **`./build/bin/whisper-stream`** - Real-time audio transcription
- **`./build/bin/whisper-server`** - HTTP API server
- **`./build/bin/whisper-bench`** - Performance benchmarking

### Common Usage Patterns
```bash
# Basic transcription
./build/bin/whisper-cli -f samples/jfk.wav -m models/ggml-base.en.bin

# Real-time streaming (requires SDL2)
./build/bin/whisper-stream -m models/ggml-base.en.bin -t 8 --step 500

# Server mode with OpenAI-compatible API
./build/bin/whisper-server -m models/ggml-base.en.bin --host 0.0.0.0 --port 8080
```

## Code Organization Patterns

### Model Loading and Context Management
The library follows a context-based pattern where `whisper_context` manages model state and `whisper_state` handles inference state. Always initialize contexts with appropriate parameters and free resources properly.

### Audio Processing Pipeline
Audio input must be 16kHz mono PCM format. The library handles mel-spectrogram conversion internally. Use `whisper_full()` for complete processing or fine-grained functions for custom pipelines.

### Memory Management
The library uses GGML's tensor allocation system. Memory usage depends on model size (see README.md for specifics). Quantized models significantly reduce memory requirements.

### Threading and Performance
The library is thread-safe for different contexts but not for concurrent access to the same context. Use `n_threads` parameter to control CPU parallelism.

## Platform-Specific Considerations

### macOS/iOS Development
- Core ML acceleration requires Xcode and coremltools
- Use `./build-xcframework.sh` for iOS framework generation
- Metal acceleration is automatically enabled when available

### Windows Development
- MSVC and MinGW both supported
- CUDA requires Visual Studio integration
- Use CMake with appropriate generators

### WebAssembly/Browser
- Examples in `examples/*.wasm/` show browser integration
- Requires Emscripten toolchain
- Memory constraints important for model selection

### Android Development
- Native examples in `examples/whisper.android/`
- JNI bindings available for Java integration
- Consider quantized models for mobile deployment

## Integration Guidelines

When integrating whisper.cpp into applications:

1. **Model Selection** - Choose appropriate model size based on accuracy/performance requirements
2. **Audio Preprocessing** - Ensure audio is converted to 16kHz mono before processing
3. **Memory Planning** - Account for model memory usage plus processing overhead
4. **Error Handling** - Check return codes from all whisper API calls
5. **Resource Cleanup** - Always call `whisper_free()` to prevent memory leaks

## Development Workflow

1. Use examples as starting points for new functionality
2. Test with multiple model sizes to ensure compatibility
3. Verify memory usage patterns with different audio lengths
4. Consider quantization for deployment scenarios
5. Test platform-specific acceleration when available