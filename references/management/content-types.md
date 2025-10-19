# Content Types

Content types define the structure of your content. They must be published to be usable.

## Get Content Type

```typescript
const contentType = await client.contentType.get({
  spaceId: 'xxx',
  environmentId: 'master',
  contentTypeId: 'blogPost',
})
```

## Create Content Type

```typescript
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

## Field Types

Common field types and their configurations:

```typescript
// Symbol (short text)
{
  id: 'title',
  name: 'Title',
  type: 'Symbol',
  required: true,
  localized: true,
}

// Text (long text)
{
  id: 'body',
  name: 'Body',
  type: 'Text',
  required: false,
  localized: true,
}

// Integer
{
  id: 'count',
  name: 'Count',
  type: 'Integer',
}

// Number (decimal)
{
  id: 'price',
  name: 'Price',
  type: 'Number',
}

// Boolean
{
  id: 'published',
  name: 'Published',
  type: 'Boolean',
}

// Date
{
  id: 'publishDate',
  name: 'Publish Date',
  type: 'Date',
}

// Location
{
  id: 'location',
  name: 'Location',
  type: 'Location',
}

// Object (JSON)
{
  id: 'metadata',
  name: 'Metadata',
  type: 'Object',
}

// Array
{
  id: 'tags',
  name: 'Tags',
  type: 'Array',
  items: { type: 'Symbol' },
}

// Link (single reference)
{
  id: 'author',
  name: 'Author',
  type: 'Link',
  linkType: 'Entry',
  validations: [
    { linkContentType: ['author', 'contributor'] }
  ],
}

// Array of Links (multiple references)
{
  id: 'relatedPosts',
  name: 'Related Posts',
  type: 'Array',
  items: {
    type: 'Link',
    linkType: 'Entry',
  },
  validations: [
    { linkContentType: ['blogPost', 'article'] }
  ],
}

// Asset Link
{
  id: 'image',
  name: 'Image',
  type: 'Link',
  linkType: 'Asset',
}

// Rich Text
{
  id: 'richBody',
  name: 'Rich Body',
  type: 'RichText',
  validations: [
    {
      nodes: {
        'embedded-entry-block': [
          { linkContentType: ['quote', 'callout'] }
        ],
      }
    }
  ],
}
```

## Validations

### Symbol/Text Validations

```typescript
// Unique values
{
  id: 'slug',
  type: 'Symbol',
  validations: [
    { unique: true }
  ]
}

// Regex pattern
{
  id: 'slug',
  type: 'Symbol',
  validations: [
    {
      regexp: {
        pattern: '^[a-z0-9-]+$',
        flags: null,
      }
    }
  ]
}

// Size constraints
{
  id: 'title',
  type: 'Symbol',
  validations: [
    { size: { min: 3, max: 100 } }
  ]
}

// Allowed values
{
  id: 'category',
  type: 'Symbol',
  validations: [
    { in: ['tech', 'design', 'business'] }
  ]
}

// Email format
{
  id: 'email',
  type: 'Symbol',
  validations: [
    { regexp: { pattern: '^\\S+@\\S+\\.\\S+$' } }
  ]
}
```

### Number Validations

```typescript
// Range
{
  id: 'rating',
  type: 'Integer',
  validations: [
    { range: { min: 1, max: 5 } }
  ]
}

// Allowed values
{
  id: 'quantity',
  type: 'Integer',
  validations: [
    { in: [1, 5, 10, 25, 50, 100] }
  ]
}
```

### Array Validations

```typescript
// Array size
{
  id: 'tags',
  type: 'Array',
  items: { type: 'Symbol' },
  validations: [
    { size: { min: 1, max: 10 } }
  ]
}

// Array of Links with content type restriction
{
  id: 'relatedPosts',
  type: 'Array',
  items: {
    type: 'Link',
    linkType: 'Entry',
  },
  validations: [
    { linkContentType: ['blogPost', 'article'] }
  ]
}
```

### Link Validations

```typescript
// Restrict to specific content types
{
  id: 'author',
  type: 'Link',
  linkType: 'Entry',
  validations: [
    { linkContentType: ['author', 'contributor'] }
  ]
}

// Asset file size
{
  id: 'image',
  type: 'Link',
  linkType: 'Asset',
  validations: [
    {
      assetFileSize: {
        min: 1024,
        max: 10485760, // 10MB
      }
    }
  ]
}

// Asset dimensions
{
  id: 'thumbnail',
  type: 'Link',
  linkType: 'Asset',
  validations: [
    {
      assetImageDimensions: {
        width: { min: 100, max: 2000 },
        height: { min: 100, max: 2000 },
      }
    }
  ]
}
```

### Rich Text Validations

```typescript
{
  id: 'richBody',
  type: 'RichText',
  validations: [
    {
      // Restrict embedded content types
      nodes: {
        'embedded-entry-block': [
          { linkContentType: ['quote', 'callout'] }
        ],
        'embedded-entry-inline': [
          { linkContentType: ['product'] }
        ],
        'embedded-asset-block': [
          { size: { min: 1, max: 5 } }
        ],
      },
    },
    {
      // Size validation
      size: { min: 10, max: 50000 }
    },
  ]
}
```

## Update Content Type

```typescript
// Get current version
const contentType = await client.contentType.get({
  spaceId: 'xxx',
  environmentId: 'master',
  contentTypeId: 'blogPost',
})

// Modify fields
const updatedFields = [
  ...contentType.fields,
  {
    id: 'newField',
    name: 'New Field',
    type: 'Symbol',
  }
]

// Update
const updated = await client.contentType.update(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    contentTypeId: 'blogPost',
  },
  {
    sys: contentType.sys,
    name: contentType.name,
    displayField: contentType.displayField,
    fields: updatedFields,
  }
)

// Publish
await client.contentType.publish(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    contentTypeId: 'blogPost',
  },
  {
    sys: updated.sys,
  }
)
```

## Delete Content Type

```typescript
// Unpublish first
await client.contentType.unpublish({
  spaceId: 'xxx',
  environmentId: 'master',
  contentTypeId: 'blogPost',
})

// Then delete
await client.contentType.delete({
  spaceId: 'xxx',
  environmentId: 'master',
  contentTypeId: 'blogPost',
})
```

## Best Practices

1. **Always publish** - Content types must be published to create entries
2. **Use descriptive IDs** - Field IDs are permanent, choose carefully
3. **Add validations early** - Harder to add constraints later with existing content
4. **Use linkContentType** - Restrict references to specific content types
5. **Consider localization** - Set `localized: true` for translatable fields
6. **Use displayField** - Makes content easier to identify in the UI
7. **Test in staging** - Create content types in a non-production environment first
