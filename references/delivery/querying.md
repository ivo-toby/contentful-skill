# Querying

Comprehensive guide to querying entries and assets with the Delivery SDK.

## Basic Query

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
})
```

## Query Parameters

### Content Type

Filter by content type:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
})
```

### Limit and Skip

Pagination:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  limit: 10, // max: 1000
  skip: 0, // offset
})
```

### Order

Sort results:

```typescript
// Ascending
const entries = await client.getEntries({
  content_type: 'blogPost',
  order: 'fields.publishDate',
})

// Descending
const entries = await client.getEntries({
  content_type: 'blogPost',
  order: '-fields.publishDate',
})

// Multiple fields
const entries = await client.getEntries({
  content_type: 'blogPost',
  order: '-fields.publishDate,fields.title',
})

// System fields
const entries = await client.getEntries({
  order: 'sys.createdAt',
})
```

### Select

Retrieve specific fields:

```typescript
// Select specific fields
const entries = await client.getEntries({
  content_type: 'blogPost',
  select: 'fields.title,fields.slug,sys.id',
})

// Only sys fields
const entries = await client.getEntries({
  content_type: 'blogPost',
  select: 'sys',
})
```

## Field Queries

### Equality

```typescript
// Exact match
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.slug': 'hello-world',
})
```

### Inequality

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.slug[ne]': 'hello-world',
})
```

### In / Not In

```typescript
// In array
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.slug[in]': 'post-1,post-2,post-3',
})

// Not in array
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.slug[nin]': 'post-1,post-2',
})
```

### Existence

```typescript
// Field exists
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.author[exists]': true,
})

// Field doesn't exist
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.author[exists]': false,
})
```

### Comparison Operators

```typescript
// Greater than
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.viewCount[gt]': 1000,
})

// Greater than or equal
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.viewCount[gte]': 1000,
})

// Less than
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.viewCount[lt]': 1000,
})

// Less than or equal
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.viewCount[lte]': 1000,
})
```

### Date Ranges

```typescript
// After date
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.publishDate[gte]': '2024-01-01',
})

// Before date
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.publishDate[lte]': '2024-12-31',
})

// Date range
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.publishDate[gte]': '2024-01-01',
  'fields.publishDate[lte]': '2024-12-31',
})

// System date fields
const entries = await client.getEntries({
  'sys.createdAt[gte]': '2024-01-01T00:00:00Z',
})
```

## Text Search

### Full-Text Search

Search across all text and symbol fields:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  query: 'contentful',
})
```

### Full-Text Search on Specific Field

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.title[match]': 'contentful',
})
```

### Multiple Terms

```typescript
// AND logic (both terms must match)
const entries = await client.getEntries({
  content_type: 'blogPost',
  query: 'contentful cms',
})
```

## Location Queries

### Near

Find entries near a location:

```typescript
const entries = await client.getEntries({
  content_type: 'venue',
  'fields.location[near]': '40.7128,-74.0060', // lat,lon
})
```

### Within Bounding Box

Find entries within a geographic area:

```typescript
const entries = await client.getEntries({
  content_type: 'venue',
  'fields.location[within]': '40.7,-74.1,40.8,-73.9', // lat1,lon1,lat2,lon2
})
```

## Array Fields

### All

Array contains all specified values:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.tags[all]': 'tech,javascript',
})
```

### In

Array contains any of the specified values:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.tags[in]': 'tech,design,business',
})
```

## Link Queries

### Reference to Entry

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.author.sys.id': 'author-id',
})
```

### Multiple References

Maximum 2 levels of references:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.author.sys.id': 'author-id',
  'fields.category.sys.id': 'category-id',
})
```

### Link Content Type

Query by linked content type:

```typescript
const entries = await client.getEntries({
  'links_to_entry': 'entry-id',
})

const entries = await client.getEntries({
  'links_to_asset': 'asset-id',
})
```

## System Field Queries

### Entry ID

```typescript
const entries = await client.getEntries({
  'sys.id': 'entry-id',
})

const entries = await client.getEntries({
  'sys.id[in]': 'id1,id2,id3',
})
```

### Content Type

```typescript
const entries = await client.getEntries({
  'sys.contentType.sys.id': 'blogPost',
})
```

### Created/Updated Dates

```typescript
const entries = await client.getEntries({
  'sys.createdAt[gte]': '2024-01-01T00:00:00Z',
})

const entries = await client.getEntries({
  'sys.updatedAt[gte]': '2024-01-01T00:00:00Z',
})
```

### Revision

```typescript
const entries = await client.getEntries({
  'sys.revision[gte]': 1,
})
```

## Asset Queries

### By MIME Type

```typescript
const assets = await client.getAssets({
  'fields.file.contentType': 'image/png',
})

// Images only
const assets = await client.getAssets({
  'fields.file.contentType[in]': 'image/jpeg,image/png,image/gif',
})
```

### By File Size

```typescript
const assets = await client.getAssets({
  'fields.file.details.size[lt]': 1048576, // Less than 1MB
})
```

### By Dimensions

```typescript
const assets = await client.getAssets({
  'fields.file.details.image.width[gte]': 1920,
})

const assets = await client.getAssets({
  'fields.file.details.image.height[gte]': 1080,
})
```

### By Title

```typescript
const assets = await client.getAssets({
  'fields.title[match]': 'hero',
})
```

## Complex Queries

### Multiple Filters

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  'fields.featured': true,
  'fields.publishDate[gte]': '2024-01-01',
  'fields.tags[in]': 'tech,javascript',
  order: '-fields.publishDate',
  limit: 10,
})
```

### TypeScript Query

```typescript
type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    slug: EntryFieldTypes.Symbol
    featured: EntryFieldTypes.Boolean
    publishDate: EntryFieldTypes.Date
    viewCount: EntryFieldTypes.Integer
    tags: EntryFieldTypes.Array<EntryFieldTypes.Symbol>
  }
}

const entries = await client.getEntries<BlogPostSkeleton>({
  content_type: 'blogPost',
  'fields.featured': true,
  'fields.viewCount[gt]': 1000,
  'fields.publishDate[gte]': '2024-01-01',
  order: '-fields.publishDate',
  limit: 10,
  skip: 0,
})

// Type-safe access
for (const entry of entries.items) {
  console.log(entry.fields.title) // string
  console.log(entry.fields.viewCount) // number
  console.log(entry.fields.featured) // boolean
}
```

## Pagination Pattern

```typescript
async function getAllEntries(contentType: string) {
  let allEntries = []
  let skip = 0
  const limit = 1000

  while (true) {
    const result = await client.getEntries({
      content_type: contentType,
      limit,
      skip,
    })

    allEntries = allEntries.concat(result.items)

    if (result.items.length < limit) break
    skip += limit
  }

  return allEntries
}
```

## Search Best Practices

1. **Always specify content_type** - Improves performance significantly
2. **Use select** - Fetch only needed fields to reduce payload
3. **Limit results** - Use limit/skip for pagination (max 1000 per request)
4. **Use specific operators** - Prefer equality over match for better performance
5. **Index fields** - Use indexed fields for filtering when possible
6. **Avoid deep includes** - Keep include parameter low (max 10)
7. **Cache results** - Implement caching for frequently accessed queries
8. **Use sys fields** - Filter by sys.id, sys.createdAt for better performance
9. **Combine filters** - Multiple filters are AND-ed together
10. **Order wisely** - Order by indexed fields when possible

## Query Limits

- Maximum limit per request: 1000
- Maximum include depth: 10
- Maximum reference searches: 2 levels
- Default limit: 100

## Common Query Patterns

### Latest Posts

```typescript
const latest = await client.getEntries({
  content_type: 'blogPost',
  order: '-fields.publishDate',
  limit: 10,
})
```

### Featured Posts

```typescript
const featured = await client.getEntries({
  content_type: 'blogPost',
  'fields.featured': true,
  order: '-fields.publishDate',
})
```

### Posts by Author

```typescript
const posts = await client.getEntries({
  content_type: 'blogPost',
  'fields.author.sys.id': 'author-id',
  order: '-fields.publishDate',
})
```

### Search Posts

```typescript
const results = await client.getEntries({
  content_type: 'blogPost',
  query: searchTerm,
  limit: 20,
})
```

### Posts by Tag

```typescript
const posts = await client.getEntries({
  content_type: 'blogPost',
  'fields.tags[in]': 'javascript',
  order: '-fields.publishDate',
})
```

### Recent Images

```typescript
const images = await client.getAssets({
  'fields.file.contentType[in]': 'image/jpeg,image/png',
  order: '-sys.createdAt',
  limit: 20,
})
```
