# Assets

Assets are media files (images, videos, documents). They require a three-step process: create, process, publish.

## Asset Workflow

The complete workflow for assets is:

1. **Create** - Create asset with file URL or upload token
2. **Process** - Process the asset to generate URLs and metadata
3. **Publish** - Make asset publicly available

## Create Asset

### From External URL

```typescript
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
          upload: 'https://example.com/image.png',
        },
      },
    },
  }
)
```

### From Upload

For direct file uploads, use the Upload API first:

```typescript
// 1. Upload file
const uploadResponse = await fetch('https://upload.contentful.com/spaces/xxx/uploads', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/octet-stream',
  },
  body: fileBuffer,
})
const upload = await uploadResponse.json()

// 2. Create asset with upload ID
const asset = await client.asset.create(
  {
    spaceId: 'xxx',
    environmentId: 'master',
  },
  {
    fields: {
      title: {
        'en-US': 'Uploaded File',
      },
      file: {
        'en-US': {
          contentType: 'image/png',
          fileName: 'file.png',
          uploadFrom: {
            sys: {
              type: 'Link',
              linkType: 'Upload',
              id: upload.sys.id,
            },
          },
        },
      },
    },
  }
)
```

## Process Asset

**CRITICAL**: After creating, process the asset to generate URLs:

```typescript
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
```

### Process Single Locale

```typescript
const processed = await client.asset.processForLocale(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: asset.sys.id,
  },
  {
    sys: asset.sys,
  },
  'en-US'
)
```

## Wait for Processing

Poll until the asset URL is available:

```typescript
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

console.log('Asset ready:', current.fields.file['en-US'].url)
```

## Publish Asset

**CRITICAL**: Publish to make the asset publicly available:

```typescript
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

## Complete Asset Workflow

```typescript
// 1. Create
const asset = await client.asset.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: 'hero-image',
  },
  {
    fields: {
      title: {
        'en-US': 'Hero Image',
      },
      description: {
        'en-US': 'Main hero image for homepage',
      },
      file: {
        'en-US': {
          contentType: 'image/png',
          fileName: 'hero.png',
          upload: 'https://example.com/hero.png',
        },
      },
    },
  }
)

// 2. Process
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

// 3. Wait for processing
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

// 4. Publish
const published = await client.asset.publish(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: current.sys.id,
  },
  {
    sys: current.sys,
  }
)

console.log('Asset published:', published.fields.file['en-US'].url)
```

## Update Asset

```typescript
// Get current version
const asset = await client.asset.get({
  spaceId: 'xxx',
  environmentId: 'master',
  assetId: 'asset-id',
})

// Update metadata
const updated = await client.asset.update(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: asset.sys.id,
  },
  {
    sys: asset.sys,
    fields: {
      title: {
        'en-US': 'Updated Title',
      },
      description: {
        'en-US': 'Updated description',
      },
      file: asset.fields.file, // Keep existing file
    },
  }
)

// Publish changes
await client.asset.publish(
  {
    spaceId: 'xxx',
    environmentId: 'master',
    assetId: asset.sys.id,
  },
  {
    sys: updated.sys,
  }
)
```

## Get Asset

```typescript
const asset = await client.asset.get({
  spaceId: 'xxx',
  environmentId: 'master',
  assetId: 'asset-id',
})

// Access file details
const file = asset.fields.file['en-US']
console.log(file.url)
console.log(file.contentType)
console.log(file.fileName)
console.log(file.details.size)
console.log(file.details.image?.width)
console.log(file.details.image?.height)
```

## Query Assets

```typescript
const assets = await client.asset.getMany({
  spaceId: 'xxx',
  environmentId: 'master',
  query: {
    'fields.title[match]': 'search term',
    'sys.createdAt[gte]': '2024-01-01',
    order: '-sys.createdAt',
    limit: 100,
    skip: 0,
  },
})
```

## Delete Asset

```typescript
// Unpublish first if published
await client.asset.unpublish({
  spaceId: 'xxx',
  environmentId: 'master',
  assetId: 'asset-id',
})

// Then delete
await client.asset.delete({
  spaceId: 'xxx',
  environmentId: 'master',
  assetId: 'asset-id',
})
```

## Archive Asset

```typescript
await client.asset.archive({
  spaceId: 'xxx',
  environmentId: 'master',
  assetId: 'asset-id',
})

// Unarchive
await client.asset.unarchive({
  spaceId: 'xxx',
  environmentId: 'master',
  assetId: 'asset-id',
})
```

## Content Types

Common content types:

```typescript
// Images
'image/png'
'image/jpeg'
'image/gif'
'image/webp'
'image/svg+xml'

// Videos
'video/mp4'
'video/webm'
'video/quicktime'

// Documents
'application/pdf'
'application/msword'
'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
'application/vnd.ms-excel'
'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'

// Other
'application/json'
'text/plain'
'application/zip'
```

## Localized Assets

Different files per locale:

```typescript
const asset = await client.asset.create(
  {
    spaceId: 'xxx',
    environmentId: 'master',
  },
  {
    fields: {
      title: {
        'en-US': 'English Version',
        'de-DE': 'German Version',
      },
      file: {
        'en-US': {
          contentType: 'application/pdf',
          fileName: 'guide-en.pdf',
          upload: 'https://example.com/guide-en.pdf',
        },
        'de-DE': {
          contentType: 'application/pdf',
          fileName: 'guide-de.pdf',
          upload: 'https://example.com/guide-de.pdf',
        },
      },
    },
  }
)

// Process all locales
await client.asset.processForAllLocales(
  { spaceId: 'xxx', environmentId: 'master', assetId: asset.sys.id },
  { sys: asset.sys }
)
```

## Image Transformation

Published assets support URL-based transformations:

```typescript
const url = asset.fields.file['en-US'].url

// Resize
`${url}?w=300&h=200`

// Format conversion
`${url}?fm=webp`

// Quality
`${url}?q=80`

// Fit options
`${url}?fit=pad&w=300&h=200&bg=rgb:ffffff`
`${url}?fit=fill&w=300&h=200`
`${url}?fit=scale&w=300`
`${url}?fit=crop&w=300&h=200&f=faces`

// Multiple parameters
`${url}?w=600&h=400&fit=fill&fm=webp&q=80`
```

## Best Practices

1. **Follow the three-step workflow** - Create, process, publish
2. **Always process** - Assets won't have URLs without processing
3. **Poll for completion** - Wait for URL to be available before publishing
4. **Use descriptive titles** - Makes assets easier to find
5. **Add descriptions** - Helpful for accessibility and SEO
6. **Use appropriate content types** - Ensures proper handling
7. **Consider localization** - Provide localized versions when needed
8. **Optimize before upload** - Compress images/videos to reduce storage
9. **Use CDN transformations** - Transform images on-the-fly instead of uploading multiple versions
10. **Version locking applies** - Always pass sys when updating
