# Product Overview

## What is droid-acp?

droid-acp is an adapter that integrates Factory.ai's Droid AI coding agent with Zed editor and other ACP-compliant editors. It acts as a bridge between the Agent Client Protocol (ACP) and Droid CLI, enabling seamless AI-assisted development directly within the editor environment.

## Core Value Proposition

- **Universal Editor Integration**: Makes Droid AI accessible from any ACP-compatible editor (Zed, VS Code, etc.)
- **Real-time Collaboration**: Provides streaming output showing Droid's thinking process and code generation
- **Safe AI Operations**: Implements risk-based permission management for file operations and system commands
- **Developer Productivity**: Eliminates context switching between CLI and editor for AI assistance

## Target Users

- **Primary**: Developers using Zed editor who want AI coding assistance
- **Secondary**: Developers using other ACP-compatible editors
- **Future**: Teams wanting collaborative AI development workflows

## Key Features

1. **ACP Protocol Implementation**: Full compliance with Agent Client Protocol for editor integration
2. **Droid CLI Management**: Automated process management and communication with Factory.ai Droid
3. **Real-time Streaming**: Live display of AI thinking, planning, and code generation
4. **File Operations**: Safe file reading/writing with diff generation and user confirmation
5. **Permission System**: Risk-level based authorization for dangerous operations
6. **Session Management**: Context-aware conversations with project state preservation

## Success Metrics

- **Technical**: Response time < 100ms, Memory usage < 20MB/session, Error rate < 1%
- **Adoption**: 1000+ monthly downloads, 100+ GitHub stars, 50+ active users
- **Quality**: 80%+ test coverage, 5+ active contributors

## License & Distribution

- **License**: MIT (open source)
- **Distribution**: npm package, Cargo crate, GitHub releases, Homebrew formula
- **Community**: Open source development with community contributions welcome