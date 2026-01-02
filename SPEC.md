# Jan Fork - Self-Contained Windows Desktop App with Embedded RAG

## Project Overview

This project is a fork of the Jan AI application, customized to create a fully self-contained Windows desktop application. The application will provide a ChatGPT-like interface that connects to OpenAI-compatible API providers while featuring local document chat capabilities through Retrieval-Augmented Generation (RAG).

**Key Differentiator**: Unlike the original Jan app, this fork removes all local LLM inference capabilities and instead focuses on being a lightweight, offline-capable RAG application that works with remote OpenAI-compatible APIs.

---

## Project Goals

### Primary Objectives

1. **Complete Self-Containment**: Create a single Windows installer that includes all dependencies, requiring no internet connection for installation or initial setup
2. **Embedded RAG Capabilities**: Bundle a small embedding model and local vector database for document chat functionality
3. **OpenAI-Compatible Only**: Simplify the codebase by supporting only OpenAI-compatible API providers
4. **Offline Operation**: Ensure the embedding and vector search functionality works completely offline
5. **Production-Ready**: Must work in restricted network environments where downloading dependencies post-installation is not permitted

### Secondary Objectives

1. Simplify the Jan codebase by removing unnecessary features
2. Improve application startup time (no model downloads)
3. Reduce installer size compared to full Jan distribution
4. Maintain professional UI/UX quality
5. Enable document-based context for LLM conversations

---

## Technical Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────┐
│         Jan.exe (Tauri Windows App)             │
├─────────────────────────────────────────────────┤
│  Frontend (React)                               │
│  - Chat UI                                      │
│  - Document Upload                              │
│  - Settings                                     │
├─────────────────────────────────────────────────┤
│  Rust Backend (Tauri)                           │
│  ├─ OpenAI API Client                          │
│  ├─ Document Processor                         │
│  ├─ Embedding Engine (ONNX Runtime)            │
│  └─ Vector Store (LanceDB)                     │
├─────────────────────────────────────────────────┤
│  Bundled Assets (in resources/)                 │
│  ├─ model.onnx (23MB)                          │
│  ├─ tokenizer.json (500KB)                     │
│  ├─ onnxruntime.dll (15MB)                     │
│  └─ config.json                                 │
└─────────────────────────────────────────────────┘
```

### Technology Stack

**Frontend**:
- React (from Jan's existing web-app)
- TypeScript
- Tailwind CSS
- Simplified UI removing model hub and hardware settings

**Backend**:
- Tauri v1.5+ (Rust-based desktop framework)
- Rust for all backend logic
- ONNX Runtime for embedding generation
- LanceDB for vector storage

**Bundled Components**:
- ONNX embedding model (all-MiniLM-L6-v2)
- Tokenizer files
- ONNX Runtime DLL (Windows)
- Vector database (embedded, no separate server)

---

## Embedding Model Selection

### Chosen Model: all-MiniLM-L6-v2 (ONNX)

**Specifications**:
- Size: ~23MB
- Embedding Dimensions: 384
- Format: ONNX (Open Neural Network Exchange)
- Speed: Very fast (~50ms per chunk on modern CPUs)
- Quality: Good balance for most document retrieval use cases

**Rationale**:
- Small enough to bundle in installer
- Fast inference on CPU (no GPU required)
- Well-tested and widely used
- ONNX format provides cross-platform compatibility
- No Python dependencies required

**Alternative Options Considered**:
1. **all-MiniLM-L12-v2**: Larger (45MB), slightly better quality, but slower
2. **BGE-small-en-v1.5**: Better quality (33MB), but English-only
3. **EmbeddingGemma-300m**: State-of-the-art but too large (300MB+)

---

## Vector Database Selection

### Chosen Database: LanceDB

**Specifications**:
- Type: Embedded vector database
- Language: Rust/JavaScript native
- Storage: Columnar format on disk
- Size: ~10MB overhead
- Performance: <100ms search for 1000+ documents

**Key Features**:
- No separate server process needed
- Pure Rust implementation (perfect for Tauri)
- Supports vector similarity search natively
- ACID compliance
- Efficient disk usage
- Incremental updates

**Alternative Considered**:
- **DuckDB with VSS extension**: SQL interface, slightly larger (~15MB), more complex setup

---

## Application Features

### Core Features (Must Have)

1. **Chat Interface**
   - Clean, modern UI similar to ChatGPT
   - Message history
   - Conversation management
   - Markdown rendering
   - Code syntax highlighting

2. **OpenAI-Compatible API Integration**
   - Support for any OpenAI API-compatible endpoint
   - Configurable base URL (e.g., `http://localhost:11434/v1` for Ollama)
   - API key management
   - Model selection from provider
   - Streaming responses
   - Error handling and retry logic

3. **Document Upload & Management**
   - Drag-and-drop file upload
   - Support for PDF, DOCX, TXT files
   - Document list with metadata
   - Delete/remove documents
   - Document preview
   - Batch upload support

4. **Local Embedding Generation**
   - Automatic text extraction from documents
   - Smart text chunking (1000 tokens, 200 token overlap)
   - Sentence-aware splitting
   - Progress indicators during processing
   - Background processing

5. **Vector Search & RAG**
   - Semantic search across uploaded documents
   - Context retrieval for LLM queries
   - Relevance scoring
   - Configurable number of retrieved chunks (default: 3-5)
   - Context window management

6. **Settings & Configuration**
   - API provider configuration (base URL, API key)
   - Model selection
   - RAG parameters (chunk size, overlap, retrieval count)
   - Context window settings
   - Export/import settings

### Nice-to-Have Features (Future)

1. Multi-document context (upload multiple files per conversation)
2. Document preprocessing (OCR for images, table extraction)
3. Prompt templates for RAG queries
4. Conversation export/import
5. API usage tracking and token counting
6. Conversation search and filtering
7. Document tags and categories

---

## Changes from Original Jan

### Features to Remove

1. **Local LLM Infrastructure**
   - llama.cpp integration
   - Model downloads from HuggingFace
   - Model management UI
   - GPU/CUDA/Metal support
   - Hardware acceleration settings
   - Model quantization

2. **Multiple Inference Backends**
   - inference-nitro-extension
   - inference-cortex-extension
   - All non-OpenAI provider extensions
   - Local model configuration

3. **Model Hub UI**
   - Model discovery
   - Model download progress
   - Model catalog
   - Community models

4. **Hardware Management**
   - GPU detection
   - VRAM monitoring
   - CPU thread configuration
   - Hardware compatibility checks

### Features to Keep & Modify

1. **Extension System** (simplified)
   - Keep: OpenAI provider extension
   - Keep: Assistant/conversation management
   - Modify: Remove extension marketplace
   - Modify: Hardcode core extensions

2. **Chat Interface**
   - Keep: Core chat UI
   - Keep: Message rendering
   - Modify: Add document context indicators
   - Modify: Show retrieved chunks on hover/click

3. **Settings UI**
   - Keep: Basic configuration
   - Modify: Focus on API and RAG settings only
   - Remove: Model and hardware settings

4. **Data Storage**
   - Keep: Conversation persistence
   - Keep: Settings storage
   - Add: Document metadata storage
   - Add: Vector database storage

---

## Directory Structure

```
jan-fork/
├── src-tauri/                          # Tauri/Rust backend
│   ├── src/
│   │   ├── main.rs                     # Application entry point
│   │   ├── api/
│   │   │   ├── mod.rs
│   │   │   └── openai_client.rs        # OpenAI-compatible API client
│   │   ├── embeddings/
│   │   │   ├── mod.rs
│   │   │   ├── onnx_engine.rs          # ONNX Runtime integration
│   │   │   ├── model_loader.rs         # Load bundled model files
│   │   │   └── tokenizer.rs            # Text tokenization
│   │   ├── vector_store/
│   │   │   ├── mod.rs
│   │   │   ├── lancedb.rs              # LanceDB integration
│   │   │   ├── search.rs               # Semantic search logic
│   │   │   └── storage.rs              # Document persistence
│   │   ├── documents/
│   │   │   ├── mod.rs
│   │   │   ├── processor.rs            # Document processing orchestrator
│   │   │   ├── extractors/
│   │   │   │   ├── pdf.rs              # PDF text extraction
│   │   │   │   ├── docx.rs             # DOCX text extraction
│   │   │   │   └── txt.rs              # Plain text handling
│   │   │   ├── chunker.rs              # Intelligent text chunking
│   │   │   └── indexer.rs              # Document indexing pipeline
│   │   ├── commands.rs                 # Tauri IPC commands
│   │   └── utils.rs                    # Shared utilities
│   ├── resources/                      # BUNDLED ASSETS
│   │   ├── models/
│   │   │   ├── model.onnx              # all-MiniLM-L6-v2 ONNX model
│   │   │   ├── tokenizer.json          # Tokenizer configuration
│   │   │   └── config.json             # Model metadata
│   │   └── onnxruntime.dll             # Windows ONNX Runtime library
│   ├── icons/                          # Application icons
│   ├── Cargo.toml                      # Rust dependencies
│   └── tauri.conf.json                 # Tauri configuration
│
├── web-app/                            # Frontend (React)
│   ├── src/
│   │   ├── containers/
│   │   │   ├── Chat/                   # Chat interface
│   │   │   ├── Documents/              # Document management UI
│   │   │   └── Settings/               # Settings panel
│   │   ├── components/                 # Reusable UI components
│   │   ├── hooks/                      # React hooks
│   │   ├── services/                   # API services
│   │   └── types/                      # TypeScript types
│   ├── public/
│   └── package.json
│
├── extensions/                         # Simplified extensions
│   └── openai-provider/                # OpenAI-compatible provider only
│       ├── src/
│       └── package.json
│
├── core/                               # Shared core (minimal)
│   ├── src/
│   │   └── types/                      # Shared TypeScript types
│   └── package.json
│
├── scripts/                            # Build and setup scripts
│   ├── prepare-models.sh               # Download bundled assets
│   ├── build-windows.sh                # Windows build script
│   └── package-installer.sh            # Create installer
│
├── docs/                               # Documentation
│   ├── ARCHITECTURE.md
│   ├── DEVELOPMENT.md
│   └── USER_GUIDE.md
│
├── package.json                        # Root package configuration
├── README.md
├── LICENSE
└── .gitignore
```

---

## Rust Dependencies (Cargo.toml)

### Core Dependencies

```toml
[dependencies]
# Tauri framework
tauri = { version = "1.5", features = ["shell-open"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# ONNX Runtime for embeddings
ort = "1.16"                           # ONNX Runtime bindings
tokenizers = "0.15"                     # HuggingFace tokenizers

# Vector database
lancedb = "0.4"                         # Embedded vector DB

# Document processing
pdf-extract = "0.7"                     # PDF text extraction
docx = "0.3"                           # DOCX parsing
lopdf = "0.31"                         # Low-level PDF operations

# HTTP client for OpenAI API
reqwest = { version = "0.11", features = ["json", "stream"] }
tokio = { version = "1", features = ["full"] }
futures = "0.3"

# Text processing
unicode-segmentation = "1.10"          # Unicode-aware text splitting
regex = "1.10"                         # Pattern matching

# Utilities
anyhow = "1.0"                         # Error handling
thiserror = "1.0"                      # Error types
tracing = "0.1"                        # Logging
tracing-subscriber = "0.3"             # Log formatting
```

### Key Library Justifications

- **ort**: Official ONNX Runtime Rust bindings, well-maintained
- **lancedb**: Modern embedded vector database, perfect for Tauri
- **tokenizers**: Official HuggingFace tokenizer library (Rust native)
- **pdf-extract**: Pure Rust PDF text extraction
- **reqwest**: Most popular Rust HTTP client, async support

---

## Tauri Configuration

### Bundle Configuration (tauri.conf.json)

**Key Settings**:
- Target: Windows only (msi, nsis)
- Bundle size: ~120-150MB
- Resources: Include models directory and DLLs
- Installer: NSIS (modern, customizable)

**Resource Bundling**:
```json
"resources": [
  "resources/models/*",
  "resources/*.dll"
]
```

**Build Targets**:
- Primary: NSIS installer (.exe)
- Alternative: MSI installer (.msi)

---

## Data Storage Strategy

### Application Data Location

**Windows AppData Structure**:
```
C:\Users\{Username}\AppData\Roaming\YourAppName\
├── config.json                         # User settings
│   ├── api_provider                    # OpenAI endpoint config
│   ├── api_key                         # Encrypted API key
│   ├── model_selection                 # Selected model
│   └── rag_settings                    # RAG parameters
│
├── vectors.lance/                      # LanceDB vector database
│   ├── data/                           # Vector data files
│   ├── index/                          # Search indices
│   └── metadata.json                   # Database metadata
│
├── documents/                          # Original uploaded files
│   ├── {document-id-1}.pdf
│   ├── {document-id-2}.docx
│   └── metadata.json                   # Document metadata
│
├── conversations/                      # Chat history
│   ├── {conversation-id-1}.json
│   └── {conversation-id-2}.json
│
└── logs/                               # Application logs
    └── app.log
```

### Data Persistence

1. **Configuration**: JSON file with encrypted sensitive data
2. **Vector Store**: LanceDB handles persistence automatically
3. **Documents**: Original files stored, metadata in JSON
4. **Conversations**: JSON files per conversation
5. **Logs**: Rotating log files for debugging

---

## Build & Deployment Process

### Pre-Build Steps

1. **Download Bundled Assets**
   ```bash
   # Script: scripts/prepare-models.sh
   # Downloads:
   # - all-MiniLM-L6-v2 ONNX model (~23MB)
   # - Tokenizer configuration (~500KB)
   # - ONNX Runtime DLL for Windows (~15MB)
   ```

2. **Prepare Development Environment**
   - Install Rust toolchain (rustup)
   - Install Node.js 20+
   - Install Yarn package manager
   - Install Windows SDK (for Tauri)

### Build Process

1. **Prepare models**: Download and place bundled assets
2. **Build frontend**: Compile React app to static files
3. **Build Rust backend**: Compile Tauri application
4. **Bundle resources**: Include models and DLLs
5. **Create installer**: Generate NSIS/MSI installer

**Build Commands**:
```bash
# Full build pipeline
yarn prepare-models        # Download assets
yarn build:web             # Build React frontend
cargo tauri build          # Build and package app
```

### Installer Specifications

**NSIS Installer**:
- Single-file executable installer
- Install location: `C:\Program Files\YourAppName\`
- Desktop shortcut creation
- Start menu integration
- Uninstaller included
- Silent install support: `/S` flag

**Size Breakdown**:
- Frontend (web-app): ~5-10MB
- Rust binary: ~15-20MB
- ONNX model: ~23MB
- ONNX Runtime: ~15MB
- Tokenizer: ~500KB
- Other assets: ~5MB
- **Total: ~120-150MB**

---

## Implementation Phases

### Phase 1: Core Infrastructure (Week 1-2)
**Goal**: Establish base application with embedding capabilities

**Tasks**:
- [ ] Set up Tauri project structure
- [ ] Configure Cargo dependencies
- [ ] Implement ONNX Runtime integration
- [ ] Load and test bundled embedding model
- [ ] Implement tokenizer
- [ ] Create basic embedding pipeline
- [ ] Unit tests for embedding generation

**Deliverable**: Working embedding engine that can generate vectors from text

---

### Phase 2: Vector Database Integration (Week 2-3)
**Goal**: Implement vector storage and semantic search

**Tasks**:
- [ ] Integrate LanceDB
- [ ] Design database schema
- [ ] Implement document storage
- [ ] Implement vector search
- [ ] Add result ranking
- [ ] Optimize search performance
- [ ] Unit tests for vector operations

**Deliverable**: Functional vector database with search capabilities

---

### Phase 3: Document Processing (Week 3-4)
**Goal**: Extract and process documents for indexing

**Tasks**:
- [ ] Implement PDF text extraction
- [ ] Implement DOCX text extraction
- [ ] Implement TXT file handling
- [ ] Create text chunking algorithm
- [ ] Build document indexing pipeline
- [ ] Add metadata management
- [ ] Handle processing errors gracefully
- [ ] Integration tests

**Deliverable**: Complete document processing pipeline

---

### Phase 4: OpenAI API Integration (Week 5-6)
**Goal**: Connect to OpenAI-compatible providers with RAG context

**Tasks**:
- [ ] Implement OpenAI API client
- [ ] Add streaming response support
- [ ] Integrate vector search with chat
- [ ] Build context injection logic
- [ ] Implement error handling and retries
- [ ] Add request/response logging
- [ ] Test with multiple providers (OpenAI, Ollama, etc.)

**Deliverable**: Working chat with RAG context retrieval

---

### Phase 5: Frontend Development (Week 7-8)
**Goal**: Build user interface

**Tasks**:
- [ ] Simplify Jan's UI (remove unnecessary features)
- [ ] Implement chat interface
- [ ] Create document upload UI
- [ ] Build document management panel
- [ ] Design settings panel
- [ ] Add context visualization (show retrieved chunks)
- [ ] Implement conversation history
- [ ] Add loading states and error messages
- [ ] Responsive design testing

**Deliverable**: Complete, polished user interface

---

### Phase 6: Testing & Optimization (Week 9)
**Goal**: Ensure quality and performance

**Tasks**:
- [ ] End-to-end testing
- [ ] Performance profiling
- [ ] Memory optimization
- [ ] Search latency optimization
- [ ] Error handling review
- [ ] Edge case testing
- [ ] User acceptance testing

**Deliverable**: Stable, performant application

---

### Phase 7: Packaging & Deployment (Week 10)
**Goal**: Create distributable installer

**Tasks**:
- [ ] Configure Tauri bundling
- [ ] Create NSIS installer
- [ ] Add application icons
- [ ] Write installation instructions
- [ ] Create user documentation
- [ ] Test installer on clean Windows machines
- [ ] Create release notes
- [ ] Final QA pass

**Deliverable**: Production-ready installer

---

## Performance Targets

### Embedding Generation
- **Target**: <50ms per document chunk (1000 tokens)
- **Batch Processing**: <5 seconds for 10-page PDF
- **Memory Usage**: <500MB during processing

### Vector Search
- **Target**: <100ms for semantic search (1000+ documents)
- **Relevance**: Top-3 results should be >0.7 cosine similarity for relevant queries
- **Scalability**: Support up to 10,000 document chunks efficiently

### Application Startup
- **Target**: <2 seconds from launch to ready
- **No Downloads**: Instant availability after installation
- **Memory Footprint**: <300MB idle

### Chat Response
- **Streaming Latency**: First token within 500ms
- **Context Injection**: <200ms to retrieve and format context
- **Memory**: <100MB overhead for conversation history

---

## Security Considerations

### API Key Storage
- Store encrypted in application data directory
- Use Windows DPAPI for encryption
- Never log API keys
- Clear from memory after use

### Local Data
- All embeddings and documents stored locally
- No data sent to external services (except chosen API provider)
- User controls all data retention
- Clear uninstall process

### Network Security
- HTTPS only for API calls
- Validate SSL certificates
- Configurable proxy support
- No telemetry or analytics

---

## Testing Strategy

### Unit Tests
- Embedding generation accuracy
- Vector similarity calculations
- Text chunking logic
- Document extraction
- API client error handling

### Integration Tests
- End-to-end document processing
- RAG pipeline (upload → embed → search → chat)
- API provider compatibility
- Database operations

### User Acceptance Tests
- Installation on clean Windows machines
- Offline functionality
- Document upload and processing
- Chat with context retrieval
- Settings persistence

### Performance Tests
- Embedding generation benchmarks
- Search latency under load
- Memory usage profiling
- Large document handling (100+ pages)

---

## Documentation Requirements

### User Documentation
1. **Installation Guide**
   - System requirements
   - Installation steps
   - First-time setup

2. **User Manual**
   - Uploading documents
   - Configuring API providers
   - Using chat with RAG
   - Managing documents
   - Settings reference

3. **Troubleshooting Guide**
   - Common issues
   - Error messages
   - Debug logs location
   - Support contact

### Developer Documentation
1. **Architecture Overview**
   - System design
   - Component interactions
   - Data flow diagrams

2. **Development Guide**
   - Setting up dev environment
   - Building from source
   - Running tests
   - Debugging tips

3. **API Reference**
   - Tauri commands
   - Rust public APIs
   - Frontend services

---

## Success Criteria

### Functionality
- ✅ Successfully installs on Windows 10/11 without internet
- ✅ Generates embeddings for uploaded documents offline
- ✅ Performs semantic search across document corpus
- ✅ Connects to and communicates with OpenAI-compatible APIs
- ✅ Injects relevant context into chat queries
- ✅ Persists conversations and settings

### Performance
- ✅ Application starts in <2 seconds
- ✅ Embedding generation <50ms per chunk
- ✅ Vector search <100ms for 1000+ documents
- ✅ Memory usage <500MB during operation

### Quality
- ✅ No crashes during normal operation
- ✅ Graceful error handling for all user actions
- ✅ Professional, intuitive UI
- ✅ Clear error messages and guidance

### Deployment
- ✅ Single installer file <200MB
- ✅ Clean uninstall with no leftovers
- ✅ Works in restricted network environments
- ✅ No external dependencies required

---

## Known Constraints & Limitations

### Technical Constraints
1. **Windows Only**: Initial version targets Windows 10/11 only
2. **No GPU Acceleration**: Embedding model runs on CPU only
3. **Document Size**: Practical limit of ~1000 pages per document
4. **File Formats**: Limited to PDF, DOCX, TXT initially
5. **Embedding Model**: Fixed model (no user choice in v1.0)

### Network Constraints
1. **Offline Embedding**: Works offline, but requires API for chat
2. **No Cloud Sync**: No cross-device synchronization
3. **Local Storage Only**: All data on single machine

### Functional Limitations
1. **No OCR**: Cannot extract text from image-based PDFs
2. **No Multi-Language**: Optimized for English documents
3. **No Collaboration**: Single-user application
4. **Limited File Preview**: Basic text preview only

---

## Future Enhancement Ideas

### Short-Term (v1.1 - v1.3)
- Support for additional file formats (Markdown, HTML, CSV)
- OCR for scanned PDFs
- Advanced chunking strategies
- Export conversations to PDF/Word
- Import conversations from other apps

### Medium-Term (v2.0)
- Multi-language embedding models
- Multiple embedding models support
- Cloud sync (optional)
- Mobile companion app
- API usage analytics dashboard

### Long-Term (v3.0+)
- Collaborative features
- Plugin system for custom processors
- Advanced RAG techniques (re-ranking, query expansion)
- Integration with enterprise systems
- Cross-platform support (macOS, Linux)

---

## Risk Assessment

### High Priority Risks

**Risk**: ONNX Runtime compatibility issues on older Windows versions
- **Mitigation**: Test on Windows 10 (1809+) and Windows 11
- **Fallback**: Provide clear system requirements

**Risk**: Embedding quality insufficient for document retrieval
- **Mitigation**: Test with diverse document corpus during development
- **Fallback**: Allow model swapping in future version

**Risk**: Installer size exceeds user expectations
- **Mitigation**: Clearly communicate size (~150MB) before download
- **Fallback**: Consider compression techniques

### Medium Priority Risks

**Risk**: Vector database performance degrades with large document sets
- **Mitigation**: Optimize indexing, implement pagination
- **Fallback**: Document limits with clear messaging

**Risk**: PDF extraction fails on complex/scanned documents
- **Mitigation**: Test with diverse PDF corpus
- **Fallback**: Graceful error messages, suggest alternatives

**Risk**: UI complexity from Jan carries over
- **Mitigation**: Aggressive simplification during Phase 5
- **Fallback**: User testing and iterative improvements

---

## Competitive Analysis

### Similar Applications

**Cursor / GitHub Copilot Workspace**
- Pros: Excellent IDE integration
- Cons: Requires internet, no local RAG

**Quivr**
- Pros: Open-source, RAG-focused
- Cons: Web-based, requires Python backend

**PrivateGPT**
- Pros: Privacy-focused, local LLMs
- Cons: Complex setup, large downloads

**Our Differentiators**:
1. ✅ Single-file installer with everything bundled
2. ✅ Works in air-gapped environments
3. ✅ Fast startup (no model downloads)
4. ✅ Lightweight (150MB vs 5GB+)
5. ✅ Professional desktop app experience

---

## Project Timeline Summary

| Phase | Duration | Key Deliverable |
|-------|----------|----------------|
| 1 - Core Infrastructure | 2 weeks | Working embedding engine |
| 2 - Vector Database | 1 week | Functional search |
| 3 - Document Processing | 2 weeks | Complete processing pipeline |
| 4 - OpenAI Integration | 2 weeks | Working RAG chat |
| 5 - Frontend Development | 2 weeks | Complete UI |
| 6 - Testing & Optimization | 1 week | Stable application |
| 7 - Packaging & Deployment | 1 week | Production installer |
| **Total** | **~10 weeks** | **Released v1.0** |

---

## Resource Requirements

### Development Team (Minimum)
- 1 Rust/Systems Developer (full-time)
- 1 Frontend Developer (part-time, weeks 7-8)
- 1 QA Engineer (part-time, weeks 9-10)

### Development Environment
- Windows 10/11 development machine
- 16GB RAM minimum (32GB recommended)
- SSD storage
- Visual Studio Code or similar IDE
- Git for version control

### Third-Party Services
- GitHub for repository hosting
- (Optional) CI/CD pipeline for automated builds
- (Optional) Code signing certificate for installer

---

## Open Questions

1. **Branding**: What is the final application name?
2. **Licensing**: Which open-source license to use? (Apache 2.0 like Jan?)
3. **Distribution**: Will there be a website, or GitHub-only releases?
4. **Support Model**: Community support only, or dedicated support?
5. **Versioning**: Semantic versioning strategy?
6. **Telemetry**: Any anonymous usage analytics? (Recommend: No)
7. **Updates**: Auto-update mechanism, or manual download?
8. **Code Signing**: Will the installer be code-signed? (Recommended for Windows)

---

## References & Resources

### Jan Project
- Repository: https://github.com/janhq/jan
- License: Apache 2.0
- Documentation: https://jan.ai/docs

### Embedding Models
- all-MiniLM-L6-v2: https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2
- ONNX Models: https://huggingface.co/onnx-models

### Technologies
- Tauri: https://tauri.app
- ONNX Runtime: https://onnxruntime.ai
- LanceDB: https://lancedb.com
- HuggingFace Tokenizers: https://github.com/huggingface/tokenizers

### RAG Resources
- LangChain Chunking Strategies: https://js.langchain.com/docs/modules/indexes/text_splitters
- RAG Best Practices: https://www.pinecone.io/learn/retrieval-augmented-generation/

---

## Conclusion

This project aims to create a focused, production-ready Windows desktop application that combines the best aspects of the Jan AI project with specialized RAG capabilities. By bundling all dependencies and focusing on a single use case (OpenAI-compatible APIs + local document chat), we can deliver a lightweight, fast, and reliable tool for users in restricted network environments.

The 10-week timeline is ambitious but achievable with dedicated focus. The key to success will be aggressive scope management—keeping only essential features and deferring nice-to-haves to future versions.

**Next Steps**:
1. Review and approve this specification
2. Set up development environment
3. Begin Phase 1: Core Infrastructure
4. Establish regular progress checkpoints (weekly)

---

**Document Version**: 1.0  
**Last Updated**: 2025-01-02  
**Status**: Draft for Review