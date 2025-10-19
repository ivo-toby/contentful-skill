---
name: contentful-management
description: Contentful Management API SDK patterns for TypeScript/JavaScript. Use when working with CMA operations like creating/updating entries, assets, content types, managing spaces/environments, or any CMA-related tasks. Covers authentication, error handling, rate limiting, and common patterns.
---

# Contentful Management SDK

Comprehensive guide for the Contentful Management JavaScript/TypeScript SDK. Use this for creating, reading, updating content types, entries, assets, and managing Contentful spaces/environments.

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

## Core Patterns

### Environments

**Always specify environment** - defaults to 'master' if omitted:

```typescript
// Get environment
const env = await client.environment.get({
  spaceId: 'xxx',
  environmentId: 'staging',
})

// Create environment (clones from master or specified source)
const newEnv = await client.environment.create(
  { spaceId: 'xxx' },
  {
    name: 'New Environment',
  }
)

// Create from specific source
const newEnv = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'feature-branch',
    sourceEnvironmentId: 'staging', // clone from staging
  },
  {
    name: 'Feature Branch',
  }
)

// Poll until ready
let env = await client.environment.get({ spaceId: 'xxx', environmentId: 'new-env' })
while (env.sys.status.sys.id !== 'ready') {
  await new Promise(resolve => setTimeout(resolve, 1000))
  env = await client.environment.get({ spaceId: 'xxx', environmentId: 'new-env' })
}
```

### Content Types

```typescript
// Get content type
const contentType = await client.contentType.get({
  spaceId: 'xxx',
  environmentId: 'master',
  contentTypeId: 'blogPost',
})

// Create content type
const newCT = await client.contentType.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    contentTypeId: 'myContentType',
  },
  {
    name: 'My Content Type',
    displayField: 'title',
    fields: [
      {
        id: 'title',
        name: 'Title',
        type: 'Symbol',
        required: true,
        localized: false,
      },
      {
        id: 'body',
        name: 'Body',
        type: 'Text',
        required: false,
        localized: true,
      },
      {
        id: 'slug',
        name: 'Slug',
        type: 'Symbol',
        required: true,
        validations: [
          {
            unique: true,
          },
          {
            regexp: {
              pattern: '^[a-z0-9-]+$',
            },
          },
        ],
      },
    ],
  }
)

// CRITICAL: Publish after creation
const published = await client.contentType.publish(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    contentTypeId: 'myContentType',
  },
  {
    sys: newCT.sys,
  }
)
```

### Entries

```typescript
// Create entry
const entry = await client.entry.create(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    contentTypeId: 'blogPost',
  },
  {
    fields: {
      title: {
        'en-US': 'My First Post',
      },
      body: {
        'en-US': 'This is the body content',
      },
    },
  }
)

// Update entry (version-locked)
const updated = await client.entry.update(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    entryId: entry.sys.id,
  },
  {
    sys: entry.sys, // MUST include sys for version locking
    fields: {
      title: {
        'en-US': 'Updated Title',
      },
      body: entry.fields.body, // Keep existing fields
    },
  }
)

// Publish
const published = await client.entry.publish(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    entryId: entry.sys.id,
  },
  {
    sys: updated.sys,
  }
)

// Query entries
const entries = await client.entry.getMany({
  spaceId: 'xxx',
  environmentId: 'master',
  query: {
    content_type: 'blogPost',
    'fields.title[match]': 'search term',
    limit: 100,
    skip: 0,
    order: '-sys.createdAt',
  },
})
```

### Assets

```typescript
// Create asset (three-step process)
const asset = await client.asset.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: 'my-image',
  },
  {
    fields: {
      title: {
        'en-US': 'My Image',
      },
      file: {
        'en-US': {
          contentType: 'image/png',
          fileName: 'image.png',
          upload: 'https://example.com/image.png', // or upload via Upload API
        },
      },
    },
  }
)

// CRITICAL: Process asset
const processed = await client.asset.processForAllLocales(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: asset.sys.id,
  },
  {
    sys: asset.sys,
  }
)

// Wait for processing (poll)
let current = await client.asset.get({
  spaceId: 'xxx',
  environmentId: 'master',
  assetId: asset.sys.id,
})
while (!current.fields.file?.['en-US']?.url) {
  await new Promise(resolve => setTimeout(resolve, 500))
  current = await client.asset.get({
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: asset.sys.id,
  })
}

// CRITICAL: Publish
await client.asset.publish(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: current.sys.id,
  },
  {
    sys: current.sys,
  }
)
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

## Error Handling

```typescript
import { AxiosError } from 'axios'

try {
  await client.entry.update(...)
} catch (error) {
  if (error instanceof AxiosError) {
    const status = error.response?.status
    const data = error.response?.data

    if (status === 429) {
      // Rate limited - SDK auto-retries by default
      const resetTime = error.response?.headers['x-contentful-ratelimit-reset']
      console.log(`Rate limited. Reset in ${resetTime}s`)
    }

    if (status === 409) {
      // Version mismatch - refetch and retry
      console.error('Version conflict:', data.sys?.version)
    }

    if (status === 422) {
      // Validation error
      console.error('Validation:', data.details?.errors)
    }
  }
  throw error
}
```

## Rate Limiting

**Default**: 7 requests/second, auto-retry enabled

```typescript
// Configure rate limiting
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    rateLimit: 6, // requests per second
    // or
    rateLimit: 'auto', // based on plan
    // or
    rateLimit: '80%', // 80% of plan limit
  }
)

// Monitor via headers
const response = await client.entry.get(...)
console.log(response.sys.id)
// Headers available on error responses:
// x-contentful-ratelimit-second-limit
// x-contentful-ratelimit-second-remaining
// x-contentful-ratelimit-reset
```

## Common Validations

```typescript
// Content type validations
{
  fields: [
    {
      id: 'slug',
      type: 'Symbol',
      validations: [
        { unique: true },
        { regexp: { pattern: '^[a-z0-9-]+$' } },
        { size: { min: 3, max: 100 } }
      ]
    },
    {
      id: 'email',
      type: 'Symbol',
      validations: [
        { regexp: { pattern: '^\\S+@\\S+\\.\\S+$' } }
      ]
    },
    {
      id: 'tags',
      type: 'Array',
      items: { type: 'Symbol' },
      validations: [
        { size: { min: 1, max: 10 } }
      ]
    },
    {
      id: 'category',
      type: 'Symbol',
      validations: [
        { in: ['tech', 'design', 'business'] }
      ]
    },
    {
      id: 'relatedPosts',
      type: 'Array',
      items: { type: 'Link', linkType: 'Entry' },
      validations: [
        { linkContentType: ['blogPost', 'article'] }
      ]
    }
  ]
}
```

## Links and References

```typescript
// Entry reference
{
  fields: {
    author: {
      'en-US': {
        sys: {
          type: 'Link',
          linkType: 'Entry',
          id: 'author-entry-id'
        }
      }
    }
  }
}

// Asset reference
{
  fields: {
    image: {
      'en-US': {
        sys: {
          type: 'Link',
          linkType: 'Asset',
          id: 'asset-id'
        }
      }
    }
  }
}

// Array of references
{
  fields: {
    gallery: {
      'en-US': [
        { sys: { type: 'Link', linkType: 'Asset', id: 'image1' } },
        { sys: { type: 'Link', linkType: 'Asset', id: 'image2' } }
      ]
    }
  }
}
```

## Bulk Operations

```typescript
// Query all entries with pagination
async function getAllEntries(client, { spaceId, environmentId, contentTypeId }) {
  let allEntries = []
  let skip = 0
  const limit = 1000 // max per request

  while (true) {
    const result = await client.entry.getMany({
      spaceId,
      environmentId,
      query: {
        content_type: contentTypeId,
        limit,
        skip,
      },
    })

    allEntries = allEntries.concat(result.items)

    if (result.items.length < limit) break
    skip += limit
  }

  return allEntries
}

// Bulk publish with concurrency control
async function bulkPublish(entries, { spaceId, environmentId }, concurrency = 5) {
  const chunks = []
  for (let i = 0; i < entries.length; i += concurrency) {
    chunks.push(entries.slice(i, i + concurrency))
  }

  for (const chunk of chunks) {
    await Promise.all(
      chunk.map(entry =>
        client.entry.publish(
          { spaceId, environmentId, entryId: entry.sys.id },
          { sys: entry.sys }
        )
      )
    )
  }
}
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

## Reference Documentation

For complete API reference including all methods and parameters:
- **API Docs**: [CMA Reference](https://www.contentful.com/developers/docs/references/content-management-api/)
- **SDK Docs**: [JS SDK Reference](https://contentful.github.io/contentful-management.js/)
- **Search Parameters**: Use CDA search params (without `include` and `locale`)

## Best Practices

1. **Use scoped clients** - set spaceId/environmentId in defaults
2. **Always include sys** when updating (version locking)
3. **Remember locale structure** - all fields use `{ 'locale': value }`
4. **Publish explicitly** - nothing is live until published
5. **Handle version conflicts** - refetch and retry on 409
6. **Respect rate limits** - SDK auto-retries, monitor headers
7. **Poll async operations** - environment creation, asset processing
8. **Use environment aliases** - avoid hardcoding environment IDs
9. **Batch operations** - use concurrency control for bulk updates
10. **Error handling** - check status codes and validation details
