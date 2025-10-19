# Bulk Operations

Patterns for pagination, batch processing, and concurrent operations with the Management SDK.

## Pagination

### Basic Pagination

```typescript
const result = await client.entry.getMany({
  spaceId: 'xxx',
  environmentId: 'master',
  query: {
    content_type: 'blogPost',
    limit: 100,
    skip: 0,
  },
})

console.log(result.items) // Array of entries
console.log(result.total) // Total count
console.log(result.skip) // Current skip
console.log(result.limit) // Current limit
```

### Fetch All Entries

```typescript
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

// Usage
const entries = await getAllEntries(client, {
  spaceId: 'xxx',
  environmentId: 'master',
  contentTypeId: 'blogPost',
})
```

### Paginate with Progress

```typescript
async function getAllEntriesWithProgress(
  client,
  { spaceId, environmentId, contentTypeId },
  onProgress?: (current: number, total: number) => void
) {
  let allEntries = []
  let skip = 0
  const limit = 1000

  // Get total count first
  const firstResult = await client.entry.getMany({
    spaceId,
    environmentId,
    query: {
      content_type: contentTypeId,
      limit: 1,
      skip: 0,
    },
  })

  const total = firstResult.total

  while (skip < total) {
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
    skip += limit

    onProgress?.(allEntries.length, total)
  }

  return allEntries
}

// Usage
const entries = await getAllEntriesWithProgress(
  client,
  { spaceId: 'xxx', environmentId: 'master', contentTypeId: 'blogPost' },
  (current, total) => {
    console.log(`Progress: ${current}/${total} (${Math.round((current / total) * 100)}%)`)
  }
)
```

## Batch Publishing

### Basic Batch Publish

```typescript
async function batchPublish(entries, { spaceId, environmentId }) {
  const results = []

  for (const entry of entries) {
    try {
      const published = await client.entry.publish(
        { spaceId, environmentId, entryId: entry.sys.id },
        { sys: entry.sys }
      )
      results.push({ success: true, entry: published })
    } catch (error) {
      results.push({ success: false, entry, error })
    }
  }

  return results
}
```

### Concurrent Batch Publish

```typescript
async function batchPublishConcurrent(
  entries,
  { spaceId, environmentId },
  concurrency = 5
) {
  const chunks = []
  for (let i = 0; i < entries.length; i += concurrency) {
    chunks.push(entries.slice(i, i + concurrency))
  }

  const results = []

  for (const chunk of chunks) {
    const chunkResults = await Promise.allSettled(
      chunk.map(entry =>
        client.entry.publish(
          { spaceId, environmentId, entryId: entry.sys.id },
          { sys: entry.sys }
        )
      )
    )

    results.push(...chunkResults)
  }

  return results
}

// Usage
const entries = await getAllEntries(client, {
  spaceId: 'xxx',
  environmentId: 'master',
  contentTypeId: 'blogPost',
})

const results = await batchPublishConcurrent(
  entries,
  { spaceId: 'xxx', environmentId: 'master' },
  5 // 5 concurrent requests
)

const successful = results.filter(r => r.status === 'fulfilled')
const failed = results.filter(r => r.status === 'rejected')

console.log(`Published: ${successful.length}/${results.length}`)
console.log(`Failed: ${failed.length}`)
```

## Concurrency Control

### Using p-limit

```typescript
import pLimit from 'p-limit'

async function batchPublishWithLimit(entries, { spaceId, environmentId }, maxConcurrency = 5) {
  const limit = pLimit(maxConcurrency)

  const promises = entries.map(entry =>
    limit(() =>
      client.entry.publish(
        { spaceId, environmentId, entryId: entry.sys.id },
        { sys: entry.sys }
      )
    )
  )

  return Promise.allSettled(promises)
}
```

### Manual Queue

```typescript
async function processQueue<T, R>(
  items: T[],
  processor: (item: T) => Promise<R>,
  concurrency = 5
): Promise<R[]> {
  const results: R[] = []
  const executing: Promise<void>[] = []

  for (const item of items) {
    const promise = processor(item).then(result => {
      results.push(result)
      executing.splice(executing.indexOf(promise), 1)
    })

    executing.push(promise)

    if (executing.length >= concurrency) {
      await Promise.race(executing)
    }
  }

  await Promise.all(executing)
  return results
}

// Usage
const results = await processQueue(
  entries,
  async entry => {
    return client.entry.publish(
      { spaceId: 'xxx', environmentId: 'master', entryId: entry.sys.id },
      { sys: entry.sys }
    )
  },
  5
)
```

## Batch Updates

### Update Multiple Entries

```typescript
async function batchUpdate(
  entryIds: string[],
  updateFn: (entry: any) => any,
  { spaceId, environmentId },
  concurrency = 5
) {
  const limit = pLimit(concurrency)

  const promises = entryIds.map(entryId =>
    limit(async () => {
      try {
        // Get current version
        const entry = await client.entry.get({
          spaceId,
          environmentId,
          entryId,
        })

        // Apply update
        const updated = await client.entry.update(
          { spaceId, environmentId, entryId },
          {
            sys: entry.sys,
            fields: updateFn(entry.fields),
          }
        )

        return { success: true, entry: updated }
      } catch (error) {
        return { success: false, entryId, error }
      }
    })
  )

  return Promise.all(promises)
}

// Usage
const results = await batchUpdate(
  ['entry1', 'entry2', 'entry3'],
  fields => ({
    ...fields,
    updatedAt: {
      'en-US': new Date().toISOString(),
    },
  }),
  { spaceId: 'xxx', environmentId: 'master' },
  5
)
```

## Bulk Delete

### Delete Multiple Entries

```typescript
async function batchDelete(
  entryIds: string[],
  { spaceId, environmentId },
  concurrency = 5
) {
  const limit = pLimit(concurrency)

  const promises = entryIds.map(entryId =>
    limit(async () => {
      try {
        // Unpublish first if needed
        try {
          await client.entry.unpublish({
            spaceId,
            environmentId,
            entryId,
          })
        } catch (error) {
          // Ignore if already unpublished
        }

        // Delete
        await client.entry.delete({
          spaceId,
          environmentId,
          entryId,
        })

        return { success: true, entryId }
      } catch (error) {
        return { success: false, entryId, error }
      }
    })
  )

  return Promise.all(promises)
}
```

## Migration Patterns

### Clone Content Between Environments

```typescript
async function cloneEntries(
  contentTypeId: string,
  { sourceSpaceId, sourceEnvId, targetSpaceId, targetEnvId },
  concurrency = 5
) {
  // 1. Get all entries from source
  const sourceEntries = await getAllEntries(client, {
    spaceId: sourceSpaceId,
    environmentId: sourceEnvId,
    contentTypeId,
  })

  // 2. Create in target
  const limit = pLimit(concurrency)

  const promises = sourceEntries.map(sourceEntry =>
    limit(async () => {
      try {
        const created = await client.entry.createWithId(
          {
            spaceId: targetSpaceId,
            environmentId: targetEnvId,
            entryId: sourceEntry.sys.id,
            contentTypeId,
          },
          {
            fields: sourceEntry.fields,
          }
        )

        // Publish if source was published
        if (sourceEntry.sys.publishedVersion) {
          await client.entry.publish(
            {
              spaceId: targetSpaceId,
              environmentId: targetEnvId,
              entryId: created.sys.id,
            },
            { sys: created.sys }
          )
        }

        return { success: true, entryId: sourceEntry.sys.id }
      } catch (error) {
        return { success: false, entryId: sourceEntry.sys.id, error }
      }
    })
  )

  return Promise.all(promises)
}
```

### Bulk Field Update

```typescript
async function bulkFieldUpdate(
  contentTypeId: string,
  fieldId: string,
  updateValue: (currentValue: any) => any,
  { spaceId, environmentId },
  concurrency = 5
) {
  // Get all entries
  const entries = await getAllEntries(client, {
    spaceId,
    environmentId,
    contentTypeId,
  })

  const limit = pLimit(concurrency)

  const promises = entries.map(entry =>
    limit(async () => {
      try {
        const currentValue = entry.fields[fieldId]
        const newValue = updateValue(currentValue)

        const updated = await client.entry.update(
          { spaceId, environmentId, entryId: entry.sys.id },
          {
            sys: entry.sys,
            fields: {
              ...entry.fields,
              [fieldId]: newValue,
            },
          }
        )

        // Re-publish if was published
        if (entry.sys.publishedVersion) {
          await client.entry.publish(
            { spaceId, environmentId, entryId: updated.sys.id },
            { sys: updated.sys }
          )
        }

        return { success: true, entryId: entry.sys.id }
      } catch (error) {
        return { success: false, entryId: entry.sys.id, error }
      }
    })
  )

  return Promise.all(promises)
}

// Usage: Add prefix to all titles
const results = await bulkFieldUpdate(
  'blogPost',
  'title',
  currentValue => ({
    'en-US': `[Updated] ${currentValue['en-US']}`,
  }),
  { spaceId: 'xxx', environmentId: 'master' },
  5
)
```

## Progress Tracking

### With Progress Bar

```typescript
async function batchOperationWithProgress<T, R>(
  items: T[],
  operation: (item: T) => Promise<R>,
  concurrency = 5
): Promise<R[]> {
  const results: R[] = []
  let completed = 0

  const limit = pLimit(concurrency)

  const promises = items.map(item =>
    limit(async () => {
      const result = await operation(item)
      results.push(result)
      completed++

      // Update progress
      const percentage = Math.round((completed / items.length) * 100)
      console.log(`Progress: ${completed}/${items.length} (${percentage}%)`)

      return result
    })
  )

  await Promise.all(promises)
  return results
}
```

## Error Recovery

### Retry Failed Operations

```typescript
async function batchWithRetry<T, R>(
  items: T[],
  operation: (item: T) => Promise<R>,
  maxRetries = 3,
  concurrency = 5
): Promise<Array<{ success: boolean; item: T; result?: R; error?: any }>> {
  const limit = pLimit(concurrency)

  const promises = items.map(item =>
    limit(async () => {
      for (let attempt = 0; attempt < maxRetries; attempt++) {
        try {
          const result = await operation(item)
          return { success: true, item, result }
        } catch (error) {
          if (attempt === maxRetries - 1) {
            return { success: false, item, error }
          }
          // Wait before retry with exponential backoff
          await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, attempt)))
        }
      }
    })
  )

  return Promise.all(promises)
}
```

## Best Practices

1. **Use pagination** - Don't load all entries at once
2. **Control concurrency** - Limit concurrent requests (5-10 recommended)
3. **Handle errors gracefully** - Use Promise.allSettled for batch operations
4. **Respect rate limits** - SDK auto-throttles, but manual control helps
5. **Track progress** - Log progress for long-running operations
6. **Retry on failure** - Use exponential backoff for retries
7. **Batch by content type** - Query specific content types for better performance
8. **Use createWithId** - For idempotent operations when migrating
9. **Preserve publish state** - Re-publish after updating if needed
10. **Monitor memory** - Process large datasets in chunks

## Complete Example

```typescript
import pLimit from 'p-limit'

async function migrateAndPublish(
  contentTypeId: string,
  { spaceId, environmentId },
  transformer: (fields: any) => any
) {
  console.log('Fetching entries...')

  // 1. Get all entries
  const entries = await getAllEntriesWithProgress(
    client,
    { spaceId, environmentId, contentTypeId },
    (current, total) => {
      console.log(`Fetched: ${current}/${total}`)
    }
  )

  console.log(`Found ${entries.length} entries`)

  // 2. Update entries
  const limit = pLimit(5)
  let completed = 0

  const results = await Promise.allSettled(
    entries.map(entry =>
      limit(async () => {
        // Get fresh version
        const current = await client.entry.get({
          spaceId,
          environmentId,
          entryId: entry.sys.id,
        })

        // Transform fields
        const newFields = transformer(current.fields)

        // Update
        const updated = await client.entry.update(
          { spaceId, environmentId, entryId: current.sys.id },
          {
            sys: current.sys,
            fields: newFields,
          }
        )

        // Publish if was published
        if (current.sys.publishedVersion) {
          await client.entry.publish(
            { spaceId, environmentId, entryId: updated.sys.id },
            { sys: updated.sys }
          )
        }

        completed++
        console.log(`Progress: ${completed}/${entries.length}`)

        return updated
      })
    )
  )

  // 3. Report results
  const successful = results.filter(r => r.status === 'fulfilled')
  const failed = results.filter(r => r.status === 'rejected')

  console.log(`✓ Successful: ${successful.length}`)
  console.log(`✗ Failed: ${failed.length}`)

  if (failed.length > 0) {
    console.error('Failed entries:', failed)
  }

  return { successful, failed }
}
```
