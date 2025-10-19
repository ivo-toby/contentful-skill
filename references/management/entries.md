# Entries

Entries are instances of content types. They follow the locale structure and require explicit publishing.

## Create Entry

```typescript
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
```

## Create Entry with ID

```typescript
const entry = await client.entry.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    entryId: 'my-custom-id',
    contentTypeId: 'blogPost',
  },
  {
    fields: {
      title: {
        'en-US': 'My First Post',
      },
    },
  }
)
```

## Get Entry

```typescript
const entry = await client.entry.get({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: 'entry-id',
})
```

## Update Entry

**CRITICAL**: Always include `sys` for version locking:

```typescript
// Get current version
const entry = await client.entry.get({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: 'entry-id',
})

// Update with version locking
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
```

## Publish Entry

```typescript
const published = await client.entry.publish(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    entryId: entry.sys.id,
  },
  {
    sys: entry.sys,
  }
)
```

## Unpublish Entry

```typescript
await client.entry.unpublish({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: 'entry-id',
})
```

## Archive Entry

```typescript
await client.entry.archive({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: 'entry-id',
})
```

## Delete Entry

```typescript
// Must unpublish first if published
await client.entry.unpublish({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: 'entry-id',
})

// Then delete
await client.entry.delete({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: 'entry-id',
})
```

## Query Entries

```typescript
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

// Access results
console.log(entries.items) // Array of entries
console.log(entries.total) // Total count
console.log(entries.skip) // Current skip
console.log(entries.limit) // Current limit
```

## Query Parameters

Common query parameters (same as CDA, without `include` and `locale`):

```typescript
// Content type filter
{ content_type: 'blogPost' }

// Field queries
{ 'fields.title': 'Exact Title' }
{ 'fields.title[match]': 'search term' }
{ 'fields.title[ne]': 'Not This' }
{ 'fields.count[gt]': 10 }
{ 'fields.count[gte]': 10 }
{ 'fields.count[lt]': 100 }
{ 'fields.count[lte]': 100 }
{ 'fields.date[gte]': '2024-01-01' }

// Full-text search
{ query: 'search across all fields' }

// Sys field queries
{ 'sys.id': 'entry-id' }
{ 'sys.id[in]': 'id1,id2,id3' }
{ 'sys.createdAt[gte]': '2024-01-01' }

// Ordering
{ order: 'sys.createdAt' }
{ order: '-sys.createdAt' } // descending
{ order: 'fields.title' }

// Pagination
{ limit: 100 }
{ skip: 0 }

// Select fields
{ select: 'fields.title,fields.slug' }
```

## Links and References

### Entry Reference

Link to another entry:

```typescript
{
  fields: {
    author: {
      'en-US': {
        sys: {
          type: 'Link',
          linkType: 'Entry',
          id: 'author-entry-id',
        },
      },
    },
  }
}
```

### Asset Reference

Link to an asset:

```typescript
{
  fields: {
    image: {
      'en-US': {
        sys: {
          type: 'Link',
          linkType: 'Asset',
          id: 'asset-id',
        },
      },
    },
  }
}
```

### Array of References

Multiple links:

```typescript
{
  fields: {
    gallery: {
      'en-US': [
        { sys: { type: 'Link', linkType: 'Asset', id: 'image1' } },
        { sys: { type: 'Link', linkType: 'Asset', id: 'image2' } },
        { sys: { type: 'Link', linkType: 'Asset', id: 'image3' } },
      ],
    },
  }
}
```

### Related Posts

```typescript
{
  fields: {
    relatedPosts: {
      'en-US': [
        { sys: { type: 'Link', linkType: 'Entry', id: 'post1' } },
        { sys: { type: 'Link', linkType: 'Entry', id: 'post2' } },
      ],
    },
  }
}
```

## Localization

### Multiple Locales

```typescript
const entry = await client.entry.create(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    contentTypeId: 'blogPost',
  },
  {
    fields: {
      title: {
        'en-US': 'English Title',
        'de-DE': 'German Title',
        'fr-FR': 'French Title',
      },
      body: {
        'en-US': 'English content',
        'de-DE': 'German content',
        'fr-FR': 'French content',
      },
    },
  }
)
```

### Fallback Locale

If a locale is not provided, Contentful uses the fallback locale chain configured in the space settings.

## Complete Workflow Example

```typescript
// 1. Create entry
const entry = await client.entry.create(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    contentTypeId: 'blogPost',
  },
  {
    fields: {
      title: { 'en-US': 'New Post' },
      slug: { 'en-US': 'new-post' },
      body: { 'en-US': 'Content here' },
    },
  }
)

// 2. Publish
const published = await client.entry.publish(
  { spaceId: 'xxx', environmentId: 'master', entryId: entry.sys.id },
  { sys: entry.sys }
)

// 3. Update
const updated = await client.entry.update(
  { spaceId: 'xxx', environmentId: 'master', entryId: entry.sys.id },
  {
    sys: published.sys,
    fields: {
      title: { 'en-US': 'Updated Title' },
      slug: published.fields.slug,
      body: published.fields.body,
    },
  }
)

// 4. Publish again
const republished = await client.entry.publish(
  { spaceId: 'xxx', environmentId: 'master', entryId: entry.sys.id },
  { sys: updated.sys }
)

// 5. Unpublish
await client.entry.unpublish({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: entry.sys.id,
})

// 6. Delete
await client.entry.delete({
  spaceId: 'xxx',
  environmentId: 'master',
  entryId: entry.sys.id,
})
```

## Best Practices

1. **Always include sys** when updating (version locking prevents conflicts)
2. **Use locale structure** - All fields must be keyed by locale
3. **Publish explicitly** - Entries are drafts until published
4. **Keep existing fields** - When updating, include unchanged fields or they'll be removed
5. **Handle version conflicts** - Refetch and retry on 409 errors
6. **Use pagination** - Query with limit/skip for large result sets
7. **Filter by content type** - Always specify `content_type` in queries for better performance
8. **Use createWithId** - For predictable IDs and idempotent operations
