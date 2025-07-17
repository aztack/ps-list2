# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ps-list** is a cross-platform Node.js library that retrieves information about running processes on macOS, Linux, and Windows. It provides a unified interface across different operating systems with platform-specific implementations.

## Development Commands

### Build & Development
- `npm run build` - Builds both CommonJS and ESM formats, then generates TypeScript declarations
- `npm run tsc` - Generates TypeScript declaration files only
- `npm test` - Runs AVA tests against built outputs

### Testing
- Tests are located in `test.js` and use AVA framework
- Type tests are in `index.test-d.ts` using TSD
- Tests require the project to be built first (`npm run build`)

## Key Architecture

### Platform-Specific Implementation
The library uses platform detection (`process.platform === 'win32'`) to choose between:

**Windows**: Uses precompiled binaries from the `vendor/` directory:
- `fastlist-0.3.0-x64.exe` for x64 architecture
- `fastlist-0.3.0-x86.exe` for ia32 architecture
- Limited feature set (no `cmd`, `cpu`, `memory`, `uid` properties)

**Unix-like (macOS/Linux)**: Uses native `ps` command with dual parsing strategy:
- Primary: Single `ps` call with regex parsing (`nonWindowsCall`)
- Fallback: Multiple `ps` calls if parsing fails (`nonWindowsMultipleCalls`)
- Full feature support including `cmd`, `cpu`, `memory`, `uid` properties

### Build System
- **esbuild** generates dual module formats (CJS and ESM)
- **TypeScript** compilation requires specific flags: `--declaration --emitDeclarationOnly --esModuleInterop --module esnext --target es2020`
- **Compatible @types/node version**: Uses @types/node@14 for TypeScript 4.9.5 compatibility

### Error Handling Pattern
Non-Windows implementation uses graceful fallback:
```typescript
try {
  return await nonWindowsCall(options);
} catch {
  return nonWindowsMultipleCalls(options);
}
```

## File Structure

- `src/index.ts` - Main implementation with platform-specific logic
- `esbuild.js` - Build configuration for dual module output
- `test.js` - Runtime tests using AVA
- `index.test-d.ts` - TypeScript type tests
- `vendor/` - Windows binary executables
- `dist/` - Built outputs (CJS, ESM, and type declarations)

## Important Notes

### TypeScript Configuration
- Uses TypeScript 4.9.5 with strict type checking
- Requires compatible @types/node version (currently @types/node@14)
- Module resolution configured for Node.js ES modules

### Process Filtering
The Unix implementation filters out `ps` processes themselves to avoid self-reference using `psPids` tracking.

### Platform Feature Differences
Windows implementation has limited properties:
- ✅ `pid`, `name`, `ppid`
- ❌ `cmd`, `cpu`, `memory`, `uid` (undefined on Windows)

### Testing Strategy
- Cross-platform compatibility testing
- Real process spawn testing with cleanup
- Property validation varies by platform
- Type-level testing for API contracts