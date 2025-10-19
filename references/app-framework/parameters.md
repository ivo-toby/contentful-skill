# Parameters

Guide to installation, instance, and invocation parameters in Contentful Apps.

## Overview

Apps can use three types of parameters:

- **Installation Parameters** - App-wide configuration (set during installation)
- **Instance Parameters** - Per-location configuration (per field, sidebar, etc.)
- **Invocation Parameters** - Dialog-specific parameters (passed when opening dialog)

## Installation Parameters

App-wide configuration set during installation.

### Define in Configuration

```typescript
import type { ConfigAppSDK } from '@contentful/app-sdk'

init<ConfigAppSDK>((sdk) => {
  sdk.app.onConfigure(() => {
    return {
      parameters: {
        apiKey: 'your-api-key',
        webhookUrl: 'https://example.com/webhook',
        enableFeature: true,
        timeout: 5000,
      },
      targetState: {
        EditorInterface: {
          blogPost: { sidebar: { position: 0 } },
        },
      },
    }
  })

  sdk.app.setReady()
})
```

### Access Installation Parameters

From any location:

```typescript
init((sdk) => {
  const params = sdk.parameters.installation

  console.log('API Key:', params.apiKey)
  console.log('Webhook URL:', params.webhookUrl)
  console.log('Feature enabled:', params.enableFeature)
  console.log('Timeout:', params.timeout)
})
```

### TypeScript Typed Parameters

```typescript
interface InstallationParameters {
  apiKey: string
  webhookUrl: string
  enableFeature: boolean
  timeout: number
}

init((sdk) => {
  const params = sdk.parameters.installation as InstallationParameters

  console.log('API Key:', params.apiKey) // Type-safe access
})
```

### Get Parameters Programmatically

```typescript
init<ConfigAppSDK>((sdk) => {
  sdk.app.getParameters().then((parameters) => {
    console.log('Current parameters:', parameters)
  })
})
```

### Update Parameters

Only possible in configuration location:

```typescript
init<ConfigAppSDK>((sdk) => {
  const [apiKey, setApiKey] = useState('')

  sdk.app.onConfigure(() => {
    return {
      parameters: {
        apiKey, // Updated value
      },
      targetState: {},
    }
  })

  sdk.app.setReady()
})
```

## Instance Parameters

Per-location configuration (e.g., per field or sidebar instance).

### Configure in Target State

```typescript
init<ConfigAppSDK>((sdk) => {
  sdk.app.onConfigure(() => {
    return {
      parameters: {
        // Installation parameters
      },
      targetState: {
        EditorInterface: {
          blogPost: {
            sidebar: {
              position: 0,
              settings: {
                // Instance parameters for sidebar
                showStats: true,
                highlightColor: '#ff0000',
              },
            },
            controls: [
              {
                fieldId: 'customField',
                settings: {
                  // Instance parameters for field
                  mode: 'advanced',
                  maxLength: 100,
                },
              },
            ],
          },
        },
      },
    }
  })
})
```

### Access Instance Parameters

From field or sidebar:

```typescript
init((sdk) => {
  const instanceParams = sdk.parameters.instance

  console.log('Instance config:', instanceParams)
})
```

### Field Instance Example

```typescript
interface FieldInstanceParameters {
  mode: 'simple' | 'advanced'
  maxLength: number
}

init<FieldAppSDK>((sdk) => {
  const params = sdk.parameters.instance as FieldInstanceParameters

  if (params.mode === 'advanced') {
    // Show advanced editor
  } else {
    // Show simple editor
  }

  const maxLength = params.maxLength || 255
})
```

### Sidebar Instance Example

```typescript
interface SidebarInstanceParameters {
  showStats: boolean
  highlightColor: string
}

init<SidebarAppSDK>((sdk) => {
  const params = sdk.parameters.instance as SidebarInstanceParameters

  if (params.showStats) {
    // Show statistics
  }

  const color = params.highlightColor || '#000000'
})
```

## Invocation Parameters

Dialog-specific parameters passed when opening a dialog.

### Pass When Opening Dialog

```typescript
// From field or sidebar
const result = await sdk.dialogs.openCurrentApp({
  title: 'Select Items',
  width: 800,
  parameters: {
    // Invocation parameters
    mode: 'select',
    contentType: 'blogPost',
    maxItems: 5,
    items: ['a', 'b', 'c'],
  },
})
```

### Access in Dialog

```typescript
interface InvocationParameters {
  mode: 'select' | 'create'
  contentType: string
  maxItems: number
  items: string[]
}

init<DialogAppSDK>((sdk) => {
  const params = sdk.parameters.invocation as InvocationParameters

  console.log('Mode:', params.mode)
  console.log('Content type:', params.contentType)
  console.log('Max items:', params.maxItems)
  console.log('Items:', params.items)

  if (params.mode === 'select') {
    // Show selection UI
  } else {
    // Show creation UI
  }
})
```

## Complete Examples

### Configuration with Form

```tsx
import { useState, useEffect } from 'react'
import { TextInput, Checkbox, Button } from '@contentful/f36-components'
import type { ConfigAppSDK } from '@contentful/app-sdk'

interface InstallationParameters {
  apiKey: string
  webhookUrl: string
  enableNotifications: boolean
}

function Config({ sdk }: { sdk: ConfigAppSDK }) {
  const [apiKey, setApiKey] = useState('')
  const [webhookUrl, setWebhookUrl] = useState('')
  const [enableNotifications, setEnableNotifications] = useState(true)
  const [contentTypes, setContentTypes] = useState([])

  useEffect(() => {
    sdk.window.startAutoResizer()

    // Load existing parameters
    sdk.app.getParameters<InstallationParameters>().then((params) => {
      if (params) {
        setApiKey(params.apiKey || '')
        setWebhookUrl(params.webhookUrl || '')
        setEnableNotifications(params.enableNotifications ?? true)
      }
    })

    // Load content types
    const cma = createCMAClient(sdk)
    cma.contentType.getMany({}).then((result) => {
      setContentTypes(result.items)
    })

    // Configure app
    sdk.app.onConfigure(() => {
      // Validate
      if (!apiKey) {
        sdk.notifier.error('API Key is required')
        return false
      }

      return {
        parameters: {
          apiKey,
          webhookUrl,
          enableNotifications,
        },
        targetState: {
          EditorInterface: {
            blogPost: {
              sidebar: {
                position: 0,
                settings: {
                  showStats: true,
                },
              },
            },
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
        isRequired
      />

      <TextInput
        value={webhookUrl}
        onChange={(e) => setWebhookUrl(e.target.value)}
        placeholder="Webhook URL"
      />

      <Checkbox
        isChecked={enableNotifications}
        onChange={() => setEnableNotifications(!enableNotifications)}
      >
        Enable Notifications
      </Checkbox>
    </div>
  )
}
```

### Field with Instance Parameters

```tsx
import { useState, useEffect } from 'react'
import { TextInput, Select } from '@contentful/f36-components'
import type { FieldAppSDK } from '@contentful/app-sdk'

interface FieldInstanceParameters {
  mode: 'simple' | 'advanced'
  maxLength: number
  placeholder: string
}

function CustomField({ sdk }: { sdk: FieldAppSDK }) {
  const [value, setValue] = useState(sdk.field.getValue() || '')

  const instanceParams = sdk.parameters.instance as FieldInstanceParameters
  const mode = instanceParams?.mode || 'simple'
  const maxLength = instanceParams?.maxLength || 255
  const placeholder = instanceParams?.placeholder || 'Enter text'

  useEffect(() => {
    sdk.window.startAutoResizer()

    const detach = sdk.field.onValueChanged((newValue) => {
      setValue(newValue || '')
    })

    return () => detach()
  }, [sdk])

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value

    if (newValue.length <= maxLength) {
      setValue(newValue)
      sdk.field.setValue(newValue)
    } else {
      sdk.notifier.warning(`Maximum length is ${maxLength}`)
    }
  }

  return (
    <div>
      {mode === 'advanced' ? (
        <div>
          <TextInput
            value={value}
            onChange={handleChange}
            placeholder={placeholder}
            maxLength={maxLength}
          />
          <div>
            {value.length} / {maxLength}
          </div>
        </div>
      ) : (
        <TextInput
          value={value}
          onChange={handleChange}
          placeholder={placeholder}
        />
      )}
    </div>
  )
}
```

### Dialog with Invocation Parameters

```tsx
import { useState, useEffect } from 'react'
import { Button, List } from '@contentful/f36-components'
import type { DialogAppSDK } from '@contentful/app-sdk'

interface InvocationParameters {
  mode: 'select'
  items: string[]
  maxItems: number
  preselected?: string[]
}

function SelectDialog({ sdk }: { sdk: DialogAppSDK }) {
  const params = sdk.parameters.invocation as InvocationParameters

  const [selected, setSelected] = useState<string[]>(params.preselected || [])

  useEffect(() => {
    sdk.window.startAutoResizer()
  }, [sdk])

  const toggleItem = (item: string) => {
    if (selected.includes(item)) {
      setSelected(selected.filter((i) => i !== item))
    } else {
      if (selected.length < params.maxItems) {
        setSelected([...selected, item])
      } else {
        sdk.notifier.warning(`Maximum ${params.maxItems} items allowed`)
      }
    }
  }

  const handleSave = () => {
    sdk.close({ selected })
  }

  const handleCancel = () => {
    sdk.close(null)
  }

  return (
    <div>
      <h2>Select Items ({selected.length}/{params.maxItems})</h2>

      <List>
        {params.items.map((item) => (
          <List.Item key={item}>
            <Checkbox
              isChecked={selected.includes(item)}
              onChange={() => toggleItem(item)}
            >
              {item}
            </Checkbox>
          </List.Item>
        ))}
      </List>

      <Button onClick={handleSave}>Save</Button>
      <Button onClick={handleCancel} variant="secondary">
        Cancel
      </Button>
    </div>
  )
}
```

### Opening Dialog with Parameters

```tsx
// From field location
import type { FieldAppSDK } from '@contentful/app-sdk'

function FieldEditor({ sdk }: { sdk: FieldAppSDK }) {
  const installationParams = sdk.parameters.installation
  const instanceParams = sdk.parameters.instance

  const openSelector = async () => {
    const result = await sdk.dialogs.openCurrentApp({
      title: 'Select Items',
      width: 800,
      parameters: {
        // Pass both installation and instance params to dialog
        mode: 'select',
        items: installationParams.availableItems,
        maxItems: instanceParams.maxItems || 5,
        preselected: sdk.field.getValue() || [],
      },
    })

    if (result) {
      sdk.field.setValue(result.selected)
      sdk.notifier.success(`Selected ${result.selected.length} items`)
    }
  }

  return <Button onClick={openSelector}>Select Items</Button>
}
```

## Parameter Validation

### Validate in Configuration

```typescript
init<ConfigAppSDK>((sdk) => {
  sdk.app.onConfigure(() => {
    const apiKey = document.getElementById('apiKey').value

    // Validate
    if (!apiKey || apiKey.length < 10) {
      sdk.notifier.error('API Key must be at least 10 characters')
      return false
    }

    // Test API key
    try {
      await fetch(`https://api.example.com/test?key=${apiKey}`)
    } catch {
      sdk.notifier.error('Invalid API Key')
      return false
    }

    return {
      parameters: { apiKey },
      targetState: {},
    }
  })
})
```

### Validate in Field

```typescript
init<FieldAppSDK>((sdk) => {
  const instanceParams = sdk.parameters.instance

  if (!instanceParams.mode) {
    sdk.notifier.error('Field not configured properly')
    return
  }

  // Continue with valid params
})
```

## Best Practices

1. **Type your parameters** - Use TypeScript interfaces for type safety
2. **Provide defaults** - Handle missing parameters gracefully
3. **Validate in configuration** - Check parameters before saving
4. **Use installation params** - For app-wide settings (API keys, URLs)
5. **Use instance params** - For per-location customization
6. **Use invocation params** - For dialog-specific data
7. **Document parameters** - Provide clear descriptions in configuration UI
8. **Test with missing params** - Handle cases where parameters are undefined
9. **Don't hardcode** - Use parameters for all configurable values
10. **Secure sensitive data** - Never expose tokens or secrets in parameters

## Parameter Flow

```
Configuration Location
  ↓
  Set Installation Parameters (app-wide)
  Set Instance Parameters (per-location)
  ↓
Any Location (Field, Sidebar, Page, etc.)
  ↓
  Access Installation Parameters
  Access Instance Parameters
  ↓
Open Dialog
  ↓
  Pass Invocation Parameters
  ↓
Dialog Location
  ↓
  Access All Three Types of Parameters
  Access Invocation Parameters
```

## Common Patterns

### API Key Configuration

```typescript
// Configuration
parameters: {
  apiKey: 'sk_live_...',
  apiEndpoint: 'https://api.example.com'
}

// Usage
const { apiKey, apiEndpoint } = sdk.parameters.installation
const response = await fetch(`${apiEndpoint}/data`, {
  headers: { Authorization: `Bearer ${apiKey}` }
})
```

### Feature Flags

```typescript
// Configuration
parameters: {
  enableAdvancedMode: true,
  enableExport: false,
  enableSync: true
}

// Usage
const params = sdk.parameters.installation

if (params.enableAdvancedMode) {
  // Show advanced features
}

if (params.enableExport) {
  // Show export button
}
```

### Per-Field Configuration

```typescript
// Target state
targetState: {
  EditorInterface: {
    product: {
      controls: [
        {
          fieldId: 'price',
          settings: {
            currency: 'USD',
            min: 0,
            max: 10000
          }
        },
        {
          fieldId: 'weight',
          settings: {
            unit: 'kg',
            precision: 2
          }
        }
      ]
    }
  }
}

// Field usage
const { currency, min, max } = sdk.parameters.instance
```
