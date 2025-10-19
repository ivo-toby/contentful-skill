# Includes & Links

Guide to link resolution, includes parameter, and handling references in the Delivery SDK.

## Link Structure

Links in Contentful reference other entries or assets:

```typescript
// Entry link field
{
  sys: {
    type: 'Link',
    linkType: 'Entry',
    id: 'entry-id'
  }
}

// Asset link field
{
  sys: {
    type: 'Link',
    linkType: 'Asset',
    id: 'asset-id'
  }
}
```

## Includes Parameter

### Default Behavior

By default, the SDK resolves one level of links:

```typescript
const entry = await client.getEntry('blog-post-id')

// Linked entries are resolved
console.log(entry.fields.author.fields.name)
console.log(entry.fields.heroImage.fields.file.url)
```

### Include Depth

Control link resolution depth (0-10):

```typescript
// No link resolution
const entries = await client.getEntries({
  content_type: 'blogPost',
  include: 0,
})
// Links are NOT resolved, only sys.id available
console.log(entries.items[0].fields.author.sys.id)

// Default (1 level)
const entries = await client.getEntries({
  content_type: 'blogPost',
  include: 1, // or omit
})
// First level resolved
console.log(entries.items[0].fields.author.fields.name)

// Deep resolution (2+ levels)
const entries = await client.getEntries({
  content_type: 'blogPost',
  include: 2,
})
// Author and author's linked entries resolved
console.log(entries.items[0].fields.author.fields.company.fields.name)
```

### Maximum Depth

Maximum include depth is 10:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  include: 10, // Maximum
})
```

## Resolution Patterns

### Single Reference

```typescript
type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    author: EntryFieldTypes.EntryLink<AuthorSkeleton>
  }
}

type AuthorSkeleton = {
  contentTypeId: 'author'
  fields: {
    name: EntryFieldTypes.Text
    email: EntryFieldTypes.Symbol
  }
}

const entry = await client.getEntry<BlogPostSkeleton>('post-id')

// Access linked author
console.log(entry.fields.author.fields.name)
console.log(entry.fields.author.fields.email)
console.log(entry.fields.author.sys.id)
```

### Array of References

```typescript
type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    relatedPosts: EntryFieldTypes.Array<
      EntryFieldTypes.EntryLink<BlogPostSkeleton>
    >
  }
}

const entry = await client.getEntry<BlogPostSkeleton>('post-id')

// Access array of linked entries
for (const related of entry.fields.relatedPosts) {
  console.log(related.fields.title)
  console.log(related.sys.id)
}
```

### Asset References

```typescript
type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    heroImage: EntryFieldTypes.AssetLink
    gallery: EntryFieldTypes.Array<EntryFieldTypes.AssetLink>
  }
}

const entry = await client.getEntry<BlogPostSkeleton>('post-id')

// Single asset
console.log(entry.fields.heroImage.fields.file.url)
console.log(entry.fields.heroImage.fields.title)

// Array of assets
for (const image of entry.fields.gallery) {
  console.log(image.fields.file.url)
  console.log(image.fields.file.details.size)
}
```

## Without Link Resolution

### Using Modifier

Skip link resolution entirely:

```typescript
const entry = await client.withoutLinkResolution.getEntry('post-id')

// Links are not resolved
console.log(entry.fields.author) // { sys: { type: 'Link', ... } }
console.log(entry.fields.author.sys.id) // Only sys available
```

### Manual Resolution

Resolve links manually when needed:

```typescript
// Get entry without resolving
const entry = await client.withoutLinkResolution.getEntry('post-id')

// Manually resolve author
const authorId = entry.fields.author.sys.id
const author = await client.getEntry(authorId)

console.log(author.fields.name)
```

## Unresolvable Links

### Handling Missing Links

Links may be unresolvable if:
- Linked entry/asset was deleted
- Linked entry is not published (in Delivery API)
- Insufficient permissions

```typescript
const entry = await client.getEntry('post-id')

// Check if link is resolved
if (entry.fields.author.sys.type === 'Link') {
  console.log('Author link is unresolved')
} else {
  console.log(entry.fields.author.fields.name)
}
```

### Without Unresolvable Links

Remove unresolvable links from response:

```typescript
const entry = await client.withoutUnresolvableLinks.getEntry('post-id')

// Unresolvable links are removed
// entry.fields.author will be undefined if unresolvable
if (entry.fields.author) {
  console.log(entry.fields.author.fields.name)
} else {
  console.log('No author available')
}
```

## Circular References

### Handling Cycles

Contentful prevents infinite resolution of circular references:

```typescript
type AuthorSkeleton = {
  contentTypeId: 'author'
  fields: {
    name: EntryFieldTypes.Text
    posts: EntryFieldTypes.Array<EntryFieldTypes.EntryLink<BlogPostSkeleton>>
  }
}

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    author: EntryFieldTypes.EntryLink<AuthorSkeleton>
  }
}

// Author → Posts → Author creates a cycle
// SDK resolves up to include depth, then stops
const entry = await client.getEntry<BlogPostSkeleton>('post-id', {
  include: 2,
})

console.log(entry.fields.author.fields.name) // Resolved
console.log(entry.fields.author.fields.posts[0].fields.title) // Resolved
// entry.fields.author.fields.posts[0].fields.author might not be fully resolved
```

## Includes Object

### Response Structure

The response includes a flat list of all resolved entities:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  include: 2,
})

// All resolved entries
console.log(entries.items) // Main results

// Linked entries (not in items)
console.log(entries.includes?.Entry) // Array of linked entries

// Linked assets
console.log(entries.includes?.Asset) // Array of linked assets
```

### Accessing Includes

```typescript
const result = await client.getEntries({
  content_type: 'blogPost',
})

// Get all unique authors from includes
const authors = result.includes?.Entry?.filter(
  entry => entry.sys.contentType.sys.id === 'author'
)

// Get all images from includes
const images = result.includes?.Asset?.filter(
  asset => asset.fields.file.contentType.startsWith('image/')
)
```

## Performance Considerations

### Over-Resolution

Avoid deep includes for better performance:

```typescript
// ❌ BAD - Resolves too much data
const entries = await client.getEntries({
  content_type: 'blogPost',
  include: 10,
})

// ✅ GOOD - Resolves only what's needed
const entries = await client.getEntries({
  content_type: 'blogPost',
  include: 2,
})
```

### Selective Resolution

Resolve links only when needed:

```typescript
// List view - no links needed
const entries = await client.getEntries({
  content_type: 'blogPost',
  select: 'fields.title,fields.slug,sys.id',
  include: 0,
})

// Detail view - resolve author and images
const entry = await client.getEntry('post-id', {
  include: 2,
})
```

## Link Queries

### Query by Linked Entry

```typescript
// Find all posts by specific author
const posts = await client.getEntries({
  content_type: 'blogPost',
  'fields.author.sys.id': 'author-id',
})
```

### Query by Multiple Links

Maximum 2 reference levels:

```typescript
// Posts by author AND category
const posts = await client.getEntries({
  content_type: 'blogPost',
  'fields.author.sys.id': 'author-id',
  'fields.category.sys.id': 'category-id',
})
```

### Links to Entry

Find all entries linking to a specific entry:

```typescript
const entries = await client.getEntries({
  links_to_entry: 'entry-id',
})
```

### Links to Asset

Find all entries using a specific asset:

```typescript
const entries = await client.getEntries({
  links_to_asset: 'asset-id',
})
```

## TypeScript Patterns

### Typed Links

```typescript
type ProductSkeleton = {
  contentTypeId: 'product'
  fields: {
    name: EntryFieldTypes.Text
    category: EntryFieldTypes.EntryLink<CategorySkeleton>
    images: EntryFieldTypes.Array<EntryFieldTypes.AssetLink>
    relatedProducts: EntryFieldTypes.Array<
      EntryFieldTypes.EntryLink<ProductSkeleton>
    >
  }
}

type CategorySkeleton = {
  contentTypeId: 'category'
  fields: {
    name: EntryFieldTypes.Text
    slug: EntryFieldTypes.Symbol
  }
}

const product = await client.getEntry<ProductSkeleton>('product-id')

// Type-safe access to linked content
console.log(product.fields.category.fields.name) // string
console.log(product.fields.images[0].fields.file.url) // string

for (const related of product.fields.relatedProducts) {
  console.log(related.fields.name) // string
}
```

### Optional Links

Handle potentially missing links:

```typescript
type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    author?: EntryFieldTypes.EntryLink<AuthorSkeleton>
  }
}

const entry = await client.getEntry<BlogPostSkeleton>('post-id')

// Safe access
if (entry.fields.author) {
  console.log(entry.fields.author.fields.name)
}
```

## Best Practices

1. **Use appropriate include depth** - Don't over-resolve (2-3 levels max recommended)
2. **Handle unresolvable links** - Check for Link type or use withoutUnresolvableLinks
3. **Use withoutLinkResolution** - For list views where links aren't needed
4. **Query by links** - Filter entries by linked content (max 2 levels)
5. **Type your links** - Use EntryFieldTypes.EntryLink<T> for type safety
6. **Check circular references** - Be aware of cycles in your content model
7. **Use includes object** - Access all resolved entities from response
8. **Cache resolved content** - Store frequently accessed linked content
9. **Select fields wisely** - Use select to reduce payload when links aren't needed
10. **Monitor performance** - Deep includes significantly increase response size

## Common Patterns

### Post with Author and Category

```typescript
const post = await client.getEntry({
  'entry-id',
  include: 1,
})

console.log(post.fields.title)
console.log(post.fields.author.fields.name)
console.log(post.fields.category.fields.name)
console.log(post.fields.heroImage.fields.file.url)
```

### List without Links

```typescript
const posts = await client.getEntries({
  content_type: 'blogPost',
  select: 'fields.title,fields.slug,sys.id',
  include: 0,
})
```

### Deep Navigation

```typescript
const post = await client.getEntry('post-id', {
  include: 3,
})

// Post → Author → Company → Logo
console.log(post.fields.author.fields.company.fields.logo.fields.file.url)
```

### Related Content

```typescript
const post = await client.getEntry<BlogPostSkeleton>('post-id', {
  include: 2,
})

// Show related posts
for (const related of post.fields.relatedPosts) {
  console.log(related.fields.title)
  console.log(related.fields.author.fields.name) // Include depth 2
}
```
