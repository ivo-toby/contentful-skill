# Localization

Guide to handling locales, fallbacks, and multi-language content in the Delivery SDK.

## Locale Parameter

### Default Locale

By default, the SDK returns content in the space's default locale:

```typescript
const entry = await client.getEntry('entry-id')

// Fields are in default locale
console.log(entry.fields.title) // Default locale value
```

### Specific Locale

Request content in a specific locale:

```typescript
const entry = await client.getEntry('entry-id', {
  locale: 'de-DE',
})

// Fields are in German
console.log(entry.fields.title) // German title
```

### Query with Locale

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  locale: 'fr-FR',
})

// All entries in French
for (const entry of entries.items) {
  console.log(entry.fields.title) // French title
}
```

## All Locales

### With All Locales Modifier

Get content in all available locales:

```typescript
type Locales = 'en-US' | 'de-DE' | 'fr-FR'

const entry = await client.withAllLocales
  .getEntry<BlogPostSkeleton, Locales>('entry-id')

// Fields are keyed by locale
console.log(entry.fields.title['en-US']) // English
console.log(entry.fields.title['de-DE']) // German
console.log(entry.fields.title['fr-FR']) // French
```

### Structure with All Locales

```typescript
{
  sys: { ... },
  fields: {
    title: {
      'en-US': 'Hello World',
      'de-DE': 'Hallo Welt',
      'fr-FR': 'Bonjour le monde'
    },
    body: {
      'en-US': 'This is content',
      'de-DE': 'Dies ist Inhalt',
      'fr-FR': 'Ceci est le contenu'
    }
  }
}
```

### Query All Locales

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  locale: '*', // Wildcard for all locales
})

// Fields keyed by locale
for (const entry of entries.items) {
  console.log(entry.fields.title['en-US'])
  console.log(entry.fields.title['de-DE'])
}
```

## Locale Fallbacks

### Fallback Chain

If content is missing in requested locale, Contentful uses the fallback chain:

```typescript
// Space has fallback: de-DE → en-US
const entry = await client.getEntry('entry-id', {
  locale: 'de-DE',
})

// If German is missing, English is returned
console.log(entry.fields.title) // Falls back to en-US if de-DE is empty
```

### Optional Fallback

Disable fallback to get only the requested locale:

```typescript
const entry = await client.getEntry('entry-id', {
  locale: 'de-DE',
  // Note: SDK doesn't have a built-in way to disable fallbacks
  // Use withAllLocales and check specific locale
})

const entry = await client.withAllLocales.getEntry('entry-id')

// Check if German exists
if (entry.fields.title['de-DE']) {
  console.log(entry.fields.title['de-DE'])
} else {
  console.log('No German translation available')
}
```

## TypeScript with Locales

### Typed Locales

Define locale type for type safety:

```typescript
type Locales = 'en-US' | 'de-DE' | 'fr-FR' | 'ja-JP'

type BlogPostSkeleton = {
  contentTypeId: 'blogPost'
  fields: {
    title: EntryFieldTypes.Text
    body: EntryFieldTypes.Text
  }
}

const entry = await client.withAllLocales
  .getEntry<BlogPostSkeleton, Locales>('entry-id')

// Type-safe locale access
console.log(entry.fields.title['en-US']) // ✅ Valid
console.log(entry.fields.title['de-DE']) // ✅ Valid
// console.log(entry.fields.title['es-ES']) // ❌ TypeScript error
```

### Default Locale Type

Without locale type, fields are strings:

```typescript
const entry = await client.getEntry<BlogPostSkeleton>('entry-id')

console.log(entry.fields.title) // string
```

## Localized Fields

### Check Field Localization

In content types, fields can be localized or not:

```typescript
// Localized field (different per locale)
{
  id: 'title',
  name: 'Title',
  type: 'Symbol',
  localized: true // ← Can be different per locale
}

// Non-localized field (same across locales)
{
  id: 'sku',
  name: 'SKU',
  type: 'Symbol',
  localized: false // ← Same value in all locales
}
```

### Behavior

```typescript
const entry = await client.withAllLocales.getEntry('product-id')

// Localized field - different per locale
console.log(entry.fields.title['en-US']) // "Blue Shirt"
console.log(entry.fields.title['de-DE']) // "Blaues Hemd"

// Non-localized field - same value in default locale only
console.log(entry.fields.sku['en-US']) // "SHIRT-001"
console.log(entry.fields.sku['de-DE']) // undefined (uses en-US value)
```

## Space Locales

### Get Available Locales

```typescript
const space = await client.getSpace()

console.log(space.locales)
/*
[
  {
    code: 'en-US',
    name: 'English (United States)',
    default: true,
    fallbackCode: null
  },
  {
    code: 'de-DE',
    name: 'German (Germany)',
    default: false,
    fallbackCode: 'en-US'
  },
  {
    code: 'fr-FR',
    name: 'French (France)',
    default: false,
    fallbackCode: 'en-US'
  }
]
*/
```

### Default Locale

```typescript
const space = await client.getSpace()
const defaultLocale = space.locales.find(locale => locale.default)

console.log(defaultLocale.code) // 'en-US'
```

## Querying by Locale

### Filter by Localized Field

```typescript
// Query in specific locale
const entries = await client.getEntries({
  content_type: 'blogPost',
  locale: 'de-DE',
  'fields.title[match]': 'Hallo',
})

// Searches German titles
```

### Full-Text Search

Full-text search searches all locales by default:

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  query: 'contentful', // Searches all locales
})
```

### Specific Locale Search

```typescript
const entries = await client.getEntries({
  content_type: 'blogPost',
  locale: 'fr-FR',
  'fields.title[match]': 'bonjour', // Searches only French
})
```

## Patterns

### Locale Switcher

```typescript
async function getEntryInLocale(entryId: string, locale: string) {
  return client.getEntry(entryId, { locale })
}

// Switch between locales
const enEntry = await getEntryInLocale('post-id', 'en-US')
const deEntry = await getEntryInLocale('post-id', 'de-DE')
const frEntry = await getEntryInLocale('post-id', 'fr-FR')
```

### Check Translation Exists

```typescript
const entry = await client.withAllLocales.getEntry('entry-id')

const locales = ['en-US', 'de-DE', 'fr-FR', 'ja-JP']

for (const locale of locales) {
  if (entry.fields.title[locale]) {
    console.log(`${locale}: ${entry.fields.title[locale]}`)
  } else {
    console.log(`${locale}: No translation`)
  }
}
```

### Translation Progress

```typescript
async function getTranslationProgress(contentType: string) {
  const entries = await client.getEntries({
    content_type: contentType,
    locale: '*',
  })

  const space = await client.getSpace()
  const locales = space.locales.map(l => l.code)

  const progress = {}

  for (const locale of locales) {
    let translated = 0
    for (const entry of entries.items) {
      if (entry.fields.title?.[locale]) {
        translated++
      }
    }
    progress[locale] = {
      translated,
      total: entries.items.length,
      percentage: Math.round((translated / entries.items.length) * 100),
    }
  }

  return progress
}

const progress = await getTranslationProgress('blogPost')
console.log(progress)
/*
{
  'en-US': { translated: 100, total: 100, percentage: 100 },
  'de-DE': { translated: 75, total: 100, percentage: 75 },
  'fr-FR': { translated: 50, total: 100, percentage: 50 }
}
*/
```

### Multilingual Site

```typescript
type SupportedLocale = 'en-US' | 'de-DE' | 'fr-FR'

async function getLocalizedContent(
  contentType: string,
  userLocale: SupportedLocale
) {
  return client.getEntries<BlogPostSkeleton>({
    content_type: contentType,
    locale: userLocale,
  })
}

// Usage
const userLocale = 'de-DE' // From user settings/browser
const content = await getLocalizedContent('blogPost', userLocale)
```

## Assets and Localization

### Localized Asset Files

Assets can have different files per locale:

```typescript
const asset = await client.withAllLocales.getAsset('asset-id')

console.log(asset.fields.file['en-US'].url) // English PDF
console.log(asset.fields.file['de-DE'].url) // German PDF
console.log(asset.fields.file['fr-FR'].url) // French PDF

console.log(asset.fields.title['en-US']) // "User Guide"
console.log(asset.fields.title['de-DE']) // "Benutzerhandbuch"
```

### Asset Query by Locale

```typescript
const assets = await client.getAssets({
  locale: 'de-DE',
  'fields.title[match]': 'Anleitung',
})
```

## Best Practices

1. **Use type-safe locales** - Define Locales type for TypeScript
2. **Handle missing translations** - Check for undefined when using withAllLocales
3. **Respect fallback chain** - Understand your space's locale fallback configuration
4. **Use default locale** - Don't specify locale for default language content
5. **Cache by locale** - Implement separate caches per locale
6. **Query with locale** - Filter by locale when searching localized content
7. **Check localized flag** - Know which fields are localized in your content model
8. **Use wildcard sparingly** - locale: '*' increases response size significantly
9. **Consider fallbacks** - Plan for missing translations in your UI
10. **Get space locales** - Fetch available locales from space for dynamic locale switching

## Common Patterns

### Single Entry, Multiple Locales

```typescript
const entry = await client.withAllLocales.getEntry('entry-id')

const translations = {
  en: entry.fields.title['en-US'],
  de: entry.fields.title['de-DE'],
  fr: entry.fields.title['fr-FR'],
}
```

### Dynamic Locale Selection

```typescript
function getPreferredLocale(): string {
  // Check user settings
  const userLocale = getUserLocale()
  if (userLocale) return userLocale

  // Check browser locale
  if (typeof navigator !== 'undefined') {
    return navigator.language || 'en-US'
  }

  return 'en-US'
}

const locale = getPreferredLocale()
const content = await client.getEntries({
  content_type: 'blogPost',
  locale,
})
```

### Translation-Complete Filter

```typescript
// Get only fully translated entries
const entries = await client.getEntries({
  content_type: 'blogPost',
  locale: 'de-DE',
})

// Filter out entries with fallback content
const fullyTranslated = entries.items.filter(entry => {
  // Check sys.locale or manually verify
  return entry.sys.locale === 'de-DE'
})
```
