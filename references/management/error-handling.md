# Error Handling

Comprehensive guide to handling errors, rate limits, and version conflicts in the Management SDK.

## Error Structure

The SDK uses Axios and throws AxiosError:

```typescript
import { AxiosError } from 'axios'

try {
  await client.entry.update(...)
} catch (error) {
  if (error instanceof AxiosError) {
    const status = error.response?.status
    const data = error.response?.data

    console.error('Status:', status)
    console.error('Error data:', data)
  }
  throw error
}
```

## Common Error Codes

### 400 Bad Request

Invalid request format or parameters:

```typescript
try {
  await client.entry.create(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 400) {
    console.error('Bad request:', error.response.data.message)
    console.error('Details:', error.response.data.details)
  }
}
```

### 401 Unauthorized

Invalid or expired access token:

```typescript
try {
  await client.entry.get(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 401) {
    console.error('Authentication failed - check access token')
    // Refresh token or re-authenticate
  }
}
```

### 403 Forbidden

Insufficient permissions:

```typescript
try {
  await client.contentType.delete(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 403) {
    console.error('Permission denied')
    console.error('Required scope:', error.response.data.details?.reasons)
  }
}
```

### 404 Not Found

Resource doesn't exist:

```typescript
try {
  await client.entry.get({ spaceId, environmentId, entryId })
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 404) {
    console.error('Entry not found:', entryId)
    // Handle missing resource
  }
}
```

### 409 Version Mismatch

Version conflict due to concurrent updates:

```typescript
try {
  await client.entry.update(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 409) {
    console.error('Version conflict')
    console.error('Current version:', error.response.data.sys?.version)

    // Refetch and retry
    const latest = await client.entry.get({ spaceId, environmentId, entryId })
    const updated = await client.entry.update(
      { spaceId, environmentId, entryId },
      {
        sys: latest.sys, // Use latest version
        fields: {
          ...latest.fields,
          // Apply your changes
        },
      }
    )
  }
}
```

### 422 Validation Error

Content validation failed:

```typescript
try {
  await client.entry.create(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 422) {
    const errors = error.response.data.details?.errors

    for (const err of errors) {
      console.error('Field:', err.path)
      console.error('Error:', err.name)
      console.error('Details:', err.details)
    }

    // Example errors:
    // - required: Field is required but missing
    // - unique: Value must be unique but isn't
    // - regexp: Value doesn't match pattern
    // - size: Value length out of range
  }
}
```

### 429 Rate Limited

Too many requests:

```typescript
try {
  await client.entry.create(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 429) {
    const resetTime = error.response.headers['x-contentful-ratelimit-reset']
    const limit = error.response.headers['x-contentful-ratelimit-second-limit']
    const remaining = error.response.headers['x-contentful-ratelimit-second-remaining']

    console.log(`Rate limited. Reset in ${resetTime}s`)
    console.log(`Limit: ${limit}/s, Remaining: ${remaining}`)

    // SDK auto-retries by default
    // Manual retry with exponential backoff:
    await new Promise(resolve => setTimeout(resolve, resetTime * 1000))
    // ... retry request
  }
}
```

### 500 Server Error

Internal server error:

```typescript
try {
  await client.entry.get(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 500) {
    console.error('Server error - retry later')
    // SDK auto-retries by default
  }
}
```

### 502/503 Service Unavailable

Temporary service issues:

```typescript
try {
  await client.entry.get(...)
} catch (error) {
  if (error instanceof AxiosError) {
    const status = error.response?.status
    if (status === 502 || status === 503) {
      console.error('Service temporarily unavailable')
      // SDK auto-retries by default
    }
  }
}
```

## Rate Limiting

### Default Behavior

The SDK automatically handles rate limiting:

```typescript
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    retryOnError: true, // Default: true
    retryLimit: 5, // Default: 5
    rateLimit: 'auto', // Default: 'auto'
  }
)
```

### Configure Rate Limiting

```typescript
// Requests per second
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    rateLimit: 6, // 6 requests/second
  }
)

// Percentage of plan limit
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    rateLimit: '80%', // 80% of plan limit
  }
)

// Auto (based on plan)
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    rateLimit: 'auto', // Automatically detect from plan
  }
)
```

### Monitor Rate Limits

```typescript
try {
  await client.entry.create(...)
} catch (error) {
  if (error instanceof AxiosError) {
    const headers = error.response?.headers

    console.log('Limit:', headers['x-contentful-ratelimit-second-limit'])
    console.log('Remaining:', headers['x-contentful-ratelimit-second-remaining'])
    console.log('Reset:', headers['x-contentful-ratelimit-reset'])
  }
}
```

### Manual Rate Limiting

For custom control:

```typescript
import pLimit from 'p-limit'

const limit = pLimit(5) // 5 concurrent requests

const promises = entries.map(entry =>
  limit(() =>
    client.entry.publish(
      { spaceId, environmentId, entryId: entry.sys.id },
      { sys: entry.sys }
    )
  )
)

await Promise.all(promises)
```

## Version Conflict Handling

### Automatic Retry

```typescript
async function updateEntryWithRetry(entryId, updates, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      // Get latest version
      const entry = await client.entry.get({
        spaceId: 'xxx',
        environmentId: 'master',
        entryId,
      })

      // Apply updates
      const updated = await client.entry.update(
        { spaceId: 'xxx', environmentId: 'master', entryId },
        {
          sys: entry.sys,
          fields: {
            ...entry.fields,
            ...updates,
          },
        }
      )

      return updated
    } catch (error) {
      if (error instanceof AxiosError && error.response?.status === 409) {
        if (attempt < maxRetries - 1) {
          console.log(`Version conflict, retrying (${attempt + 1}/${maxRetries})...`)
          await new Promise(resolve => setTimeout(resolve, 1000 * (attempt + 1)))
          continue
        }
      }
      throw error
    }
  }
}
```

### Merge Strategy

```typescript
async function updateWithMerge(entryId, fieldUpdates) {
  const entry = await client.entry.get({
    spaceId: 'xxx',
    environmentId: 'master',
    entryId,
  })

  // Merge updates with existing fields
  const mergedFields = { ...entry.fields }
  for (const [key, value] of Object.entries(fieldUpdates)) {
    mergedFields[key] = {
      ...mergedFields[key],
      ...value,
    }
  }

  try {
    return await client.entry.update(
      { spaceId: 'xxx', environmentId: 'master', entryId },
      {
        sys: entry.sys,
        fields: mergedFields,
      }
    )
  } catch (error) {
    if (error instanceof AxiosError && error.response?.status === 409) {
      // Recursive retry with fresh data
      return updateWithMerge(entryId, fieldUpdates)
    }
    throw error
  }
}
```

## Validation Error Handling

```typescript
try {
  await client.entry.create(...)
} catch (error) {
  if (error instanceof AxiosError && error.response?.status === 422) {
    const errors = error.response.data.details?.errors || []

    const errorMap = errors.reduce((acc, err) => {
      acc[err.path] = {
        name: err.name,
        details: err.details,
      }
      return acc
    }, {})

    console.error('Validation errors:', errorMap)

    // Handle specific validation errors
    if (errorMap['fields.slug']) {
      console.error('Slug validation failed:', errorMap['fields.slug'].name)
      // - unique: Slug already exists
      // - regexp: Slug doesn't match pattern
      // - required: Slug is required
    }

    if (errorMap['fields.title']) {
      console.error('Title validation failed:', errorMap['fields.title'].name)
      // - size: Title too long/short
      // - required: Title is required
    }
  }
}
```

## Logging and Monitoring

### Custom Logger

```typescript
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    logHandler: (level, data) => {
      console.log(`[${level}]`, JSON.stringify(data, null, 2))
    },
  }
)
```

### Request/Response Interceptors

```typescript
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    requestLogger: (config) => {
      console.log('Request:', config.method, config.url)
    },
    responseLogger: (response) => {
      console.log('Response:', response.status, response.config.url)
    },
  }
)
```

## Best Practices

1. **Check status codes** - Handle specific errors appropriately
2. **Refetch on 409** - Always get latest version before retrying
3. **Use auto-retry** - SDK handles 429 and 500 errors by default
4. **Monitor rate limits** - Check headers to avoid hitting limits
5. **Validate locally** - Check data before sending to API
6. **Log errors** - Use custom loggers for debugging
7. **Handle missing resources** - Check for 404 and handle gracefully
8. **Exponential backoff** - Use increasing delays for retries
9. **Parse validation errors** - Extract field-specific errors from 422 responses
10. **Set retry limits** - Prevent infinite retry loops

## Complete Error Handler

```typescript
async function safeApiCall<T>(
  operation: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await operation()
    } catch (error) {
      if (!(error instanceof AxiosError)) {
        throw error
      }

      const status = error.response?.status

      switch (status) {
        case 401:
          throw new Error('Authentication failed')

        case 403:
          throw new Error('Permission denied')

        case 404:
          throw new Error('Resource not found')

        case 409:
          if (attempt < maxRetries - 1) {
            console.log('Version conflict, retrying...')
            await new Promise(resolve => setTimeout(resolve, 1000 * (attempt + 1)))
            continue
          }
          throw new Error('Version conflict - max retries exceeded')

        case 422:
          const errors = error.response.data.details?.errors || []
          throw new Error(`Validation failed: ${JSON.stringify(errors)}`)

        case 429:
          const resetTime = error.response.headers['x-contentful-ratelimit-reset']
          console.log(`Rate limited, waiting ${resetTime}s...`)
          await new Promise(resolve => setTimeout(resolve, resetTime * 1000))
          continue

        case 500:
        case 502:
        case 503:
          if (attempt < maxRetries - 1) {
            console.log('Server error, retrying...')
            await new Promise(resolve => setTimeout(resolve, 2000 * (attempt + 1)))
            continue
          }
          throw new Error('Server error - max retries exceeded')

        default:
          throw error
      }
    }
  }

  throw new Error('Max retries exceeded')
}

// Usage
const entry = await safeApiCall(() =>
  client.entry.create(
    { spaceId: 'xxx', environmentId: 'master', contentTypeId: 'blogPost' },
    { fields: { ... } }
  )
)
```
