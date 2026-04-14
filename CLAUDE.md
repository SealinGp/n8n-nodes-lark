# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An n8n community node package that integrates Feishu/Lark APIs into n8n workflows. Provides three nodes: `Lark` (main action node), `LarkTrigger` (event subscription via WebSocket), and an MCP adapter.

## Commands

```bash
# Build (compile TypeScript + copy SVG icons to dist/)
npm run build

# Watch mode for development
npm run dev

# Run all tests
npm test

# Run a single test file
npx jest __tests__/path/to/test.test.ts

# Lint
npm run lint
npm run lintfix
```

> The project uses `npm`, not `bun`, despite any prior session notes.

After modifying TypeScript source, always run `npm run build` before testing in n8n — n8n loads from `dist/`.

## Architecture

### Resource/Operation Registration (Critical Pattern)

The node dynamically auto-discovers resources and operations at runtime using `ResourceFactory.build(__dirname)` in `Lark.node.ts`:

1. **`ModuleLoadUtils.loadModules(basedir, 'resource/*.js')`** — scans compiled JS files directly in `nodes/Lark/resource/` to find resource definitions. Each file exports `{ name, value, order }`.
2. **`ModuleLoadUtils.loadModules(basedir, 'resource/${resource.value}/*.js')`** — for each registered resource, loads operation files from its subdirectory. Each file exports `{ name, value, order, options[], call() }`.
3. `ResourceBuilder` assembles the n8n `INodeProperties[]` from these dynamically.

**Key consequence**: A resource folder (e.g., `wiki_spaces/`) is **invisible** unless a corresponding `.ts` file exists in `nodes/Lark/resource/` (e.g., `Wiki.resource.ts`) that exports `{ name, value: ResourceType.Wiki, order }`.

### Adding a New Resource (Checklist)

To expose a resource like `wiki_spaces` that already has operation files but doesn't appear in the UI:

1. Create `nodes/Lark/resource/Wiki.resource.ts` exporting `{ name: WORDING.ResourceWiki, value: ResourceType.Wiki, order: N }`.
2. Add `ResourceWiki: string` to `nodes/help/i18n/types.ts` → `IWording`.
3. Add `ResourceWiki: 'Wiki Spaces'` to `nodes/help/i18n/locales/en.ts`.
4. Add `ResourceWiki: '知识空间'` to `nodes/help/i18n/locales/zh.ts`.
5. Run `npm run build`.

### Directory Layout

```
nodes/
  Lark/
    Lark.node.ts          # Main node, wires up ResourceFactory + listSearch methods
    LarkTrigger.node.ts   # WebSocket/webhook trigger node
    GenericFunctions.ts   # listSearch helpers (calendar, sheet, bitable, etc.)
    resource/
      *.resource.ts       # Resource registrations (one per resource)
      base/               # Operations for Base (Bitable) resource
      calendar/
      contacts/
      document/
      message/
      space/
      spreadsheet/
      wiki_spaces/        # ⚠️ Operations exist but resource file is missing
  help/
    builder/
      ResourceFactory.ts  # Auto-discovers & assembles resources from JS files
      ResourceBuilder.ts  # Builds INodeProperties[] and routes execute calls
    type/
      IResource.ts        # ResourceOperation / ResourceOptions types
      enums.ts            # ResourceType, OperationType, FileType, etc.
    i18n/
      types.ts            # IWording interface — add keys here for new resources
      locales/en.ts
      locales/zh.ts
    utils/
      ModuleLoadUtils.ts  # Glob-based JS module loader
      RequestUtils.ts     # Wraps Feishu API HTTP calls
credentials/
  LarkTokenApi.credentials.ts    # Tenant Access Token
  LarkOAuth2Api.credentials.ts   # User Access Token (OAuth2)
__tests__/                # Jest test files
```

### Operation File Structure

Each file in `resource/<name>/` must export a default conforming to `ResourceOperation`:

```typescript
export default {
  name: 'Display Name',
  value: 'operationValue',   // matches OperationType enum
  order: 100,                // higher = shown first
  options: [ /* INodeProperties[] shown when this op is selected */ ],
  async call(this: IExecuteFunctions, index: number): Promise<IDataObject> {
    return RequestUtils.request.call(this, { method, url, qs, body });
  },
} as ResourceOperation;
```

### Output Types

The `execute()` in `Lark.node.ts` handles three output shapes returned by `call()`:
- **Default** (no `outputType`): single JSON output
- **`OutputType.Multiple`**: multiple output branches via `outputData: INodeExecutionData[][]`
- **`OutputType.Binary`**: binary file output via `binaryData` + optional `binaryPropertyName`

### i18n

Locale is controlled by `N8N_DEFAULT_LOCALE` env var (defaults to `zh`). All display strings go through `WORDING` exported from `nodes/help/wording/index.ts`, which reads from `nodes/help/i18n/locales/`.

## Known Issues

- `nodes/Lark/resource/Spreadsheet.reource.ts` — typo in filename (`.reource.ts`). Works at runtime because `ModuleLoadUtils` globs all `*.js` files, not just `*.resource.js`. Do not rename without checking for other references.
- `wiki_spaces/` operations are fully implemented but the resource is not registered — no `Wiki.resource.ts` exists and `ResourceWiki` is absent from `IWording`. See "Adding a New Resource" checklist above.
