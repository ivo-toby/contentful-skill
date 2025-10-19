# Management SDK Overview

Quick start guide for the Contentful Management JavaScript/TypeScript SDK (CMA).

## Installation

```bash
npm install contentful-management
```

## Quick Start

### Plain Client (Recommended)

The "plain" client provides flat API access to all CMA endpoints:

```typescript
import contentful from 'contentful-management'

// Basic client
const client = contentful.createClient(
  {
    accessToken: process.env.CONTENTFUL_MANAGEMENT_TOKEN,
  },
  { type: 'plain' }
)

// Scoped client (recommended for most use cases)
const scopedClient = contentful.createClient(
  {
    accessToken: process.env.CONTENTFUL_MANAGEMENT_TOKEN,
  },
  {
    type: 'plain',
    defaults: {
      spaceId: 'your-space-id',
      environmentId: 'master', // or environment alias
    },
  }
)
```

### Client Options

```typescript
contentful.createClient({
  accessToken: 'YOUR_TOKEN',
  host: 'api.contentful.com', // default
  uploadHost: 'upload.contentful.com', // for assets
  basePath: '', // for custom gateways
  headers: {}, // additional headers
  retryOnError: true, // auto-retry on 429/500
  retryLimit: 5,
  rateLimit: 'auto', // or number, or percentage string
  logHandler: (level, data) => {}, // custom logging
  requestLogger: (config) => {}, // request interceptor
  responseLogger: (response) => {}, // response interceptor
})
```

## Critical Patterns

### Version Locking

**ALWAYS pass sys when updating** - Contentful uses optimistic locking:

```typescript
// ✅ CORRECT
const entry = await client.entry.get({ spaceId, environmentId, entryId })
const updated = await client.entry.update(
  { spaceId, environmentId, entryId },
  {
    sys: entry.sys, // Version info
    fields: { ...entry.fields, title: { 'en-US': 'New' } },
  }
)

// ❌ WRONG - will fail with version mismatch
await client.entry.update(
  { spaceId, environmentId, entryId },
  {
    fields: { title: { 'en-US': 'New' } },
  }
)
```

### Locale Structure

Fields are **always** keyed by locale:

```typescript
// ✅ CORRECT
{
  fields: {
    title: {
      'en-US': 'English Title',
      'de-DE': 'German Title'
    }
  }
}

// ❌ WRONG
{
  fields: {
    title: 'English Title'
  }
}
```

### Publish Workflow

Content types, entries, and assets follow: **Create → Publish** or **Update → Publish**:

```typescript
// Create/update returns DRAFT
const draft = await client.entry.create(...)

// MUST publish explicitly
await client.entry.publish(
  { spaceId, environmentId, entryId: draft.sys.id },
  { sys: draft.sys }
)
```

## App SDK Integration

For Contentful Apps (no exposed management token needed):

```typescript
import { init } from '@contentful/app-sdk'
import contentful from 'contentful-management'

init(sdk => {
  const cma = contentful.createClient(
    { apiAdapter: sdk.cmaAdapter },
    {
      type: 'plain',
      defaults: {
        environmentId: sdk.ids.environmentAlias ?? sdk.ids.environment,
        spaceId: sdk.ids.space,
      },
    }
  )

  // Use cma as normal
})
```

## Topics

Detailed guides for specific operations:

- **[Environments](environments.md)** - Create, clone, and manage environments
- **[Content Types](content-types.md)** - Define and update content models with validations
- **[Entries](entries.md)** - Create, update, query, and publish entries
- **[Assets](assets.md)** - Upload, process, and manage media files
- **[Error Handling](error-handling.md)** - Handle rate limits, version conflicts, and validation errors
- **[Bulk Operations](bulk-operations.md)** - Pagination, batch processing, and concurrency control

## Best Practices

1. **Use scoped clients** - set spaceId/environmentId in defaults
2. **Always include sys** when updating (version locking)
3. **Remember locale structure** - all fields use `{ 'locale': value }`
4. **Publish explicitly** - nothing is live until published
5. **Handle version conflicts** - refetch and retry on 409
6. **Respect rate limits** - SDK auto-retries, monitor headers
7. **Poll async operations** - environment creation, asset processing
8. **Use environment aliases** - avoid hardcoding environment IDs

## Reference Documentation

- **API Docs**: [CMA Reference](https://www.contentful.com/developers/docs/references/content-management-api/)
- **SDK Docs**: [JS SDK Reference](https://contentful.github.io/contentful-management.js/)
