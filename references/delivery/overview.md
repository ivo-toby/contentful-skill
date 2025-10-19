# Delivery SDK Overview

Quick start guide for the Contentful Delivery JavaScript/TypeScript SDK (CDA).

## Installation

```bash
npm install contentful
```

## Quick Start

### Basic Client

```typescript
import * as contentful from 'contentful'

const client = contentful.createClient({
  space: 'your-space-id',
  accessToken: 'your-delivery-token',
})
```

### Client Options

```typescript
const client = contentful.createClient({
  space: 'your-space-id',
  accessToken: 'your-delivery-token',
  environment: 'master', // default: 'master'
  host: 'cdn.contentful.com', // default: 'cdn.contentful.com'
  basePath: '', // optional custom path
  insecure: false, // use http instead of https
  retryOnError: true, // retry on network errors
  logHandler: (level, data) => {}, // custom logging
  adapter: customHttpAdapter, // custom http adapter
  timeout: 30000, // request timeout in ms
})
```

### Preview Client

For fetching draft content:

```typescript
const previewClient = contentful.createClient({
  space: 'your-space-id',
  accessToken: 'your-preview-token',
  host: 'preview.contentful.com', // Use preview API
})
```

## Basic Usage

### Get Single Entry

```typescript
const entry = await client.getEntry('entry-id')

console.log(entry.sys.id)
console.log(entry.sys.contentType.sys.id)
console.log(entry.fields.title)
console.log(entry.fields.slug)
```

### Get Multiple Entries

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  limit: 10,
})

console.log(entries.items) // Array of entries
console.log(entries.total) // Total count
console.log(entries.skip) // Current skip
console.log(entries.limit) // Current limit
```

### Get Single Asset

```typescript
const asset = await client.getAsset('asset-id')

console.log(asset.fields.title)
console.log(asset.fields.file.url)
console.log(asset.fields.file.contentType)
console.log(asset.fields.file.details.size)
```

### Get Multiple Assets

```typescript
const assets = await client.getAssets({
  limit: 10,
  'fields.title[match]': 'hero',
})
```

### Get Content Type

```typescript
const contentType = await client.getContentType('blogPost')

console.log(contentType.name)
console.log(contentType.fields)
console.log(contentType.displayField)
```

### Get Space

```typescript
const space = await client.getSpace()

console.log(space.name)
console.log(space.locales)
```

## TypeScript Support

### Entry Skeleton

Define type-safe entry structure:

```typescript
import type { EntryFieldTypes } from 'contentful'

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    slug: EntryFieldTypes.Symbol
    body: EntryFieldTypes.RichText
    author: EntryFieldTypes.EntryLink<AuthorSkeleton>
    publishDate: EntryFieldTypes.Date
    tags: EntryFieldTypes.Array<EntryFieldTypes.Symbol>
    featured: EntryFieldTypes.Boolean
    heroImage: EntryFieldTypes.AssetLink
  }
}

type AuthorSkeleton = {
  contentTypeId: 'author'
  fields: {
    name: EntryFieldTypes.Text
    bio: EntryFieldTypes.Text
    photo: EntryFieldTypes.AssetLink
  }
}
```

### Typed Queries

```typescript
const entry = await client.getEntry<BlogPostSkeleton>('entry-id')

// TypeScript knows the fields
console.log(entry.fields.title) // string
console.log(entry.fields.publishDate) // string (ISO date)
console.log(entry.fields.featured) // boolean

const entries = await client.getEntries<BlogPostSkeleton>({
  content_type: 'blogPost',
  'fields.featured': true,
})

for (const entry of entries.items) {
  console.log(entry.fields.title) // Type-safe access
}
```

### Field Types

Common EntryFieldTypes:

```typescript
EntryFieldTypes.Text // string
EntryFieldTypes.Symbol // string
EntryFieldTypes.Integer // number
EntryFieldTypes.Number // number
EntryFieldTypes.Date // string (ISO 8601)
EntryFieldTypes.Boolean // boolean
EntryFieldTypes.Location // { lat: number; lon: number }
EntryFieldTypes.Object // Record<string, any>
EntryFieldTypes.RichText // Document (from @contentful/rich-text-types)
EntryFieldTypes.Array<T> // T[]
EntryFieldTypes.EntryLink<T> // Entry<T>
EntryFieldTypes.AssetLink // Asset
```

## Locale Modifiers

### With All Locales

Get all locales for an entry:

```typescript
type Locales = 'en-US' | 'de-DE' | 'fr-FR'

const entry = await client.withAllLocales
  .getEntry<BlogPostSkeleton, Locales>('entry-id')

// Fields are keyed by locale
console.log(entry.fields.title['en-US'])
console.log(entry.fields.title['de-DE'])
console.log(entry.fields.title['fr-FR'])
```

### Without Link Resolution

Skip resolving links:

```typescript
const entry = await client.withoutLinkResolution.getEntry('entry-id')

// Links are not resolved, sys.id only
console.log(entry.fields.author.sys.id)
```

### Without Unresolvable Links

Remove unresolvable links from response:

```typescript
const entry = await client.withoutUnresolvableLinks.getEntry('entry-id')
```

### Chaining Modifiers

```typescript
const entry = await client
  .withAllLocales
  .withoutLinkResolution
  .getEntry<BlogPostSkeleton, Locales>('entry-id')
```

## Sync API

Synchronize content for offline use:

```typescript
// Initial sync
const sync = await client.sync({ initial: true })

console.log(sync.entries) // All entries
console.log(sync.assets) // All assets
console.log(sync.deletedEntries) // Deleted entries
console.log(sync.deletedAssets) // Deleted assets
console.log(sync.nextSyncToken) // Token for next sync

// Subsequent sync
const nextSync = await client.sync({ nextSyncToken: sync.nextSyncToken })

console.log(nextSync.entries) // Updated entries
console.log(nextSync.assets) // Updated assets
console.log(nextSync.deletedEntries) // Newly deleted entries
console.log(nextSync.deletedAssets) // Newly deleted assets
```

## Error Handling

```typescript
try {
  const entry = await client.getEntry('entry-id')
} catch (error) {
  if (error.sys?.id === 'NotFound') {
    console.error('Entry not found')
  } else if (error.sys?.id === 'AccessTokenInvalid') {
    console.error('Invalid access token')
  } else {
    console.error('Error:', error.message)
  }
}
```

## Topics

Detailed guides for specific features:

- **[Querying](querying.md)** - Search parameters, filtering, operators, full-text search
- **[Includes & Links](includes-links.md)** - Link resolution, includes parameter, circular references
- **[Localization](localization.md)** - Locale handling, fallbacks, all locales
- **[Rich Text](rich-text.md)** - Rendering rich text fields, embedded entries and assets

## Best Practices

1. **Use TypeScript** - Define entry skeletons for type safety
2. **Use delivery tokens** - Never expose management tokens in client-side apps
3. **Cache responses** - Implement caching for better performance
4. **Use preview API** - For draft content and content previews
5. **Specify content_type** - Always filter by content type for better performance
6. **Use includes wisely** - Don't over-resolve links (max depth is 10)
7. **Handle errors** - Check for NotFound and AccessTokenInvalid
8. **Use sync API** - For offline-first applications
9. **Limit results** - Use limit/skip for pagination
10. **Use modifiers** - Chain modifiers for specific response shapes

## Reference Documentation

- **API Docs**: [CDA Reference](https://www.contentful.com/developers/docs/references/content-delivery-api/)
- **SDK Docs**: [contentful.js](https://contentful.github.io/contentful.js/contentful/latest/)
- **GitHub**: [contentful/contentful.js](https://github.com/contentful/contentful.js)
- **NPM**: [contentful](https://www.npmjs.com/package/contentful)
