# Feature Removal Guide for Jan Fork

This document identifies all components that should be removed to transform Jan into a lightweight, RAG-focused Windows desktop app.

**Goal**: Remove all local LLM inference infrastructure and focus on OpenAI-compatible APIs + embedded RAG.

**Estimated Reduction**: 20,000-30,000 lines of code

---

## Quick Reference: What to Remove

### Extensions (2 complete removals)
- ❌ `extensions/llamacpp-extension/` - Local llama.cpp inference
- ❌ `extensions/download-extension/` - Model downloads from HuggingFace
- ⚠️  `extensions/rag-extension/` - Keep but remove llamacpp dependencies
- ✅ `extensions/vector-db-extension/` - Keep
- ✅ `extensions/conversational-extension/` - Keep
- ✅ `extensions/assistant-extension/` - Keep

### Tauri Plugins (2 complete removals)
- ❌ `src-tauri/plugins/tauri-plugin-llamacpp/` - Local inference backend
- ❌ `src-tauri/plugins/tauri-plugin-hardware/` - GPU/CPU monitoring
- ✅ `src-tauri/plugins/tauri-plugin-rag/` - Keep (document parsing)
- ✅ `src-tauri/plugins/tauri-plugin-vector-db/` - Keep

### Frontend Routes (3 complete removals)
- ❌ `web-app/src/routes/hub/` - Model hub UI
- ❌ `web-app/src/routes/settings/hardware.tsx` - Hardware settings
- ❌ `web-app/src/routes/system-monitor.tsx` - System monitoring

### Build Scripts (2 removals)
- ❌ `scripts/download-bin.mjs` - Downloads llama.cpp binaries
- ❌ `scripts/download-lib.mjs` - Downloads optional libraries

---

## Phase 1: Critical Infrastructure Removal

### 1.1 Remove Tauri Plugin: llamacpp

**Directory**: `src-tauri/plugins/tauri-plugin-llamacpp/`

**What it does**:
- Manages llama.cpp process lifecycle
- Model loading/unloading (35+ commands)
- Backend version checking and downloads
- GGUF metadata reading
- GPU/device detection
- Session management

**Key files**:
```
tauri-plugin-llamacpp/
├── src/
│   ├── backend.rs          # Backend installation & version management
│   ├── commands.rs         # 35+ Tauri command handlers
│   ├── process.rs          # Subprocess management
│   ├── device.rs           # GPU/hardware device selection
│   ├── gguf/               # GGUF file parsing
│   │   ├── loader.rs
│   │   ├── metadata.rs
│   │   └── planner.rs
│   └── session.rs          # Session management
├── Cargo.toml
└── permissions/
```

**Dependencies to remove from Cargo.toml**:
- `sysinfo = "0.34.2"`
- `tauri-plugin-hardware`
- `reqwest` (backend downloads)

**Impact**: Eliminates entire local LLM infrastructure (largest removal)

---

### 1.2 Remove Tauri Plugin: hardware

**Directory**: `src-tauri/plugins/tauri-plugin-hardware/`

**What it does**:
- GPU/VRAM monitoring (NVIDIA, AMD, Metal)
- CPU thread detection
- System resource tracking
- Vulkan/CUDA capability detection

**Key files**:
```
tauri-plugin-hardware/
├── src/
│   ├── gpu.rs              # GPU detection
│   ├── cpu.rs              # CPU info
│   ├── vendor/
│   │   ├── nvidia.rs       # NVIDIA-specific
│   │   ├── amd.rs          # AMD-specific
│   │   ├── vulkan.rs       # Vulkan detection
│   │   └── metal.rs        # Metal (macOS)
│   └── commands.rs         # Hardware monitoring commands
└── Cargo.toml
```

**Dependencies to remove**:
- `nvml-wrapper = "0.10.0"`
- `vulkano = "0.34"`
- `ash = "0.37"`
- `libloading = "0.8"`
- `sysinfo = "0.34.2"`

**Impact**: Removes hardware monitoring UI and settings

---

### 1.3 Remove Extensions

**Extension 1**: `extensions/llamacpp-extension/`

**What it does**:
- Orchestrates llamacpp plugin
- Handles model loading UI integration
- Manages inference sessions
- Device selection logic

**Remove entire directory**

---

**Extension 2**: `extensions/download-extension/`

**What it does**:
- HuggingFace model downloads
- Download progress tracking
- File validation
- Download queue management

**Remove entire directory**

---

### 1.4 Update Root Cargo.toml

**File**: `src-tauri/Cargo.toml`

**Remove**:
```toml
[dependencies]
tauri-plugin-llamacpp = { path = "plugins/tauri-plugin-llamacpp" }
tauri-plugin-hardware = { path = "plugins/tauri-plugin-hardware", optional = true }
```

**Modify features**:
```toml
[features]
# BEFORE:
desktop = ["tauri-plugin-hardware", "tauri-plugin-deep-link"]

# AFTER:
desktop = ["tauri-plugin-deep-link"]
```

---

### 1.5 Update Main Entry Point

**File**: `src-tauri/src/lib.rs` or `src-tauri/src/main.rs`

**Remove plugin registrations**:
```rust
// Remove these lines:
.plugin(tauri_plugin_llamacpp::init())
.plugin(tauri_plugin_hardware::init())
```

---

## Phase 2: Frontend Cleanup

### 2.1 Remove Model Hub Route

**Directory**: `web-app/src/routes/hub/`

**Files**:
```
hub/
├── index.tsx              # Model hub main page (1000+ lines)
├── $modelId.tsx           # Model detail page
└── __tests__/
    └── huggingface-conversion.test.ts
```

**What it does**:
- Model discovery and browsing
- HuggingFace integration
- Model download UI
- Model filtering/search

**Remove entire directory**

---

### 2.2 Remove Hardware Settings Route

**File**: `web-app/src/routes/settings/hardware.tsx`

**What it does**:
- GPU/VRAM monitoring UI
- Device selection for inference
- CPU thread configuration
- Hardware compatibility display

**Remove entire file**

**Related test**: `web-app/src/routes/settings/__tests__/hardware.test.tsx` (remove)

---

### 2.3 Remove System Monitor Route

**File**: `web-app/src/routes/system-monitor.tsx`

**What it does**:
- Real-time system resource monitoring
- GPU usage graphs
- Memory usage tracking

**Remove entire file**

---

### 2.4 Remove Download-Related Components

**Components to remove**:

1. `web-app/src/containers/DownloadManegement.tsx` - Download progress popover
2. `web-app/src/containers/DownloadButton.tsx` - Download action button
3. `web-app/src/containers/ModelDownloadAction.tsx` - Per-model download UI
4. ⚠️  `web-app/src/containers/ModelCombobox.tsx` - **KEEP & MODIFY** for remote model selection
5. ⚠️  `web-app/src/containers/ModelInfoHoverCard.tsx` - **KEEP & MODIFY** for API model info
6. ❌ `web-app/src/containers/ModelSetting.tsx` - REMOVE (local model config)
7. ❌ `web-app/src/containers/ModelSupportStatus.tsx` - REMOVE (local compatibility)
8. ❌ `web-app/src/containers/FavoriteModelAction.tsx` - REMOVE (or keep if useful for API models)

**Modifications needed**:
- `ModelCombobox.tsx`: Change to fetch models from API endpoint instead of local catalog
- `ModelInfoHoverCard.tsx`: Show API model metadata (context window, pricing if available)

---

### 2.5 Remove Hooks

**Hooks to remove**:

1. ❌ `web-app/src/hooks/useDownloadStore.ts` - Download state management
2. ❌ `web-app/src/hooks/useHardware.ts` - Hardware info hook
3. ❌ `web-app/src/hooks/useLlamacppDevices.ts` - Device selection hook
4. ❌ `web-app/src/hooks/useModelLoad.ts` - Model loading hook (for local models)
5. ⚠️  `web-app/src/hooks/useModelSources.ts` - **MODIFY** to fetch from API instead of local catalog
6. ⚠️  `web-app/src/hooks/useFavoriteModel.ts` - **KEEP** if useful for API models, else remove

**Related tests** (remove all):
- `web-app/src/hooks/__tests__/useHardware.test.ts`
- `web-app/src/hooks/__tests__/useLlamacppDevices.test.ts`
- `web-app/src/hooks/__tests__/useDownloadStore.test.ts`
- `web-app/src/hooks/__tests__/useModelSources.test.ts`

**Modifications needed**:
- `useModelSources.ts`: Change to call OpenAI-compatible `/v1/models` endpoint to get available models
- This hook should fetch remote models, not local model catalog

---

### 2.6 Remove Service Modules

**Service directories to remove**:

1. `web-app/src/services/hardware/` (entire directory)
   - `default.ts`
   - `tauri.ts`
   - `types.ts`

**Related tests**:
- `web-app/src/services/__tests__/hardware.test.ts`

---

### 2.7 Update Settings Menu

**File**: `web-app/src/containers/SettingsMenu.tsx`

**Remove navigation items**:
- Hardware settings link
- Hub link (if exists)
- Local API Server settings link

**Keep**:
- General settings
- API provider settings
- Appearance settings
- About

---

### 2.8 Update General Settings

**File**: `web-app/src/routes/settings/general.tsx`

**Remove sections**:
- HuggingFace token input
- Model download directory
- Model cache settings (for local models)
- GPU/hardware settings
- Local inference settings

**Keep & Add**:
- ✅ API provider configuration (endpoint URL, API key)
- ✅ Model selection (from API)
- ✅ UI preferences (theme, language)
- ✅ **Storage preferences** (document storage location, vector DB path)
- ✅ **RAG settings** (chunk size, overlap, retrieval count, similarity threshold)
- ✅ **Embedding settings** (if configurable - model is bundled but settings might be adjustable)

**New sections to add**:
- Document storage location picker
- Vector database location (default: AppData)
- RAG parameters panel:
  - Chunk size (default: 1000 tokens)
  - Chunk overlap (default: 200 tokens)
  - Number of chunks to retrieve (default: 3-5)
  - Similarity threshold (default: 0.7)

---

### 2.9 Remove Localization Files

**Pattern**: `web-app/src/locales/*/system-monitor.json`

**Files to remove** (14 total, one per language):
```
locales/en/system-monitor.json
locales/es/system-monitor.json
locales/fr/system-monitor.json
locales/ja/system-monitor.json
locales/ko/system-monitor.json
locales/ru/system-monitor.json
locales/vi/system-monitor.json
locales/zh-CN/system-monitor.json
locales/zh-TW/system-monitor.json
... (and others)
```

**Also update**:
- Remove hardware-related strings from `settings.json` in all languages
- Remove download-related strings from all locale files

---

## Phase 3: Backend Refactoring

### 3.1 Remove Downloads Module

**Directory**: `src-tauri/src/core/downloads/`

**Files**:
```
downloads/
├── commands.rs           # Download command handlers
├── models.rs             # Model download orchestration
├── helpers.rs            # Download utilities
└── tests.rs              # Tests
```

**What it does**:
- Manages HuggingFace downloads
- Model installation
- File validation
- Progress reporting

**Remove entire directory**

---

### 3.2 Update App Module

**File**: `src-tauri/src/core/app/models.rs`

**Remove**:
- Model-related data structures for local models
- GGUF model definitions
- Hardware capability structures

**Keep**:
- Basic app configuration structures
- API provider configurations

---

### 3.3 Update Command Registrations

**File**: `src-tauri/src/core/mod.rs` or similar

**Remove command imports and registrations for**:
- Download commands
- Model loading commands
- Hardware monitoring commands

---

## Phase 4: Build Scripts & Dependencies

### 4.1 Remove Build Scripts

**Files to remove**:

1. `scripts/download-bin.mjs` - Downloads llama.cpp binaries
2. `scripts/download-lib.mjs` - Downloads optional libraries (if exists)

---

### 4.2 Update Package.json Scripts

**File**: `package.json` (root)

**Remove scripts**:
```json
{
  "download:lib": "...",
  "download:bin": "...",
}
```

**Update scripts** (remove download:bin dependency):
```json
{
  "build:tauri:win32": "...",  // Remove && yarn download:bin
  "build:tauri:linux": "...",  // Remove && yarn download:bin
  "build:tauri:darwin": "..."  // Remove && yarn download:bin
}
```

---

### 4.3 Remove Extension Package Files

**Files to remove**:

1. `extensions/llamacpp-extension/package.json`
2. `extensions/download-extension/package.json`

---

### 4.4 Update Workspace Configuration

**File**: `package.json` (root)

**Update workspaces array** - remove:
```json
{
  "workspaces": [
    // Remove these:
    "extensions/llamacpp-extension",
    "extensions/download-extension"
  ]
}
```

---

## Phase 5: Configuration Updates

### 5.1 Update Tauri Configuration

**File**: `src-tauri/tauri.conf.json`

**Remove/Update**:
- Remove model resource bundling (keep only ONNX resources)
- Remove hardware monitoring permissions
- Remove model management capabilities

**Before**:
```json
{
  "resources": [
    "resources/llama.cpp/*",
    "resources/models/*"
  ]
}
```

**After**:
```json
{
  "resources": [
    "resources/models/*",      // ONNX model only
    "resources/*.dll"          // ONNX Runtime DLL
  ]
}
```

---

### 5.2 Update Capabilities

**Directory**: `src-tauri/capabilities/`

**Remove capabilities for**:
- Hardware monitoring (`get_system_info`, `get_system_usage`)
- Model management (`load_llama_model`, `unload_llama_model`, etc.)
- Download operations

**Keep capabilities for**:
- Document processing
- Vector database operations
- RAG operations
- OpenAI API calls

---

### 5.3 Remove Pre-install Directory

**Directory**: `pre-install/` (if exists)

**What it contains**:
- Pre-downloaded binaries
- Bundled models
- Extension packages

**Remove entire directory** (no longer needed)

---

## Phase 6: Documentation Updates

### 6.1 Update README.md

**File**: `README.md`

**Remove sections**:
- "Run offline LLMs"
- Hardware requirements for local inference
- GPU support information
- Model downloads instructions

**Add sections**:
- "OpenAI-compatible API required"
- RAG functionality explanation
- Windows-only notice
- Bundled ONNX model information

---

### 6.2 Update CONTRIBUTING.md

**File**: `CONTRIBUTING.md`

**Update**:
- Remove references to llamacpp plugin
- Remove hardware plugin development
- Add RAG development guidelines
- Add ONNX integration guidelines

---

### 6.3 Update Component-Specific Guides

**Files to update**:
- `web-app/CONTRIBUTING.md` - Remove model hub development
- `src-tauri/CONTRIBUTING.md` - Remove llamacpp integration
- `src-tauri/plugins/CONTRIBUTING.md` - Remove hardware/llamacpp examples

---

## Phase 7: Type Definitions & Interfaces

### 7.1 Update Core Types

**File**: `core/src/types/Extension.ts` or similar

**Remove types**:
- `InferenceEngine` interface
- `LocalModel` type
- `HardwareInfo` type
- `DownloadProgress` type

**Keep/Add types**:
- `OpenAIProvider` interface
- `RAGConfig` type
- `EmbeddingConfig` type
- `VectorSearchResult` type

---

### 7.2 Update Model Types

**File**: `core/src/types/Model.ts` or similar

**Before** (has local model support):
```typescript
interface Model {
  id: string
  provider: 'llamacpp' | 'openai' | 'anthropic'
  path?: string           // Local model path
  quantization?: string   // GGUF quantization
  gpuLayers?: number      // GPU offloading
}
```

**After** (OpenAI-compatible only):
```typescript
interface RemoteModel {
  id: string              // e.g., "gpt-4", "gpt-3.5-turbo", "llama-3-8b"
  name?: string           // Display name
  provider: 'openai'      // Always OpenAI-compatible
  contextWindow?: number  // Max context length (e.g., 8192)
  pricing?: {             // Optional pricing info
    input: number
    output: number
  }
}

interface APIProvider {
  name: string            // User-friendly name
  endpoint: string        // e.g., "https://api.openai.com/v1"
  apiKey: string          // Encrypted API key
  models: RemoteModel[]   // Available models from this provider
}

interface RAGConfig {
  chunkSize: number       // Default: 1000 tokens
  chunkOverlap: number    // Default: 200 tokens
  retrievalCount: number  // Default: 3-5 chunks
  similarityThreshold: number  // Default: 0.7
}

interface EmbeddingProgress {
  documentId: string
  totalChunks: number
  processedChunks: number
  status: 'processing' | 'completed' | 'failed'
  error?: string
}

interface VectorSearchResult {
  chunkId: string
  documentId: string
  content: string
  similarity: number      // Cosine similarity score
  metadata: {
    documentName: string
    chunkIndex: number
    pageNumber?: number
  }
}
```

---

## Verification Checklist

After completing all removals, verify:

- [ ] `yarn install` succeeds without errors
- [ ] `yarn build:core` succeeds
- [ ] `yarn build:extensions` only builds kept extensions
- [ ] `cargo build --manifest-path src-tauri/Cargo.toml` succeeds
- [ ] No references to `tauri-plugin-llamacpp` in codebase
- [ ] No references to `tauri-plugin-hardware` in codebase
- [ ] No references to `download-extension` in codebase
- [ ] No references to `llamacpp-extension` in codebase
- [ ] Hub route (/hub) removed from routing
- [ ] Hardware settings route removed
- [ ] System monitor route removed
- [ ] All TypeScript compilation succeeds
- [ ] All tests pass (yarn test)
- [ ] Dev server starts (`yarn dev`)

---

## Automated Search & Replace

Use these commands to find remaining references:

```bash
# Find llamacpp references
rg -i "llamacpp" --type ts --type tsx --type rust

# Find hardware plugin references
rg -i "plugin.*hardware" --type ts --type tsx --type rust

# Find download extension references
rg -i "download.*extension" --type ts --type tsx

# Find model hub references
rg -i "model.*hub|hub.*model" --type ts --type tsx

# Find GGUF references (should be none)
rg -i "gguf" --type ts --type tsx --type rust
```

---

## Estimated Impact

| Metric | Before | After | Reduction |
|--------|--------|-------|-----------|
| Extensions | 6 | 4 | -33% |
| Tauri Plugins | 4 | 2 | -50% |
| Frontend Routes | ~15 | ~12 | -20% |
| Rust LOC | ~15,000 | ~10,000 | -33% |
| TypeScript LOC | ~30,000 | ~22,000 | -27% |
| Dependencies | ~100 | ~80 | -20% |
| Build Time | ~5 min | ~3 min | -40% |
| Installer Size | ~500MB | ~150MB | -70% |

---

## Risk Assessment

### High Risk Removals (Test Thoroughly)
- Tauri plugin removals - May break IPC communication
- Extension removals - May break frontend initialization
- Type definition changes - May cause TypeScript errors throughout

### Medium Risk Removals
- Frontend component removals - May have unexpected dependencies
- Hook removals - May be used in unexpected places
- Service module removals - May have circular dependencies

### Low Risk Removals
- Documentation files
- Localization files
- Test files
- Build scripts

---

## Next Steps

1. **Create removal branch**: `git checkout -b feature/remove-local-inference`
2. **Start with Phase 1**: Remove Tauri plugins first
3. **Test after each phase**: Ensure builds still work
4. **Commit frequently**: One commit per major removal
5. **Update CLAUDE.md**: Keep it in sync with changes
6. **Update SPEC.md**: Mark implementation progress

---

## Architectural Decisions (RESOLVED)

### 1. Extension Architecture
**Decision**: Simplify extension system - OpenAI provider only, possibly move to core

**Rationale**: We will only ever connect to OpenAI-compatible APIs for LLM access. No need for multiple provider extensions.

**Action**:
- Remove all non-OpenAI provider extensions
- Consider moving OpenAI client directly into core or Rust backend
- Simplify extension loading system

---

### 2. Model Concept
**Decision**: KEEP model selection - for remote API models

**Rationale**: The OpenAI-compatible API will have multiple models (gpt-4, gpt-3.5-turbo, custom models, etc.). Users need to select which model to use for each chat.

**Action**:
- Keep model selection UI (dropdown/combobox)
- Remove local model management (file paths, quantization)
- Keep model metadata (name, provider, endpoint)
- Model list comes from API response (not local catalog)

---

### 3. Settings Storage
**Decision**: KEEP storage preferences and RAG settings

**Rationale**: Users will create their own RAG library. They need to:
- Configure document storage location
- Manage vector database settings
- Customize RAG parameters (chunk size, retrieval count)
- Set API provider and keys

**Action**:
- Remove: HuggingFace tokens, model cache settings, GPU settings
- Keep: API provider config, storage paths, RAG parameters, UI preferences

---

### 4. Event System
**Decision**: REMOVE download/model loading events

**Rationale**: No model downloads - embedding model is bundled with installer. No model loading - API is always available.

**Action**:
- Remove `DownloadEvent` namespace completely
- Remove model loading events
- Keep document processing events
- Keep chat events
- Add RAG-specific events (embedding progress, search results)

---

### 5. Type Definitions
**Decision**: Create fork-specific types, simplify existing

**Action**:
- Create `RemoteModel` type (for API models)
- Create `RAGConfig` type (chunk size, overlap, retrieval count)
- Create `EmbeddingProgress` type
- Remove `LocalModel`, `InferenceEngine`, `HardwareInfo` types

---

## Prioritized Action Plan (Based on Decisions)

### Week 1: Critical Infrastructure Removal

**Goal**: Remove local LLM infrastructure without breaking the build

**TDD Approach**: Test-Before-Remove

**Tasks**:
1. ✅ Create removal branch: `git checkout -b feature/remove-local-inference`

2. 🧪 **BEFORE REMOVAL - Run baseline tests**:
   ```bash
   # Record current test status
   yarn test > baseline-tests.log
   cargo test --workspace > baseline-rust-tests.log

   # Verify all tests pass before starting
   make test
   ```

3. ⚠️  Remove Tauri plugins (HIGH RISK - test thoroughly):
   - Delete `src-tauri/plugins/tauri-plugin-llamacpp/`
   - Delete `src-tauri/plugins/tauri-plugin-hardware/`
   - Update `src-tauri/Cargo.toml` to remove plugin dependencies
   - Update plugin registrations in `src-tauri/src/lib.rs`

4. 🧪 **AFTER PLUGIN REMOVAL - Fix broken tests**:
   ```bash
   # Identify broken tests
   cargo test --workspace 2>&1 | grep "FAILED"

   # Remove tests for removed plugins
   # Fix tests that depended on plugins
   # Ensure no orphaned test files remain
   ```

5. ⚠️  Remove extensions:
   - Delete `extensions/llamacpp-extension/`
   - Delete `extensions/download-extension/`
   - Update workspace in root `package.json`

6. 🧪 **AFTER EXTENSION REMOVAL - Fix broken tests**:
   ```bash
   # Remove tests for removed extensions
   rm -rf extensions/llamacpp-extension/__tests__
   rm -rf extensions/download-extension/__tests__

   # Fix frontend tests that imported removed extensions
   yarn test 2>&1 | grep "FAIL"
   ```

7. ✅ Verify builds and tests:
   ```bash
   # Build verification
   cargo build --manifest-path src-tauri/Cargo.toml
   yarn install
   yarn build:core
   yarn build:extensions

   # Test verification (MUST PASS before commit)
   yarn test
   cargo test --workspace
   make test
   ```

8. ✅ Coverage check:
   ```bash
   # Ensure coverage hasn't dropped significantly
   yarn test:coverage
   # Should maintain similar coverage % as baseline
   ```

9. ✅ Commit: `git commit -m "refactor: remove local LLM infrastructure (plugins & extensions)"`

**Estimated time**: 1-2 days
**Risk level**: HIGH (may break builds if dependencies remain)
**Testing requirement**: All remaining tests must pass before committing

---

### Week 2: Frontend Route & UI Cleanup

**Goal**: Remove UI for model hub, downloads, and hardware

**Tasks**:
1. Remove entire routes:
   - Delete `web-app/src/routes/hub/`
   - Delete `web-app/src/routes/settings/hardware.tsx`
   - Delete `web-app/src/routes/system-monitor.tsx`
2. Remove download components:
   - `DownloadManegement.tsx`, `DownloadButton.tsx`, `ModelDownloadAction.tsx`
   - `ModelSetting.tsx`, `ModelSupportStatus.tsx`
3. Update navigation:
   - Remove hub link from `SettingsMenu.tsx`
   - Remove hardware settings link
4. Remove hooks:
   - `useDownloadStore.ts`, `useHardware.ts`, `useLlamacppDevices.ts`, `useModelLoad.ts`
5. Remove services:
   - Delete `web-app/src/services/hardware/`
6. Verify frontend builds:
   - `yarn dev:web-app` (should start without errors)
   - Check for TypeScript errors: `yarn tsc --noEmit`
7. Commit: `git commit -m "refactor: remove model hub, downloads, and hardware UI"`

**Estimated time**: 2-3 days
**Risk level**: MEDIUM (TypeScript errors likely, but won't break runtime)

---

### Week 3: Modify Model Selection for Remote APIs

**Goal**: Update model selection to fetch from OpenAI-compatible API

**TDD Approach**: Test-First Development (Red-Green-Refactor)

**Tasks**:

1. 🔴 **RED - Write tests first** (define expected behavior):
   ```typescript
   // core/src/types/__tests__/RemoteModel.test.ts
   describe('RemoteModel', () => {
       it('should validate model structure', () => {
           const model: RemoteModel = {
               id: 'gpt-4',
               name: 'GPT-4',
               provider: 'openai',
               contextWindow: 8192
           };
           expect(validateRemoteModel(model)).toBe(true);
       });
   });

   // hooks/__tests__/useModelSources.test.ts
   describe('useModelSources', () => {
       it('should fetch models from API endpoint', async () => {
           // Mock API response
           global.fetch = vi.fn().mockResolvedValue({
               json: async () => ({ data: [{ id: 'gpt-4', object: 'model' }] })
           });

           const { result } = renderHook(() => useModelSources());
           await waitFor(() => expect(result.current.models).toHaveLength(1));
           expect(result.current.models[0].id).toBe('gpt-4');
       });

       it('should cache model list', async () => {
           const { result } = renderHook(() => useModelSources());
           await waitFor(() => expect(result.current.models.length).toBeGreaterThan(0));

           // Should not refetch if called again
           expect(global.fetch).toHaveBeenCalledTimes(1);
       });
   });

   // containers/__tests__/ModelCombobox.test.tsx
   describe('ModelCombobox', () => {
       it('should display remote models in dropdown', async () => {
           render(<ModelCombobox />);
           await waitFor(() => {
               expect(screen.getByText('gpt-4')).toBeInTheDocument();
               expect(screen.getByText('gpt-3.5-turbo')).toBeInTheDocument();
           });
       });
   });
   ```

2. 🟢 **GREEN - Implement minimal code**:
   - Create type definitions:
     ```typescript
     // core/src/types/RemoteModel.ts
     interface RemoteModel {
         id: string;
         name?: string;
         provider: 'openai';
         contextWindow?: number;
     }
     ```
   - Modify `useModelSources.ts`:
     ```typescript
     export function useModelSources() {
         const [models, setModels] = useState<RemoteModel[]>([]);

         useEffect(() => {
             fetchModelsFromAPI().then(setModels);
         }, []);

         return { models };
     }
     ```
   - Modify `ModelCombobox.tsx` to use remote models

3. 🔵 **REFACTOR - Improve while keeping tests green**:
   - Add error handling
   - Add loading states
   - Add retry logic
   - Optimize caching

4. 🧪 **Add integration tests**:
   ```typescript
   describe('Model Selection Integration', () => {
       it('should fetch, display, and select remote models', async () => {
           // Configure API endpoint
           await updateSettings({ apiEndpoint: 'http://localhost:11434/v1' });

           // Render model selector
           render(<ModelCombobox />);

           // Wait for models to load
           await waitFor(() => expect(screen.getByText('llama-3-8b')).toBeInTheDocument());

           // Select a model
           fireEvent.click(screen.getByText('llama-3-8b'));

           // Verify selection persists
           expect(getSelectedModel()).toBe('llama-3-8b');
       });
   });
   ```

5. ✅ **Verify all tests pass**:
   ```bash
   yarn test hooks/useModelSources
   yarn test containers/ModelCombobox
   yarn test:coverage  # Ensure 80%+ coverage
   ```

6. Update type definitions (remove old types):
   - Remove `LocalModel`, `InferenceEngine` types
   - Create `APIProvider` interface

7. Update settings UI:
   - Remove HuggingFace token from `settings/general.tsx`
   - Keep API provider config (endpoint + key)
   - Write tests for settings persistence

8. ✅ Final verification:
   ```bash
   yarn test
   yarn tsc --noEmit  # No TypeScript errors
   yarn dev:web-app   # Should start and display remote models
   ```

9. Commit: `git commit -m "feat: update model selection for remote OpenAI-compatible APIs"`

**Estimated time**: 3-4 days
**Risk level**: MEDIUM (requires API integration testing)
**Testing requirement**: 80%+ test coverage, all tests passing

---

### Week 4: Add RAG Settings UI

**Goal**: Add user-facing settings for RAG library management

**TDD Approach**: Test-First Development (Red-Green-Refactor)

**Tasks**:

1. 🔴 **RED - Write tests first**:
   ```typescript
   // types/__tests__/RAGConfig.test.ts
   describe('RAGConfig', () => {
       it('should validate RAG configuration', () => {
           const config: RAGConfig = {
               chunkSize: 1000,
               chunkOverlap: 200,
               retrievalCount: 3,
               similarityThreshold: 0.7
           };
           expect(validateRAGConfig(config)).toBe(true);
       });

       it('should reject invalid chunk sizes', () => {
           const config: RAGConfig = {
               chunkSize: -1,  // Invalid
               chunkOverlap: 200,
               retrievalCount: 3,
               similarityThreshold: 0.7
           };
           expect(validateRAGConfig(config)).toBe(false);
       });
   });

   // hooks/__tests__/useRAGSettings.test.ts
   describe('useRAGSettings', () => {
       it('should load default settings on first use', () => {
           const { result } = renderHook(() => useRAGSettings());

           expect(result.current.config).toEqual({
               chunkSize: 1000,
               chunkOverlap: 200,
               retrievalCount: 3,
               similarityThreshold: 0.7
           });
       });

       it('should persist settings changes', async () => {
           const { result } = renderHook(() => useRAGSettings());

           act(() => {
               result.current.updateConfig({ chunkSize: 1500 });
           });

           await waitFor(() => {
               expect(result.current.config.chunkSize).toBe(1500);
           });

           // Verify persistence
           const savedConfig = await loadSettingsFromDisk();
           expect(savedConfig.chunkSize).toBe(1500);
       });
   });

   // routes/settings/__tests__/rag-settings.test.tsx
   describe('RAG Settings Panel', () => {
       it('should render all RAG configuration options', () => {
           render(<RAGSettingsPanel />);

           expect(screen.getByLabelText('Chunk Size')).toBeInTheDocument();
           expect(screen.getByLabelText('Chunk Overlap')).toBeInTheDocument();
           expect(screen.getByLabelText('Retrieval Count')).toBeInTheDocument();
           expect(screen.getByLabelText('Similarity Threshold')).toBeInTheDocument();
       });

       it('should update settings when sliders change', async () => {
           render(<RAGSettingsPanel />);

           const chunkSizeSlider = screen.getByLabelText('Chunk Size');
           fireEvent.change(chunkSizeSlider, { target: { value: '1500' } });

           await waitFor(() => {
               expect(screen.getByText('1500')).toBeInTheDocument();
           });
       });

       it('should show validation errors for invalid values', async () => {
           render(<RAGSettingsPanel />);

           const retrievalInput = screen.getByLabelText('Retrieval Count');
           fireEvent.change(retrievalInput, { target: { value: '0' } });

           await waitFor(() => {
               expect(screen.getByText('Must be at least 1')).toBeInTheDocument();
           });
       });
   });
   ```

2. 🟢 **GREEN - Implement minimal code**:
   - Create type definitions:
     ```typescript
     // core/src/types/RAGConfig.ts
     interface RAGConfig {
         chunkSize: number;        // Default: 1000
         chunkOverlap: number;     // Default: 200
         retrievalCount: number;   // Default: 3
         similarityThreshold: number;  // Default: 0.7
     }
     ```
   - Create settings hook:
     ```typescript
     // hooks/useRAGSettings.ts
     export function useRAGSettings() {
         const [config, setConfig] = useState<RAGConfig>(DEFAULT_RAG_CONFIG);

         useEffect(() => {
             loadSettings().then(setConfig);
         }, []);

         const updateConfig = async (updates: Partial<RAGConfig>) => {
             const newConfig = { ...config, ...updates };
             await saveSettings(newConfig);
             setConfig(newConfig);
         };

         return { config, updateConfig };
     }
     ```
   - Create UI component:
     ```typescript
     // routes/settings/rag-settings.tsx
     export function RAGSettingsPanel() {
         const { config, updateConfig } = useRAGSettings();

         return (
             <div>
                 <Slider
                     label="Chunk Size"
                     value={config.chunkSize}
                     onChange={(val) => updateConfig({ chunkSize: val })}
                     min={100}
                     max={2000}
                 />
                 {/* Other controls */}
             </div>
         );
     }
     ```

3. 🔵 **REFACTOR - Improve while keeping tests green**:
   - Add input validation
   - Add helpful tooltips
   - Add "Reset to defaults" button
   - Add visual feedback for saved state
   - Add document storage location picker
   - Add vector DB location display

4. 🧪 **Add integration tests**:
   ```typescript
   describe('RAG Settings Integration', () => {
       it('should apply settings to document processing', async () => {
           // Update chunk size
           await updateRAGSettings({ chunkSize: 1500 });

           // Upload document
           const doc = await uploadDocument('test.pdf');

           // Verify chunks respect new size
           const chunks = await getDocumentChunks(doc.id);
           expect(chunks.every(c => c.length <= 1500)).toBe(true);
       });
   });
   ```

5. ✅ **Verify all tests pass**:
   ```bash
   yarn test types/RAGConfig
   yarn test hooks/useRAGSettings
   yarn test routes/settings/rag-settings
   yarn test:coverage  # Ensure 80%+ coverage
   ```

6. 🧪 **Add Rust backend tests** (if settings affect backend):
   ```rust
   #[cfg(test)]
   mod tests {
       use super::*;

       #[test]
       fn test_load_rag_config_from_file() {
           let config = RAGConfig::load_from_appdata().unwrap();
           assert_eq!(config.chunk_size, 1000);
           assert_eq!(config.chunk_overlap, 200);
       }

       #[test]
       fn test_validate_rag_config() {
           let config = RAGConfig {
               chunk_size: 1000,
               chunk_overlap: 200,
               retrieval_count: 3,
               similarity_threshold: 0.7,
           };
           assert!(config.validate().is_ok());
       }
   }
   ```

7. ✅ Final verification:
   ```bash
   yarn test
   cargo test --workspace
   yarn dev:web-app  # Verify settings UI appears and works
   ```

8. Commit: `git commit -m "feat: add RAG settings for user customization"`

**Estimated time**: 2-3 days
**Risk level**: LOW (new feature, no removal)
**Testing requirement**: 80%+ test coverage, all tests passing, settings persistence verified

---

### Week 5: Backend & Event System Cleanup

**Goal**: Remove download/model loading code from Rust backend

**Tasks**:
1. Remove backend modules:
   - Delete `src-tauri/src/core/downloads/`
   - Update `src-tauri/src/core/app/models.rs` (remove local model structs)
2. Remove event types:
   - Remove `DownloadEvent` namespace
   - Remove model loading events
   - Keep document processing events
3. Add RAG events:
   - `EmbeddingProgress` event
   - `VectorSearchComplete` event
   - `DocumentProcessed` event
4. Remove build scripts:
   - Delete `scripts/download-bin.mjs`
   - Delete `scripts/download-lib.mjs`
   - Update `package.json` scripts
5. Verify builds:
   - `make clean && make dev` (should work end-to-end)
6. Commit: `git commit -m "refactor: remove download backend and add RAG events"`

**Estimated time**: 2-3 days
**Risk level**: MEDIUM

---

### Week 6: Documentation & Localization Updates

**Goal**: Clean up documentation and translations

**Tasks**:
1. Update README.md:
   - Remove "Run offline LLMs" messaging
   - Add "OpenAI-compatible API required"
   - Add RAG functionality explanation
   - Add Windows-only notice
2. Update CONTRIBUTING.md:
   - Remove llamacpp/hardware plugin references
   - Add RAG development guidelines
3. Remove localization files:
   - Delete all `system-monitor.json` files (14 files)
   - Update `settings.json` in all languages (remove hardware/download strings)
4. Update configuration:
   - Update `tauri.conf.json` (remove model bundling, keep ONNX)
   - Update capabilities (remove hardware/model management)
5. Commit: `git commit -m "docs: update documentation for fork"`

**Estimated time**: 1-2 days
**Risk level**: LOW

---

### Week 7: Final Verification & Testing

**Goal**: Ensure everything works end-to-end

**Tasks**:
1. Run verification checklist (see section above)
2. Test critical paths:
   - App starts successfully
   - Can configure OpenAI-compatible API
   - Can fetch and select models from API
   - Can upload documents (when implemented)
   - Settings persist correctly
3. Automated search for remaining references:
   ```bash
   rg -i "llamacpp" --type ts --type tsx --type rust
   rg -i "plugin.*hardware" --type ts --type tsx --type rust
   rg -i "download.*extension" --type ts --type tsx
   rg -i "gguf" --type ts --type tsx --type rust
   ```
4. Fix any remaining references
5. Performance check:
   - Build time should be faster (~40% reduction)
   - App size should be smaller
6. Create PR:
   - Detailed description of changes
   - Before/after metrics (LOC, build time, installer size)
   - Testing performed
7. Merge to `dev` branch

**Estimated time**: 2-3 days
**Risk level**: LOW (testing and verification)

---

## Summary: 7-Week Cleanup Plan

| Week | Phase | Risk | Estimated Days |
|------|-------|------|----------------|
| 1 | Remove plugins & extensions | HIGH | 1-2 |
| 2 | Remove frontend routes & UI | MEDIUM | 2-3 |
| 3 | Modify model selection for APIs | MEDIUM | 3-4 |
| 4 | Add RAG settings UI | LOW | 2-3 |
| 5 | Backend & event cleanup | MEDIUM | 2-3 |
| 6 | Documentation updates | LOW | 1-2 |
| 7 | Verification & testing | LOW | 2-3 |
| **Total** | | | **~15-20 days** |

**After completion**:
- Codebase reduced by ~20,000-30,000 lines
- Build time reduced by ~40%
- Installer size reduced by ~70% (500MB → 150MB)
- Ready for Phase 1 implementation (ONNX embedding engine)

---

**Document Version**: 1.1
**Last Updated**: 2026-01-02
**Status**: Action Plan Ready
