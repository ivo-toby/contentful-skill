# Rich Text

Guide to handling and rendering rich text fields with the Delivery SDK.

## Overview

Rich text fields store structured content with formatting, headings, lists, links, and embedded entries/assets.

## Rich Text Structure

Rich text uses the `@contentful/rich-text-types` format:

```typescript
import type { Document } from '@contentful/rich-text-types'

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    body: EntryFieldTypes.RichText // Document type
  }
}
```

## Document Structure

```typescript
{
  nodeType: 'document',
  data: {},
  content: [
    {
      nodeType: 'paragraph',
      data: {},
      content: [
        {
          nodeType: 'text',
          value: 'Hello world',
          marks: [],
          data: {}
        }
      ]
    }
  ]
}
```

## Rendering Rich Text

### Using @contentful/rich-text-react-renderer

Install the React renderer:

```bash
npm install @contentful/rich-text-react-renderer
```

Basic rendering:

```tsx
import { documentToReactComponents } from '@contentful/rich-text-react-renderer'
import type { Document } from '@contentful/rich-text-types'

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    body: EntryFieldTypes.RichText
  }
}

function BlogPost({ entry }: { entry: Entry<BlogPostSkeleton> }) {
  return (
    <article>
      <h1>{entry.fields.title}</h1>
      <div>{documentToReactComponents(entry.fields.body)}</div>
    </article>
  )
}
```

### Using @contentful/rich-text-html-renderer

Install the HTML renderer:

```bash
npm install @contentful/rich-text-html-renderer
```

Basic rendering:

```typescript
import { documentToHtmlString } from '@contentful/rich-text-html-renderer'

const entry = await client.getEntry<BlogPostSkeleton>('entry-id')
const html = documentToHtmlString(entry.fields.body)

console.log(html)
// "<p>Hello world</p>"
```

## Custom Rendering Options

### React Components

```tsx
import { documentToReactComponents } from '@contentful/rich-text-react-renderer'
import { BLOCKS, INLINES, MARKS } from '@contentful/rich-text-types'

const options = {
  renderMark: {
    [MARKS.BOLD]: (text) => <strong className="font-bold">{text}</strong>,
    [MARKS.ITALIC]: (text) => <em className="italic">{text}</em>,
    [MARKS.CODE]: (text) => <code className="font-mono">{text}</code>,
  },
  renderNode: {
    [BLOCKS.PARAGRAPH]: (node, children) => (
      <p className="my-4">{children}</p>
    ),
    [BLOCKS.HEADING_1]: (node, children) => (
      <h1 className="text-4xl font-bold my-6">{children}</h1>
    ),
    [BLOCKS.HEADING_2]: (node, children) => (
      <h2 className="text-3xl font-bold my-5">{children}</h2>
    ),
    [BLOCKS.HEADING_3]: (node, children) => (
      <h3 className="text-2xl font-bold my-4">{children}</h3>
    ),
    [BLOCKS.UL_LIST]: (node, children) => (
      <ul className="list-disc ml-6 my-4">{children}</ul>
    ),
    [BLOCKS.OL_LIST]: (node, children) => (
      <ol className="list-decimal ml-6 my-4">{children}</ol>
    ),
    [BLOCKS.QUOTE]: (node, children) => (
      <blockquote className="border-l-4 pl-4 italic my-4">
        {children}
      </blockquote>
    ),
    [BLOCKS.HR]: () => <hr className="my-8 border-gray-300" />,
  },
}

function BlogPost({ entry }: { entry: Entry<BlogPostSkeleton> }) {
  return (
    <div>
      {documentToReactComponents(entry.fields.body, options)}
    </div>
  )
}
```

### HTML String Options

```typescript
import { documentToHtmlString } from '@contentful/rich-text-html-renderer'
import { BLOCKS, INLINES } from '@contentful/rich-text-types'

const options = {
  renderNode: {
    [BLOCKS.PARAGRAPH]: (node, next) =>
      `<p class="my-4">${next(node.content)}</p>`,
    [BLOCKS.HEADING_1]: (node, next) =>
      `<h1 class="text-4xl">${next(node.content)}</h1>`,
    [BLOCKS.QUOTE]: (node, next) =>
      `<blockquote class="italic">${next(node.content)}</blockquote>`,
  },
}

const html = documentToHtmlString(entry.fields.body, options)
```

## Embedded Content

### Embedded Entries

```tsx
import { BLOCKS } from '@contentful/rich-text-types'

const options = {
  renderNode: {
    [BLOCKS.EMBEDDED_ENTRY]: (node) => {
      const entry = node.data.target

      if (entry.sys.contentType.sys.id === 'videoEmbed') {
        return (
          <div className="video-container">
            <iframe
              src={entry.fields.url}
              title={entry.fields.title}
              allowFullScreen
            />
          </div>
        )
      }

      if (entry.sys.contentType.sys.id === 'codeBlock') {
        return (
          <pre className="bg-gray-100 p-4 rounded">
            <code>{entry.fields.code}</code>
          </pre>
        )
      }

      return null
    },
  },
}
```

### Embedded Assets

```tsx
import { BLOCKS } from '@contentful/rich-text-types'

const options = {
  renderNode: {
    [BLOCKS.EMBEDDED_ASSET]: (node) => {
      const asset = node.data.target

      if (asset.fields.file.contentType.startsWith('image/')) {
        return (
          <img
            src={asset.fields.file.url}
            alt={asset.fields.title}
            width={asset.fields.file.details.image.width}
            height={asset.fields.file.details.image.height}
          />
        )
      }

      if (asset.fields.file.contentType === 'video/mp4') {
        return (
          <video controls>
            <source src={asset.fields.file.url} type="video/mp4" />
          </video>
        )
      }

      return (
        <a href={asset.fields.file.url} download>
          {asset.fields.title}
        </a>
      )
    },
  },
}
```

### Inline Entries

```tsx
import { INLINES } from '@contentful/rich-text-types'

const options = {
  renderNode: {
    [INLINES.EMBEDDED_ENTRY]: (node) => {
      const entry = node.data.target

      if (entry.sys.contentType.sys.id === 'product') {
        return (
          <a href={`/products/${entry.fields.slug}`} className="product-link">
            {entry.fields.name}
          </a>
        )
      }

      return <span>{entry.fields.name}</span>
    },
  },
}
```

## Hyperlinks

### Entry Links

```tsx
import { INLINES } from '@contentful/rich-text-types'
import Link from 'next/link'

const options = {
  renderNode: {
    [INLINES.ENTRY_HYPERLINK]: (node, children) => {
      const entry = node.data.target

      if (entry.sys.contentType.sys.id === 'blogPost') {
        return (
          <Link href={`/blog/${entry.fields.slug}`}>
            {children}
          </Link>
        )
      }

      return <span>{children}</span>
    },
  },
}
```

### Asset Links

```tsx
import { INLINES } from '@contentful/rich-text-types'

const options = {
  renderNode: {
    [INLINES.ASSET_HYPERLINK]: (node, children) => {
      const asset = node.data.target
      return (
        <a href={asset.fields.file.url} download>
          {children}
        </a>
      )
    },
  },
}
```

### External Links

```tsx
import { INLINES } from '@contentful/rich-text-types'

const options = {
  renderNode: {
    [INLINES.HYPERLINK]: (node, children) => {
      return (
        <a
          href={node.data.uri}
          target="_blank"
          rel="noopener noreferrer"
          className="text-blue-600 underline"
        >
          {children}
        </a>
      )
    },
  },
}
```

## Node Types

### Block Types

```typescript
import { BLOCKS } from '@contentful/rich-text-types'

BLOCKS.DOCUMENT          // Root document node
BLOCKS.PARAGRAPH         // Paragraph
BLOCKS.HEADING_1         // H1
BLOCKS.HEADING_2         // H2
BLOCKS.HEADING_3         // H3
BLOCKS.HEADING_4         // H4
BLOCKS.HEADING_5         // H5
BLOCKS.HEADING_6         // H6
BLOCKS.UL_LIST           // Unordered list
BLOCKS.OL_LIST           // Ordered list
BLOCKS.LIST_ITEM         // List item
BLOCKS.QUOTE             // Blockquote
BLOCKS.HR                // Horizontal rule
BLOCKS.EMBEDDED_ENTRY    // Embedded entry block
BLOCKS.EMBEDDED_ASSET    // Embedded asset block
BLOCKS.TABLE             // Table
BLOCKS.TABLE_ROW         // Table row
BLOCKS.TABLE_CELL        // Table cell
BLOCKS.TABLE_HEADER_CELL // Table header cell
```

### Inline Types

```typescript
import { INLINES } from '@contentful/rich-text-types'

INLINES.HYPERLINK         // External link
INLINES.ENTRY_HYPERLINK   // Link to entry
INLINES.ASSET_HYPERLINK   // Link to asset
INLINES.EMBEDDED_ENTRY    // Inline embedded entry
```

### Mark Types

```typescript
import { MARKS } from '@contentful/rich-text-types'

MARKS.BOLD               // Bold text
MARKS.ITALIC             // Italic text
MARKS.UNDERLINE          // Underlined text
MARKS.CODE               // Code/monospace text
MARKS.SUPERSCRIPT        // Superscript
MARKS.SUBSCRIPT          // Subscript
```

## TypeScript Types

### Typed Rich Text Field

```typescript
import type { Document } from '@contentful/rich-text-types'
import type { EntryFieldTypes, Entry } from 'contentful'

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    body: EntryFieldTypes.RichText
  }
}

const entry = await client.getEntry<BlogPostSkeleton>('entry-id')

// entry.fields.body is type Document
const richText: Document = entry.fields.body
```

### Typed Embedded Content

```typescript
type VideoEmbedSkeleton = {
  contentTypeId: 'videoEmbed'
  fields: {
    title: EntryFieldTypes.Text
    url: EntryFieldTypes.Symbol
  }
}

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    body: EntryFieldTypes.RichText
  }
}

const entry = await client.getEntry<BlogPostSkeleton>('entry-id', {
  include: 10, // Resolve embedded entries
})

const options = {
  renderNode: {
    [BLOCKS.EMBEDDED_ENTRY]: (node) => {
      const entry = node.data.target as Entry<VideoEmbedSkeleton>

      if (entry.sys.contentType.sys.id === 'videoEmbed') {
        return <video src={entry.fields.url} />
      }
    },
  },
}
```

## Tables

### Rendering Tables

```tsx
import { BLOCKS } from '@contentful/rich-text-types'

const options = {
  renderNode: {
    [BLOCKS.TABLE]: (node, children) => (
      <table className="min-w-full border">{children}</table>
    ),
    [BLOCKS.TABLE_ROW]: (node, children) => (
      <tr className="border-b">{children}</tr>
    ),
    [BLOCKS.TABLE_HEADER_CELL]: (node, children) => (
      <th className="border px-4 py-2 font-bold">{children}</th>
    ),
    [BLOCKS.TABLE_CELL]: (node, children) => (
      <td className="border px-4 py-2">{children}</td>
    ),
  },
}
```

## Best Practices

1. **Always include embedded content** - Use include: 10 for rich text with embedded entries/assets
2. **Handle missing content** - Check if embedded entries/assets exist before rendering
3. **Use TypeScript** - Type your entry skeletons for type-safe access
4. **Custom rendering** - Implement custom components for better control
5. **Sanitize HTML** - If rendering to HTML string, sanitize output
6. **Optimize images** - Use Contentful's image API for embedded assets
7. **Handle all node types** - Provide fallbacks for unknown node types
8. **Test deeply nested content** - Ensure rendering works with complex structures
9. **Cache rendered output** - Cache HTML/React output for performance
10. **Accessibility** - Ensure rendered content meets accessibility standards

## Complete Example

```tsx
import { Entry } from 'contentful'
import { documentToReactComponents } from '@contentful/rich-text-react-renderer'
import { BLOCKS, INLINES, MARKS } from '@contentful/rich-text-types'
import Image from 'next/image'
import Link from 'next/link'

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    body: EntryFieldTypes.RichText
  }
}

const renderOptions = {
  renderMark: {
    [MARKS.BOLD]: (text) => <strong>{text}</strong>,
    [MARKS.ITALIC]: (text) => <em>{text}</em>,
    [MARKS.CODE]: (text) => <code className="bg-gray-100 px-1">{text}</code>,
  },
  renderNode: {
    [BLOCKS.PARAGRAPH]: (node, children) => <p className="my-4">{children}</p>,
    [BLOCKS.HEADING_1]: (node, children) => <h1 className="text-4xl font-bold my-6">{children}</h1>,
    [BLOCKS.HEADING_2]: (node, children) => <h2 className="text-3xl font-bold my-5">{children}</h2>,
    [BLOCKS.QUOTE]: (node, children) => (
      <blockquote className="border-l-4 border-gray-300 pl-4 italic my-4">
        {children}
      </blockquote>
    ),
    [BLOCKS.EMBEDDED_ASSET]: (node) => {
      const asset = node.data.target
      if (!asset) return null

      if (asset.fields.file.contentType.startsWith('image/')) {
        return (
          <Image
            src={`https:${asset.fields.file.url}`}
            alt={asset.fields.title || ''}
            width={asset.fields.file.details.image.width}
            height={asset.fields.file.details.image.height}
            className="my-6 rounded-lg"
          />
        )
      }

      return null
    },
    [BLOCKS.EMBEDDED_ENTRY]: (node) => {
      const entry = node.data.target
      if (!entry) return null

      // Custom rendering based on content type
      return <div className="my-6 p-4 bg-gray-50 rounded">{entry.fields.title}</div>
    },
    [INLINES.HYPERLINK]: (node, children) => (
      <a
        href={node.data.uri}
        target="_blank"
        rel="noopener noreferrer"
        className="text-blue-600 hover:underline"
      >
        {children}
      </a>
    ),
    [INLINES.ENTRY_HYPERLINK]: (node, children) => {
      const entry = node.data.target
      if (!entry) return <span>{children}</span>

      return (
        <Link href={`/blog/${entry.fields.slug}`} className="text-blue-600 hover:underline">
          {children}
        </Link>
      )
    },
  },
}

export function BlogPostContent({ entry }: { entry: Entry<BlogPostSkeleton> }) {
  return (
    <article className="prose lg:prose-xl">
      <h1>{entry.fields.title}</h1>
      {documentToReactComponents(entry.fields.body, renderOptions)}
    </article>
  )
}
```
