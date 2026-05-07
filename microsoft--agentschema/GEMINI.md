## agentschema

> AgentSchema is a TypeSpec-based schema definition project that generates runtime libraries for C#, Python, TypeScript, and Go. The project defines a specification for AI agents with structured metadata, inputs, outputs, tools, and templates.

# AgentSchema Project Instructions

## Project Overview

AgentSchema is a TypeSpec-based schema definition project that generates runtime libraries for C#, Python, TypeScript, and Go. The project defines a specification for AI agents with structured metadata, inputs, outputs, tools, and templates.

## Quick Reference

- **Schema Source**: `agentschema-emitter/lib/model/*.tsp` (TypeSpec files)
- **Emitter Source**: `agentschema-emitter/src/` (TypeScript + Nunjucks templates)
- **Generated Output**: `runtime/`, `schemas/`, `docs/src/content/docs/reference/`

See detailed instructions in `.github/instructions/` for specific file types.

## Project Structure

```
AgentSchema/
├── agentschema-emitter/   # Self-contained TypeSpec emitter package
│   ├── lib/model/         # .tsp files defining the schema (source of truth)
│   ├── src/               # Emitter source code
│   │   ├── templates/     # Nunjucks templates for code generation
│   │   │   ├── csharp/
│   │   │   ├── python/
│   │   │   ├── typescript/
│   │   │   └── go/
│   │   ├── csharp.ts      # C# code generator
│   │   ├── python.ts      # Python code generator
│   │   ├── typescript.ts  # TypeScript code generator
│   │   └── go.ts          # Go code generator
│   └── dist/              # Compiled emitter output
├── runtime/               # Generated runtime libraries
│   ├── csharp/            # C# library (AgentSchema.dll)
│   ├── python/            # Python package (agentschema)
│   ├── typescript/        # TypeScript/npm package (agentschema)
│   └── go/                # Go module (agentschema)
├── schemas/               # Generated JSON/YAML schemas
└── docs/                  # Astro documentation site
```

## Build Workflow

1. **Edit TypeSpec** (`agentschema-emitter/lib/model/*.tsp`) - Define or modify schema
2. **Build Emitter** (`agentschema-emitter/`) - `npm run build`
3. **Generate Code** (`agentschema-emitter/`) - `npm run generate`
4. **Test Runtimes** - Run tests in each runtime folder

## Key Commands

```bash
# Build the emitter
cd agentschema-emitter && npm run build

# Generate all runtime code
cd agentschema-emitter && npm run generate

# Run tests
cd runtime/csharp && dotnet test
cd runtime/python/agentschema && uv run pytest tests/
cd runtime/typescript/agentschema && npm test
cd runtime/go/agentschema && go test ./...
```

## Code Generation Architecture

- **TypeSpec models** define the schema with `@sample` and `@shorthand` decorators
- **Emitter** (`agentschema-emitter`) parses TypeSpec AST and generates code via Nunjucks templates
- **Templates** produce idiomatic code for each language with serialization/deserialization logic
- **Tests** are auto-generated alongside the runtime code

## Important Conventions

### TypeSpec Files

- All models use `namespace AgentSchema;` (not `AgentSchema.Core`)
- Use `@sample()` decorator to provide example values for test generation
- Use `@shorthand()` for alternate scalar representations
- Use `@abstract` for base types with polymorphic children

### Generated Code Characteristics

- **C#**: PascalCase properties, `FromJson`/`FromYaml`/`ToJson`/`ToYaml` methods
- **Python**: snake_case internally, `load()`/`save()`/`to_json()`/`to_yaml()` methods
- **TypeScript**: camelCase properties, `fromJson`/`fromYaml`/`toJson`/`toYaml` methods

### When Modifying the Emitter

1. Edit source in `agentschema-emitter/src/`
2. Templates are in `src/templates/{language}/`
3. Rebuild with `npm run build` (compiles TypeScript AND copies templates)
4. Regenerate and test all runtimes

## Common Tasks

### Adding a New Property to a Model

1. Edit the `.tsp` file in `agentschema-emitter/lib/model/`
2. Add `@sample()` decorator with example value
3. Regenerate code: `cd agentschema-emitter && npm run generate`
4. Run tests in all runtimes

### Adding a New Model Type

1. Create or edit `.tsp` file with new model
2. Import it in `main.tsp` if it's a new file
3. Add `@sample()` decorators for test generation
4. Regenerate and verify tests pass

### Modifying Code Generation Templates

1. Edit `.njk` template in `agentschema-emitter/src/templates/{language}/`
2. Rebuild emitter: `npm run build`
3. Regenerate: `npm run generate`
4. Test affected runtime

## Do NOT

- Edit files in `runtime/` directly - they are auto-generated
- Edit files in `schemas/` directly - they are auto-generated
- Edit files in `docs/src/content/docs/reference/` directly - they are auto-generated
- Forget to copy templates to `dist/` after editing them
- Use `AgentSchema.Core` namespace - use `AgentSchema` only

## Related Instructions

For detailed guidance on specific areas:

- [TypeSpec editing](./instructions/typespec.instructions.md)
- [Emitter development](./instructions/emitter.instructions.md)
- [Template editing](./instructions/templates.instructions.md)
- [Publishing packages](./instructions/publishing.instructions.md)

## Prompt Files

Use these for common tasks:

- [regenerate.prompt.md](./prompts/regenerate.prompt.md) - Full rebuild workflow
- [add-property.prompt.md](./prompts/add-property.prompt.md) - Adding properties to models
- [add-model.prompt.md](./prompts/add-model.prompt.md) - Creating new model types

---
> Source: [microsoft/AgentSchema](https://github.com/microsoft/AgentSchema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-07 -->
