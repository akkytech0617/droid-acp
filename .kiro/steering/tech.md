# Technology Stack & Build System

## Implementation Strategy

This project uses a **dual-language approach** with staged implementation:

### Phase 1: TypeScript MVP (Current Focus)
- **Runtime**: Node.js 18+
- **Language**: TypeScript 5.0+
- **Key Dependencies**:
  - `@zed-industries/agent-client-protocol` - ACP protocol implementation
  - `zod` - Runtime type validation
  - Standard Node.js modules for process management and file operations

### Phase 2: Rust Production (Future)
- **Language**: Rust 2021 edition
- **Runtime**: Tokio async runtime
- **Key Dependencies**:
  - `agent-client-protocol` crate - ACP protocol
  - `tokio` - Async runtime and process management
  - `serde` + `serde_json` - Serialization
  - `anyhow` + `thiserror` - Error handling

## Build Commands

### TypeScript Development
```bash
# Install dependencies
npm install

# Development build with watch
npm run dev

# Production build
npm run build

# Run tests
npm test

# Lint and format
npm run lint
npm run format

# Start the adapter
npm start
# or
npx droid-acp-adapter
```

### Rust Development (Future)
```bash
# Build debug version
cargo build

# Build release version
cargo build --release

# Run tests
cargo test

# Run with logging
RUST_LOG=debug cargo run

# Install locally
cargo install --path .
```

## Project Structure Conventions

### TypeScript Structure
```
src/
├── index.ts              # Main entry point
├── adapter/              # Core adapter logic
├── droid/               # Droid CLI integration
├── protocol/            # ACP protocol implementation
├── session/             # Session management
├── permission/          # Permission handling
└── utils/               # Shared utilities

tests/
├── unit/                # Unit tests
├── integration/         # Integration tests
└── fixtures/            # Test data
```

### Configuration Files
- `package.json` - Node.js project configuration
- `tsconfig.json` - TypeScript compiler settings
- `jest.config.js` - Test configuration
- `.env.example` - Environment variable template

## Environment Variables

### Required
- `FACTORY_API_KEY` - Factory.ai API authentication token

### Optional
- `DROID_EXECUTABLE` - Path to Droid CLI (default: "droid")
- `DROID_AUTO_APPROVE_LEVEL` - Auto-approval level: "low", "medium", "high"
- `DROID_ACP_DEBUG` - Enable debug logging (default: false)
- `DROID_ACP_LOG_LEVEL` - Log level: "error", "warn", "info", "debug"

## Development Workflow

### Code Style
- **TypeScript**: Use Prettier + ESLint with strict settings
- **Rust**: Use `rustfmt` + `clippy` with default settings
- **Commits**: Conventional commits format
- **Branches**: Feature branches with descriptive names

### Testing Strategy
- **Unit Tests**: Mock external dependencies (Droid CLI, file system)
- **Integration Tests**: Use temporary directories and mock Droid responses
- **E2E Tests**: Test with actual Droid CLI in CI environment

### Performance Requirements
- **Startup Time**: < 500ms (TypeScript), < 100ms (Rust)
- **Memory Usage**: < 20MB per session
- **Response Latency**: < 50ms for local operations
- **Stream Processing**: < 50ms buffering delay

## Dependencies Management

### TypeScript
- Keep dependencies minimal for faster startup
- Use exact versions in package-lock.json
- Regular security audits with `npm audit`

### Rust
- Use workspace for multi-crate setup if needed
- Pin major versions in Cargo.toml
- Regular `cargo audit` for security

## CI/CD Pipeline

### GitHub Actions Workflow
```yaml
# Automated on push/PR
- Lint and format check
- Unit and integration tests
- Build verification
- Security audit
- Performance regression tests

# Release workflow
- Automated versioning
- Multi-platform builds
- Package publishing (npm/crates.io)
- GitHub releases with binaries
```

## Distribution Targets

### TypeScript Package
- **npm**: `@community/droid-acp-adapter`
- **Binary**: Packaged with `pkg` for standalone execution

### Rust Package (Future)
- **crates.io**: `droid-acp-adapter`
- **GitHub Releases**: Pre-built binaries for major platforms
- **Homebrew**: Formula for macOS installation