---
url: "https://github.com/standard-community/standard-json"
clipped: 2026-08-03
title: "Standard JSON — Standard Schema to JSON Schema convertor"
---

# standard-community/standard-json

Standard Schema to JSON Schema convertor

- Stars: 50
- Forks: 4
- License: MIT
- npm: https://www.npmjs.com/package/@standard-community/standard-json
- Package: `@standard-community/standard-json` (v0.3.5 as of clip date)
- Homepage: https://github.com/standard-community

## About

Standard Schema Validator's JSON Schema Converter — converts schemas that implement the [Standard Schema](https://github.com/standard-schema/standard-schema) interface into JSON Schema.

## Installation

Install the main package:

```sh
pnpm add @standard-community/standard-json
```

For some specific vendors, install the respective package also:

| Vendor  | Package                 |
| ------- | ----------------------- |
| Zod v3  | `zod-to-json-schema`    |
| Valibot | `@valibot/to-json-schema` |

## Usage

```ts
import { toJsonSchema } from "@standard-community/standard-json";

// Define your validation schema
const schema = {
    // ...
};

// Convert it to JSON Schema
const jsonSchema = await toJsonSchema(schema);
```

### Sync Usage

This is useful for:

#### Adding support for Unsupported validation libs

```ts
import { toJsonSchema, loadVendor } from "@standard-community/standard-json";
import { convertSchemaToJson } from "your-validation-lib";

// The lib should support Standard Schema, like Sury
// as we use 'schema["~standard"].vendor' to get the vendor name
// Eg. loadVendor(zod["~standard"].vendor, convertorFunction)
loadVendor("validation-lib-name", convertSchemaToJson)

// Define your validation schema
const schema = {
    // ...
};

// Convert it to JSON Schema
const jsonSchema = toJsonSchema(schema);
```

#### Customize the toJSONFunction of a supported lib

```ts
import { toJsonSchema, loadVendor } from "@standard-community/standard-json";
import zodHandler from "@standard-community/standard-json/zod";

// Or pass a custom implmentation
loadVendor("zod", zodHandler())

// Define your validation schema
const schema = {
    // ...
};

// Convert it to JSON Schema
const jsonSchema = await toJsonSchema(schema);
```

## Package metadata (from package.json)

Keywords: `standard-schema`, `standard-community`, `standard-schema-community`, `json-schema`, `convertor`

Peer dependencies (many optional):

- `@standard-schema/spec` ^1.0.0
- `@types/json-schema` ^7.0.15
- `zod` ^3.25.0 \|\| ^4.0.0
- `zod-to-json-schema` ^3.24.5 (optional)
- `valibot` ^1.1.0 (optional)
- `@valibot/to-json-schema` ^1.3.0 (optional)
- `arktype` ^2.1.20 (optional)
- `effect` ^3.16.8 (optional)
- `typebox` ^1.0.17 (optional)
- `sury` ^10.0.0 (optional)

Runtime dependency: `quansync` ^0.2.11

Exports: ESM + CJS (`dist/index.js` / `dist/index.cjs`), types included.

## Repository structure

- `src/` — library source
- `tests/` — vitest tests
- `.github/workflows/` — CI
- Tooling: Biome, TypeScript, Vitest, pkgroll, pnpm
