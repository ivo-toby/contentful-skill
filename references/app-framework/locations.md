# App Locations

Guide to all app locations where Contentful Apps can render.

## Overview

Apps can render in different locations within the Contentful web app:

- **Entry Field** - Custom field editors
- **Entry Sidebar** - Sidebar widgets in the entry editor
- **Entry Editor** - Full entry page replacement
- **Dialog** - Modal dialogs for workflows
- **Page** - Standalone pages
- **App Configuration** - App setup and configuration screen

## Entry Field

Custom field editors for specific fields.

### Field SDK

```typescript
import { init, locations } from '@contentful/app-sdk'
import type { FieldAppSDK } from '@contentful/app-sdk'

init<FieldAppSDK>((sdk) => {
  if (!sdk.location.is(locations.LOCATION_ENTRY_FIELD)) return

  // Get current value
  const value = sdk.field.getValue()
  console.log('Current value:', value)

  // Set value
  sdk.field.setValue('new value')

  // Listen to changes
  sdk.field.onValueChanged((value) => {
    console.log('Value changed:', value)
  })

  // Access field metadata
  console.log('Field ID:', sdk.field.id)
  console.log('Field type:', sdk.field.type)
  console.log('Field locale:', sdk.field.locale)
  console.log('Field required:', sdk.field.required)
  console.log('Field validations:', sdk.field.validations)

  sdk.window.startAutoResizer()
})
```

### Field Methods

```typescript
// Get value
const value = sdk.field.getValue()

// Set value
sdk.field.setValue('new value')

// Set value for specific locale
sdk.field.setValue('German value', 'de-DE')

// Remove value
sdk.field.removeValue()

// On value changed
const detach = sdk.field.onValueChanged((value) => {
  console.log('New value:', value)
})

// Unsubscribe
detach()

// On field disabled
sdk.field.onIsDisabledChanged((isDisabled) => {
  console.log('Field disabled:', isDisabled)
})

// Set invalid (validation error)
sdk.field.setInvalid(true)

// Check schema errors
const errors = sdk.field.validations
```

### Field Types

Handle different field types:

```typescript
init<FieldAppSDK>((sdk) => {
  const fieldType = sdk.field.type

  switch (fieldType) {
    case 'Symbol':
    case 'Text':
      const text = sdk.field.getValue() as string
      sdk.field.setValue('Updated text')
      break

    case 'Integer':
    case 'Number':
      const number = sdk.field.getValue() as number
      sdk.field.setValue(42)
      break

    case 'Boolean':
      const bool = sdk.field.getValue() as boolean
      sdk.field.setValue(!bool)
      break

    case 'Date':
      const date = sdk.field.getValue() as string // ISO 8601
      sdk.field.setValue(new Date().toISOString())
      break

    case 'Location':
      const location = sdk.field.getValue() as { lat: number; lon: number }
      sdk.field.setValue({ lat: 40.7128, lon: -74.006 })
      break

    case 'Object':
      const obj = sdk.field.getValue() as Record<string, any>
      sdk.field.setValue({ key: 'value' })
      break

    case 'Array':
      const array = sdk.field.getValue() as any[]
      sdk.field.setValue([...array, 'new item'])
      break

    case 'Link':
      const link = sdk.field.getValue() as {
        sys: { type: 'Link'; linkType: 'Entry' | 'Asset'; id: string }
      }
      break

    case 'RichText':
      const richText = sdk.field.getValue() as Document
      break
  }
})
```

### React Field Example

```tsx
import { useEffect, useState } from 'react'
import { TextInput } from '@contentful/f36-components'

function FieldEditor({ sdk }: { sdk: FieldAppSDK }) {
  const [value, setValue] = useState(sdk.field.getValue() || '')

  useEffect(() => {
    sdk.window.startAutoResizer()
  }, [sdk])

  useEffect(() => {
    const detach = sdk.field.onValueChanged((newValue) => {
      setValue(newValue || '')
    })

    return () => detach()
  }, [sdk])

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value
    setValue(newValue)
    sdk.field.setValue(newValue)
  }

  return <TextInput value={value} onChange={handleChange} />
}
```

## Entry Sidebar

Sidebar widgets that appear alongside the entry editor.

### Sidebar SDK

```typescript
import { init, locations } from '@contentful/app-sdk'
import type { SidebarAppSDK } from '@contentful/app-sdk'

init<SidebarAppSDK>((sdk) => {
  if (!sdk.location.is(locations.LOCATION_ENTRY_SIDEBAR)) return

  // Access entry
  const entry = sdk.entry

  // Get all fields
  const fields = entry.fields
  console.log('Fields:', fields)

  // Get specific field
  const titleField = entry.fields.title
  const title = titleField.getValue()

  // Listen to field changes
  titleField.onValueChanged((value) => {
    console.log('Title changed:', value)
  })

  // Get entry metadata
  console.log('Entry ID:', entry.getSys().id)
  console.log('Content type:', entry.getSys().contentType.sys.id)
  console.log('Published:', entry.getSys().publishedVersion)

  // Listen to entry changes
  entry.onSysChanged((sys) => {
    console.log('Entry sys changed:', sys)
  })

  sdk.window.startAutoResizer()
})
```

### Entry Methods

```typescript
// Get sys metadata
const sys = sdk.entry.getSys()

// Get all fields
const fields = sdk.entry.fields

// Get locales
const locales = sdk.entry.getLocales()

// Set field value
sdk.entry.fields.title.setValue('New title')

// On sys changed
sdk.entry.onSysChanged((sys) => {
  console.log('Entry updated:', sys.updatedAt)
})
```

### React Sidebar Example

```tsx
function Sidebar({ sdk }: { sdk: SidebarAppSDK }) {
  const [title, setTitle] = useState('')
  const [wordCount, setWordCount] = useState(0)

  useEffect(() => {
    sdk.window.startAutoResizer()

    const titleField = sdk.entry.fields.title
    const bodyField = sdk.entry.fields.body

    const updateTitle = (value: string) => {
      setTitle(value || '')
    }

    const updateWordCount = (value: string) => {
      if (value) {
        setWordCount(value.split(/\s+/).length)
      } else {
        setWordCount(0)
      }
    }

    updateTitle(titleField.getValue())
    updateWordCount(bodyField.getValue())

    const detachTitle = titleField.onValueChanged(updateTitle)
    const detachBody = bodyField.onValueChanged(updateWordCount)

    return () => {
      detachTitle()
      detachBody()
    }
  }, [sdk])

  return (
    <div>
      <h3>Entry Stats</h3>
      <p>Title: {title}</p>
      <p>Word count: {wordCount}</p>
    </div>
  )
}
```

## Entry Editor

Full entry page replacement or extension.

### Editor SDK

```typescript
import { init, locations } from '@contentful/app-sdk'
import type { EditorAppSDK } from '@contentful/app-sdk'

init<EditorAppSDK>((sdk) => {
  if (!sdk.location.is(locations.LOCATION_ENTRY_EDITOR)) return

  // Access entry (same as sidebar)
  const entry = sdk.entry

  // Get content type
  const contentType = sdk.contentType

  console.log('Content type name:', contentType.name)
  console.log('Fields:', contentType.fields)
  console.log('Display field:', contentType.displayField)

  // Editor-specific methods
  sdk.editor.editorInterface.getEditorLayout().then((layout) => {
    console.log('Editor layout:', layout)
  })

  sdk.window.startAutoResizer()
})
```

## Dialog

Modal dialogs for workflows and user interactions.

### Dialog SDK

```typescript
import { init, locations } from '@contentful/app-sdk'
import type { DialogAppSDK } from '@contentful/app-sdk'

init<DialogAppSDK>((sdk) => {
  if (!sdk.location.is(locations.LOCATION_DIALOG)) return

  // Get invocation parameters (passed when opening dialog)
  const params = sdk.parameters.invocation
  console.log('Dialog params:', params)

  // Close dialog with result
  document.getElementById('save')?.addEventListener('click', () => {
    sdk.close({ result: 'saved', data: { foo: 'bar' } })
  })

  document.getElementById('cancel')?.addEventListener('click', () => {
    sdk.close(null)
  })

  sdk.window.startAutoResizer()
})
```

### Open Dialog

From other locations:

```typescript
// Open dialog from field or sidebar
const result = await sdk.dialogs.openCurrentApp({
  title: 'Select Item',
  width: 800,
  parameters: { mode: 'select', items: ['a', 'b', 'c'] },
  allowHeightOverflow: true,
})

if (result) {
  console.log('Dialog result:', result)
}
```

### React Dialog Example

```tsx
function Dialog({ sdk }: { sdk: DialogAppSDK }) {
  const params = sdk.parameters.invocation as { items: string[] }

  const [selected, setSelected] = useState<string | null>(null)

  useEffect(() => {
    sdk.window.startAutoResizer()
  }, [sdk])

  const handleSave = () => {
    sdk.close({ selected })
  }

  const handleCancel = () => {
    sdk.close(null)
  }

  return (
    <div>
      <h2>Select Item</h2>
      {params.items.map((item) => (
        <div key={item}>
          <input
            type="radio"
            name="item"
            value={item}
            onChange={() => setSelected(item)}
          />
          <label>{item}</label>
        </div>
      ))}
      <button onClick={handleSave}>Save</button>
      <button onClick={handleCancel}>Cancel</button>
    </div>
  )
}
```

## Page

Standalone pages accessible from the app navigation.

### Page SDK

```typescript
import { init, locations } from '@contentful/app-sdk'
import type { PageAppSDK } from '@contentful/app-sdk'

init<PageAppSDK>((sdk) => {
  if (!sdk.location.is(locations.LOCATION_PAGE)) return

  // Page-specific methods
  console.log('Space:', sdk.ids.space)
  console.log('Environment:', sdk.ids.environment)

  // Use navigator
  sdk.navigator.openEntry('entry-id')
  sdk.navigator.openAsset('asset-id')

  // No auto-resize needed for pages
})
```

### React Page Example

```tsx
function Page({ sdk }: { sdk: PageAppSDK }) {
  const [entries, setEntries] = useState([])

  useEffect(() => {
    // Fetch entries using CMA
    const cma = createCMAClient(sdk)

    cma.entry.getMany({ content_type: 'blogPost' }).then((result) => {
      setEntries(result.items)
    })
  }, [sdk])

  const openEntry = (entryId: string) => {
    sdk.navigator.openEntry(entryId)
  }

  return (
    <div>
      <h1>Blog Posts</h1>
      <ul>
        {entries.map((entry) => (
          <li key={entry.sys.id}>
            <button onClick={() => openEntry(entry.sys.id)}>
              {entry.fields.title['en-US']}
            </button>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

## App Configuration

Configuration screen for app installation.

### Configuration SDK

```typescript
import { init, locations } from '@contentful/app-sdk'
import type { ConfigAppSDK } from '@contentful/app-sdk'

init<ConfigAppSDK>((sdk) => {
  if (!sdk.location.is(locations.LOCATION_APP_CONFIG)) return

  // Configure app
  sdk.app.onConfigure(async () => {
    const currentState = await sdk.app.getCurrentState()

    return {
      parameters: {
        apiKey: 'your-api-key',
        webhookUrl: 'https://example.com',
      },
      targetState: {
        EditorInterface: {
          blogPost: {
            sidebar: { position: 0 },
            controls: [{ fieldId: 'title' }],
          },
        },
      },
    }
  })

  // Set app ready
  sdk.app.setReady()

  // Get current parameters
  sdk.app.getParameters().then((params) => {
    console.log('Current parameters:', params)
  })

  sdk.window.startAutoResizer()
})
```

### React Configuration Example

```tsx
function Config({ sdk }: { sdk: ConfigAppSDK }) {
  const [apiKey, setApiKey] = useState('')
  const [contentTypes, setContentTypes] = useState([])

  useEffect(() => {
    sdk.window.startAutoResizer()

    // Load current parameters
    sdk.app.getParameters().then((params) => {
      if (params?.apiKey) {
        setApiKey(params.apiKey)
      }
    })

    // Load content types
    const cma = createCMAClient(sdk)
    cma.contentType.getMany({}).then((result) => {
      setContentTypes(result.items)
    })

    // Configure app
    sdk.app.onConfigure(() => {
      return {
        parameters: { apiKey },
        targetState: {
          EditorInterface: {
            blogPost: { sidebar: { position: 0 } },
          },
        },
      }
    })

    sdk.app.setReady()
  }, [sdk])

  return (
    <div>
      <h2>App Configuration</h2>
      <TextInput
        value={apiKey}
        onChange={(e) => setApiKey(e.target.value)}
        placeholder="API Key"
      />
    </div>
  )
}
```

## Best Practices

1. **Check location** - Always verify location before accessing location-specific APIs
2. **Auto-resize** - Enable auto-resizing for field, sidebar, and dialog locations
3. **Unsubscribe** - Clean up event listeners when component unmounts
4. **Handle errors** - Wrap SDK calls in try-catch blocks
5. **Use TypeScript** - Type your SDK for location-specific APIs
6. **Validate field values** - Check field type before operations
7. **Test all locations** - Ensure app works in all configured locations
8. **Use React hooks** - Leverage @contentful/react-apps-toolkit for cleaner code
9. **Handle entry state** - Monitor entry sys changes for published/draft state
10. **Provide fallbacks** - Handle missing fields or values gracefully
