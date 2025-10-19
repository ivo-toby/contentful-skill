# App Framework SDK Overview

Quick start guide for building Contentful Apps with the App Framework SDK.

## Installation

```bash
npm install @contentful/app-sdk
```

## Create Contentful App

Use the CLI to scaffold a new app:

```bash
# TypeScript (default)
npx create-contentful-app my-app

# JavaScript
npx create-contentful-app --javascript my-app
```

## SDK Initialization

### Basic Init

```typescript
import { init } from '@contentful/app-sdk'

init((sdk) => {
  console.log('App initialized:', sdk)

  // Access SDK features
  console.log('Location:', sdk.location.is('entry-field'))
  console.log('IDs:', sdk.ids)
  console.log('User:', sdk.user)
})
```

### With Type Safety

```typescript
import { init, locations } from '@contentful/app-sdk'
import type { FieldAppSDK } from '@contentful/app-sdk'

init<FieldAppSDK>((sdk) => {
  // sdk is typed as FieldAppSDK
  if (sdk.location.is(locations.LOCATION_ENTRY_FIELD)) {
    const value = sdk.field.getValue()
    console.log('Field value:', value)
  }
})
```

### Async Init

```typescript
import { init } from '@contentful/app-sdk'

async function initApp() {
  const sdk = await init()

  console.log('SDK ready:', sdk)
  return sdk
}

initApp().catch(console.error)
```

## SDK Structure

### IDs

Access current space, environment, entry IDs:

```typescript
init((sdk) => {
  console.log(sdk.ids.space) // Space ID
  console.log(sdk.ids.environment) // Environment ID
  console.log(sdk.ids.environmentAlias) // Environment alias (if used)
  console.log(sdk.ids.entry) // Current entry ID (if in entry context)
  console.log(sdk.ids.contentType) // Content type ID (if in entry context)
  console.log(sdk.ids.field) // Field ID (if in field context)
  console.log(sdk.ids.user) // Current user ID
  console.log(sdk.ids.app) // App definition ID
})
```

### User

Access current user information:

```typescript
init((sdk) => {
  console.log(sdk.user.email)
  console.log(sdk.user.firstName)
  console.log(sdk.user.lastName)
  console.log(sdk.user.avatarUrl)
  console.log(sdk.user.sys.id)
})
```

### Location

Determine where the app is running:

```typescript
import { locations } from '@contentful/app-sdk'

init((sdk) => {
  if (sdk.location.is(locations.LOCATION_ENTRY_FIELD)) {
    console.log('Running in field editor')
  }

  if (sdk.location.is(locations.LOCATION_ENTRY_SIDEBAR)) {
    console.log('Running in sidebar')
  }

  if (sdk.location.is(locations.LOCATION_DIALOG)) {
    console.log('Running in dialog')
  }

  if (sdk.location.is(locations.LOCATION_ENTRY_EDITOR)) {
    console.log('Running in entry editor')
  }

  if (sdk.location.is(locations.LOCATION_PAGE)) {
    console.log('Running in page')
  }

  if (sdk.location.is(locations.LOCATION_APP_CONFIG)) {
    console.log('Running in app configuration')
  }
})
```

## CMA Integration

### Create CMA Client

Use the SDK's CMA adapter for authenticated API access:

```typescript
import { init } from '@contentful/app-sdk'
import contentful from 'contentful-management'

init(async (sdk) => {
  const cma = contentful.createClient(
    { apiAdapter: sdk.cmaAdapter },
    {
      type: 'plain',
      defaults: {
        environmentId: sdk.ids.environmentAlias ?? sdk.ids.environment,
        spaceId: sdk.ids.space,
      },
    }
  )

  // Use CMA without exposing token
  const entry = await cma.entry.get({
    entryId: sdk.ids.entry,
  })

  console.log(entry.fields)
})
```

### With Scoped Client

```typescript
import { init } from '@contentful/app-sdk'
import contentful from 'contentful-management'

async function createCMAClient(sdk) {
  return contentful.createClient(
    { apiAdapter: sdk.cmaAdapter },
    {
      type: 'plain',
      defaults: {
        spaceId: sdk.ids.space,
        environmentId: sdk.ids.environmentAlias ?? sdk.ids.environment,
      },
    }
  )
}

init(async (sdk) => {
  const cma = await createCMAClient(sdk)

  // Create entry
  const newEntry = await cma.entry.create(
    {
      contentTypeId: 'blogPost',
    },
    {
      fields: {
        title: { 'en-US': 'New Post' },
      },
    }
  )

  console.log('Created:', newEntry.sys.id)
})
```

## TypeScript Types

### Location-Specific SDK Types

```typescript
import type {
  FieldAppSDK,
  SidebarAppSDK,
  DialogAppSDK,
  EditorAppSDK,
  PageAppSDK,
  ConfigAppSDK,
} from '@contentful/app-sdk'

// Field editor
init<FieldAppSDK>((sdk) => {
  const value = sdk.field.getValue()
  sdk.field.setValue('new value')
})

// Sidebar
init<SidebarAppSDK>((sdk) => {
  const entry = sdk.entry
  console.log(entry.fields)
})

// Dialog
init<DialogAppSDK>((sdk) => {
  sdk.close({ result: 'success' })
})

// Entry editor
init<EditorAppSDK>((sdk) => {
  const entry = sdk.entry
  console.log(entry.getSys())
})

// Page
init<PageAppSDK>((sdk) => {
  sdk.navigator.openEntry(entryId)
})

// Configuration
init<ConfigAppSDK>((sdk) => {
  sdk.app.onConfigure(() => {
    return {
      parameters: {},
      targetState: {},
    }
  })
})
```

### Field Value Types

```typescript
import type { FieldAppSDK } from '@contentful/app-sdk'

type TextFieldValue = string
type NumberFieldValue = number
type BooleanFieldValue = boolean
type DateFieldValue = string // ISO 8601
type LocationFieldValue = { lat: number; lon: number }
type LinkFieldValue = { sys: { type: 'Link'; linkType: 'Entry' | 'Asset'; id: string } }
type ArrayFieldValue = any[]

init<FieldAppSDK>((sdk) => {
  const value = sdk.field.getValue() as TextFieldValue
  console.log('Text value:', value)
})
```

## App Configuration

### Installation Parameters

Configure app settings:

```typescript
import type { ConfigAppSDK } from '@contentful/app-sdk'

init<ConfigAppSDK>((sdk) => {
  sdk.app.onConfigure(async () => {
    const parameters = {
      apiKey: 'your-api-key',
      webhookUrl: 'https://example.com/webhook',
      enableFeature: true,
    }

    return {
      parameters,
      targetState: {
        EditorInterface: {
          blogPost: {
            sidebar: { position: 0 },
            controls: [
              {
                fieldId: 'title',
              },
            ],
          },
        },
      },
    }
  })

  sdk.app.onConfigurationCompleted((parameters) => {
    console.log('App configured with:', parameters)
  })

  sdk.app.getParameters().then((parameters) => {
    console.log('Current parameters:', parameters)
  })

  sdk.app.setReady()
})
```

### Access Parameters

```typescript
import { init } from '@contentful/app-sdk'

init((sdk) => {
  // Installation parameters (app-wide)
  const installationParams = sdk.parameters.installation
  console.log('API Key:', installationParams.apiKey)

  // Instance parameters (per-field or per-location)
  const instanceParams = sdk.parameters.instance
  console.log('Field config:', instanceParams)

  // Invocation parameters (for dialogs)
  const invocationParams = sdk.parameters.invocation
  console.log('Dialog params:', invocationParams)
})
```

## Auto-Resizing

### Enable Auto-Resize

```typescript
init((sdk) => {
  sdk.window.startAutoResizer()

  // Disable later if needed
  // sdk.window.stopAutoResizer()
})
```

### Manual Resize

```typescript
init((sdk) => {
  sdk.window.updateHeight(500)
})
```

## Topics

Detailed guides for specific features:

- **[Locations](locations.md)** - All app locations (field, sidebar, dialog, entry editor, page)
- **[SDK APIs](sdk-apis.md)** - Navigator, dialogs, notifier, access, and utilities
- **[Parameters](parameters.md)** - Installation, instance, and invocation parameters

## Best Practices

1. **Use TypeScript** - Leverage SDK types for type safety
2. **Use cmaAdapter** - Never expose management tokens in app code
3. **Handle initialization** - Wait for SDK to be ready before operations
4. **Auto-resize** - Enable auto-resizing for better UX
5. **Check location** - Determine app location for conditional logic
6. **Use scoped CMA client** - Set defaults for space/environment
7. **Access parameters** - Use installation/instance parameters for configuration
8. **Handle errors** - Wrap SDK calls in try-catch
9. **Test all locations** - Ensure app works in all configured locations
10. **Follow UI guidelines** - Use Forma 36 components for consistent UI

## React Integration

### With Hooks

```tsx
import { useEffect, useState } from 'react'
import { init } from '@contentful/app-sdk'
import type { FieldAppSDK } from '@contentful/app-sdk'

function App() {
  const [sdk, setSdk] = useState<FieldAppSDK | null>(null)

  useEffect(() => {
    init<FieldAppSDK>((sdk) => {
      setSdk(sdk)
      sdk.window.startAutoResizer()
    })
  }, [])

  if (!sdk) {
    return <div>Loading...</div>
  }

  return <FieldEditor sdk={sdk} />
}
```

### React Apps Toolkit

```tsx
import { useSDK, useAutoResizer } from '@contentful/react-apps-toolkit'

function App() {
  const sdk = useSDK()
  useAutoResizer()

  return <div>App content</div>
}
```

## Reference Documentation

- **SDK Docs**: [App Framework SDK](https://www.contentful.com/developers/docs/extensibility/app-framework/sdk/)
- **NPM**: [@contentful/app-sdk](https://www.npmjs.com/package/@contentful/app-sdk)
- **Create App**: [create-contentful-app](https://www.contentful.com/developers/docs/extensibility/app-framework/create-contentful-app/)
- **Tutorial**: [Custom App Tutorial](https://www.contentful.com/developers/docs/extensibility/app-framework/tutorial/)
- **React Toolkit**: [@contentful/react-apps-toolkit](https://www.npmjs.com/package/@contentful/react-apps-toolkit)
