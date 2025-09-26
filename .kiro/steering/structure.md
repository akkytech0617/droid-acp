# Project Organization & Structure

## Repository Layout

```
droid-acp/
├── .git/                     # Git repository data
├── .gitignore               # Git ignore patterns (Rust-focused)
├── LICENSE                  # MIT License
├── README.md               # Project overview and quick start
├── .kiro/                  # Kiro IDE configuration
│   ├── steering/           # AI assistant guidance (this file)
│   └── specs/              # Feature specifications
│       └── droid-acp-adapter/
├── docs/                   # Comprehensive documentation
│   ├── bucklog.md         # Product backlog (Japanese)
│   ├── design_rust.md     # Rust implementation design
│   └── design_ts.md       # TypeScript implementation design
└── src/                    # Source code (to be created)
```

## Implementation Phases

### Phase 1: TypeScript MVP
The initial implementation focuses on rapid prototyping and validation:

```
src/
├── index.ts                # CLI entry point
├── adapter/
│   ├── index.ts           # Main DroidACPAdapter class
│   ├── connection.ts      # ACP connection management
│   └── handler.ts         # Message routing and handling
├── droid/
│   ├── index.ts           # Droid CLI interface
│   ├── process.ts         # Process lifecycle management
│   └── parser.ts          # Output stream parsing
├── protocol/
│   ├── index.ts           # ACP protocol types and methods
│   ├── types.ts           # TypeScript type definitions
│   └── methods.ts         # ACP method implementations
├── session/
│   ├── index.ts           # Session management
│   └── state.ts           # Session state tracking
├── permission/
│   ├── index.ts           # Permission system
│   └── handler.ts         # Risk assessment and approval
└── utils/
    ├── logger.ts          # Structured logging
    ├── config.ts          # Configuration management
    └── errors.ts          # Error types and handling
```

### Phase 2: Rust Production
The production implementation emphasizes performance and reliability:

```
src/
├── main.rs                # Binary entry point
├── lib.rs                 # Library root
├── adapter/
│   ├── mod.rs            # Module declaration
│   ├── connection.rs     # ACP connection (async)
│   └── handler.rs        # Message handling
├── droid/
│   ├── mod.rs            # Module declaration
│   ├── process.rs        # Tokio process management
│   └── parser.rs         # Stream parsing with state machine
├── protocol/
│   ├── mod.rs            # Module declaration
│   ├── types.rs          # Serde-based type definitions
│   └── methods.rs        # ACP method implementations
├── session/
│   ├── mod.rs            # Module declaration
│   └── state.rs          # Thread-safe session state
└── permission/
    ├── mod.rs            # Module declaration
    └── handler.rs        # Permission management
```

## Configuration Structure

### Environment Configuration
- `.env.example` - Template for environment variables
- Configuration precedence: CLI args > env vars > config file > defaults

### Package Configuration
- `package.json` - Node.js project metadata and scripts
- `tsconfig.json` - TypeScript compiler configuration
- `Cargo.toml` - Rust project metadata and dependencies (future)

## Documentation Organization

### User Documentation
- `README.md` - Quick start and basic usage
- `docs/` - Comprehensive guides and API reference
- Inline code comments for complex logic
- JSDoc/rustdoc for API documentation

### Development Documentation
- `docs/design_*.md` - Architecture and implementation details
- `docs/bucklog.md` - Product requirements and user stories
- `.kiro/specs/` - Detailed feature specifications
- Code comments explaining business logic and design decisions

## Testing Structure

### TypeScript Testing
```
tests/
├── unit/                  # Isolated component tests
│   ├── adapter.test.ts
│   ├── droid.test.ts
│   └── parser.test.ts
├── integration/           # Multi-component tests
│   ├── e2e.test.ts
│   └── protocol.test.ts
├── fixtures/              # Test data and mocks
│   ├── droid-responses/
│   └── acp-messages/
└── helpers/               # Test utilities
    └── mocks.ts
```

### Test Data Management
- Mock Droid CLI responses in `fixtures/`
- ACP protocol test messages
- Temporary file creation for file operation tests
- Process mocking for unit tests

## Build Artifacts

### Development Artifacts
- `dist/` - Compiled TypeScript output
- `target/` - Rust build artifacts (future)
- `node_modules/` - Node.js dependencies
- Coverage reports in `coverage/`

### Distribution Artifacts
- npm package with compiled JavaScript
- Standalone binaries (pkg for TypeScript, native for Rust)
- Docker images for containerized deployment
- GitHub releases with platform-specific binaries

## Code Organization Principles

### Modular Architecture
- **Single Responsibility**: Each module handles one concern
- **Dependency Injection**: Easy testing and configuration
- **Interface Segregation**: Small, focused interfaces
- **Error Boundaries**: Isolated error handling per module

### File Naming Conventions
- **kebab-case** for file names and directories
- **PascalCase** for class names and types
- **camelCase** for functions and variables
- **SCREAMING_SNAKE_CASE** for constants

### Import/Export Patterns
- Barrel exports (`index.ts`) for clean module interfaces
- Explicit imports for better tree-shaking
- Type-only imports where applicable
- Consistent import ordering (external, internal, relative)

## Development Workflow

### Branch Strategy
- `main` - Production-ready code
- `develop` - Integration branch for features
- `feature/*` - Individual feature development
- `hotfix/*` - Critical production fixes

### Code Review Process
- All changes via pull requests
- Automated checks: lint, test, build
- Manual review for architecture and business logic
- Documentation updates required for new features

## Deployment Structure

### Package Distribution
- **npm**: `@community/droid-acp-adapter`
- **Cargo**: `droid-acp-adapter` (future)
- **GitHub Releases**: Platform binaries
- **Docker Hub**: Container images

### Installation Paths
- Global npm install: `~/.npm/bin/droid-acp-adapter`
- Cargo install: `~/.cargo/bin/droid-acp-adapter`
- Homebrew: `/opt/homebrew/bin/droid-acp-adapter`
- Manual: User-specified location