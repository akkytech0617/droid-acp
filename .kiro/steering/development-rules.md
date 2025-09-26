# Development Rules & Guidelines

## Language and Communication Rules

### Documentation and Code
- **All code comments, documentation, and technical writing MUST be in English**
- **Variable names, function names, and API documentation MUST be in English**
- **Git commit messages MUST be in English using conventional commits format**
- **README files and technical specifications MUST be in English**

### User Communication
- **All communication with users (including Japanese users) SHOULD be in Japanese**
- **Error messages shown to end users SHOULD be in Japanese when appropriate**
- **User-facing documentation MAY be provided in both English and Japanese**

## Development Methodology

### Test-Driven Development (TDD)
- **MUST follow TDD cycle**: Red → Green → Refactor
- **Write failing tests BEFORE implementing functionality**
- **All new features MUST have corresponding tests written first**
- **Refactor only after tests are passing**
- **Maintain test coverage above 80%**

### Technology Research Protocol
- **MUST use MCP (Model Context Protocol) to verify latest technical information before implementation**
- **Check official documentation and recent updates for all dependencies**
- **Verify compatibility and best practices using MCP tools**
- **Document any breaking changes or deprecated features discovered**

## Task Management Framework

### Task Planning Format
Every task execution MUST begin with a structured plan using the following format:

#### Required Items
- **完了時の状態** (Completion State): What state will be achieved when this task is finished
- **設計方針** (Design Approach): What design approach will be taken
- **実装対象** (Implementation Targets): Which files will be implemented and what functionality
- **テスト戦略** (Test Strategy): What tests will be created (following TDD)
- **検証方法** (Verification Method): How completion will be confirmed

#### Recommended Items
- **前提条件** (Prerequisites): Conditions required to start this task
- **依存関係** (Dependencies): Relationships with other tasks or components
- **リスク要因** (Risk Factors): Expected issues or points of attention
- **参考資料** (Reference Materials): Related documents or code examples
- **レビューポイント** (Review Points): Areas that should be carefully reviewed

### Task Planning Template

```markdown
## タスク: [Task Name]

### 完了時の状態
- [ ] Specific completion state 1
- [ ] Specific completion state 2

### 設計方針
- Basic design principles
- Architectural considerations

### 実装対象
- `src/example.ts`: Implementation of Feature A
- `tests/example.test.ts`: Tests for Feature A

### テスト戦略（TDD）
1. Create test case 1
2. Create test case 2
3. Implement functionality to pass tests

### 検証方法
- Confirm test execution results
- Manual verification procedures

### 前提条件
- Required prerequisites

### 依存関係
- Dependent tasks or components

### リスク要因
- Expected issues

### 参考資料
- Links to related documentation or technical information
```

## Implementation Guidelines

### Code Quality Standards
- **Follow TypeScript strict mode settings**
- **Use ESLint and Prettier with project configurations**
- **Write self-documenting code with clear variable and function names**
- **Add JSDoc comments for public APIs**
- **Handle errors explicitly and provide meaningful error messages**

### Git Workflow
- **Create feature branches for each task**: `feature/task-description`
- **Use conventional commits**: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`
- **Squash commits before merging to main**
- **Include task completion checklist in PR descriptions**

### Performance Considerations
- **Profile code before optimization**
- **Measure memory usage and startup time**
- **Use async/await appropriately for I/O operations**
- **Implement proper error boundaries and timeouts**

## Review and Quality Assurance

### Code Review Checklist
- [ ] Tests written first (TDD compliance)
- [ ] All tests passing
- [ ] Code follows project style guidelines
- [ ] Error handling implemented
- [ ] Performance considerations addressed
- [ ] Documentation updated
- [ ] No security vulnerabilities introduced

### Definition of Done
A task is considered complete when:
1. All planned tests are written and passing
2. Implementation meets the specified completion state
3. Code review has been completed
4. Documentation has been updated
5. Integration tests pass
6. Performance requirements are met

## MCP Integration Requirements

### Before Starting Implementation
- [ ] Use MCP to check latest versions of dependencies
- [ ] Verify current best practices for the technology stack
- [ ] Check for any recent security advisories
- [ ] Confirm API compatibility and breaking changes

### During Implementation
- [ ] Use MCP to resolve technical questions
- [ ] Verify implementation patterns against current standards
- [ ] Check for updated documentation or examples

### Technology Stack Verification
- **Agent Client Protocol**: Verify latest ACP specification
- **TypeScript**: Check for latest features and best practices
- **Node.js**: Confirm compatibility and performance optimizations
- **Testing frameworks**: Verify current testing patterns and tools