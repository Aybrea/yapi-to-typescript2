# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`yapi-to-typescript` (ytt) is a code generation tool that generates TypeScript/JavaScript interface types and request functions from YApi or Swagger API definitions. The tool can be used as a CLI (`ytt`) or programmatically via its exported API.

## Build and Development Commands

```bash
# Build the project (compiles TypeScript to both CJS and ESM)
npm run build

# Run tests
npm test                   # Run tests with coverage
npm run testOnly           # Run tests without coverage
npm run testUpdateSnapshot # Update test snapshots

# Test the CLI locally
npm run testApi # Runs ts-node -T src/cli.ts

# Documentation
npm run docs:dev   # Start documentation dev server
npm run docs:build # Build documentation
npm run docs       # Build docs and deploy to gh-pages

# Release
npm run release     # Full release: test, version bump, build, docs, publish
npm run releaseBeta # Beta release with --prerelease beta tag
```

## Architecture

### Core Components

1. **Generator** (`src/Generator.ts`)

   - Main orchestrator that processes configurations and generates output files
   - Handles both YApi and Swagger sources
   - Manages the entire code generation pipeline: fetching API definitions, processing interfaces, generating TypeScript types and request functions
   - Outputs files are grouped by `outputFilePath` configuration

2. **SwaggerToYApiServer** (`src/SwaggerToYApiServer.ts`)

   - Adapter that converts Swagger definitions to YApi format
   - Starts a local HTTP server that mimics YApi API endpoints
   - Allows the tool to work with Swagger sources using the same pipeline as YApi

3. **CLI** (`src/cli.ts`)

   - Entry point for the `ytt` command
   - Uses ts-node to support both `.ts` and `.js` config files
   - Commands: `ytt init` (initialize config), `ytt` (generate code), `ytt help`
   - Config files: `ytt.config.ts` or `ytt.config.js` (or custom path with `-c`)

4. **Types** (`src/types.ts`)

   - Central type definitions for the entire project
   - Defines configuration interfaces, API data structures, and code generation options

5. **Helpers** (`src/helpers.ts`)

   - Exports `defineConfig()` helper for type-safe configuration
   - Exports `FileData` class for handling file uploads across platforms

6. **Utils** (`src/utils.ts`)
   - Utility functions for JSON Schema conversion, HTTP requests, Prettier formatting, etc.

### Configuration Hierarchy

The tool supports a three-level configuration hierarchy (server → project → category), where lower levels override higher levels:

1. **Server level**: `serverType`, `serverUrl`, `projects[]`
2. **Project level**: `token`, `categories[]`, plus common configs
3. **Category level**: `id`, plus common configs

Common configs include: `target`, `outputFilePath`, `requestFunctionFilePath`, `dataKey`, `reactHooks`, `jsonSchema`, `comment`, etc.

### Code Generation Flow

1. Config is loaded from `ytt.config.ts/js`
2. If `serverType: 'swagger'`, SwaggerToYApiServer starts a local server to adapt Swagger to YApi format
3. Generator fetches interface definitions from YApi (or the Swagger adapter)
4. For each interface:
   - Preprocesses via `preproccessInterface` hook if configured
   - Generates TypeScript types from JSON Schema using `json-schema-to-typescript`
   - Generates request functions with proper typing
   - Optionally generates React Hooks wrappers
   - Optionally includes JSON Schema for runtime validation
5. Groups generated code by output file path
6. Formats output with Prettier (prefers project-local Prettier, supports Prettier 3)
7. Writes files to disk

### Key Features

- **Dual Source Support**: Works with both YApi and Swagger API definitions
- **Flexible Output**: Can generate TypeScript or JavaScript, types-only or full request functions
- **React Hooks**: Optional React Hooks generation for API calls
- **JSON Schema**: Optional JSON Schema generation for request/response validation
- **Customization**: Extensive hooks for customizing naming, preprocessing, and metadata
- **Multi-format**: Supports various query string array formats (brackets, indices, repeat, comma, json)

## Testing

Tests are located in `tests/` directory and use Jest. The project uses `haoma` for Jest configuration. Test files mirror the source structure:

- `tests/Generator.test.ts`
- `tests/SwaggerToYApiServer.test.ts`
- `tests/cli.test.ts`
- `tests/helpers.test.ts`

## Build System

The project uses `haoma` (a build tool) for compilation:

- Compiles to both CommonJS (`lib/cjs/`) and ESM (`lib/esm/`)
- Entry points: `lib/cjs/index.js` (main), `lib/esm/index.js` (module)
- CLI binary: `lib/cjs/cli.js` (exposed as `ytt` command)

## Important Notes

- The CLI uses ts-node with `transpileOnly: true` to support TypeScript config files without type checking
- Prettier formatting prioritizes project-local Prettier installation and supports Prettier 3
- When working with Swagger sources, path parameters and query parameters are transparently passed through with their types
- The tool supports custom type mapping via `customTypeMapping` config to convert non-JSONSchema types (e.g., Java types like `byte`, `short`, `int`) to JSONSchema types
