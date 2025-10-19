# SDK APIs

Guide to SDK methods and APIs for navigation, dialogs, notifications, and utilities.

## Navigator

Navigate to different areas of the Contentful web app.

### Open Entry

```typescript
// Open entry in current space/environment
sdk.navigator.openEntry('entry-id')

// Open entry with slide-in
sdk.navigator.openEntry('entry-id', { slideIn: true })

// Open entry in new tab
sdk.navigator.openEntry('entry-id', { slideIn: false })
```

### Open Asset

```typescript
// Open asset in current space/environment
sdk.navigator.openAsset('asset-id')

// Open asset with slide-in
sdk.navigator.openAsset('asset-id', { slideIn: true })
```

### Open New Entry

```typescript
// Open editor to create new entry
sdk.navigator.openNewEntry('blogPost')

// With slide-in
sdk.navigator.openNewEntry('blogPost', { slideIn: true })
```

### Open New Asset

```typescript
// Open asset upload dialog
sdk.navigator.openNewAsset()

// With slide-in
sdk.navigator.openNewAsset({ slideIn: true })
```

### Open Page Extension

```typescript
// Navigate to custom page
sdk.navigator.openPageExtension({ id: 'my-page' })
```

### Open Current App

```typescript
// Open current app's page
sdk.navigator.openCurrentApp()

// With path
sdk.navigator.openCurrentApp({ path: '/settings' })
```

## Dialogs

Open different types of dialogs.

### Current App Dialog

Open a dialog rendering your app:

```typescript
const result = await sdk.dialogs.openCurrentApp({
  title: 'Select Item',
  width: 800,
  minHeight: 400,
  parameters: { mode: 'select', items: ['a', 'b', 'c'] },
  allowHeightOverflow: true,
  shouldCloseOnOverlayClick: true,
  shouldCloseOnEscapePress: true,
})

if (result) {
  console.log('User selected:', result.selected)
} else {
  console.log('Dialog cancelled')
}
```

### Extension Dialog

Open a custom extension:

```typescript
const result = await sdk.dialogs.openExtension({
  id: 'my-extension-id',
  title: 'Custom Dialog',
  width: 600,
  parameters: { foo: 'bar' },
})
```

### Select Single Entry

```typescript
const entry = await sdk.dialogs.selectSingleEntry({
  locale: 'en-US',
  contentTypes: ['blogPost', 'article'],
})

if (entry) {
  console.log('Selected entry:', entry.sys.id)
}
```

### Select Multiple Entries

```typescript
const entries = await sdk.dialogs.selectMultipleEntries({
  locale: 'en-US',
  contentTypes: ['blogPost'],
  min: 1,
  max: 5,
})

console.log('Selected:', entries.map(e => e.sys.id))
```

### Select Single Asset

```typescript
const asset = await sdk.dialogs.selectSingleAsset({
  locale: 'en-US',
  mimetypeGroups: ['image'],
})

if (asset) {
  console.log('Selected asset:', asset.fields.file.url)
}
```

### Select Multiple Assets

```typescript
const assets = await sdk.dialogs.selectMultipleAssets({
  locale: 'en-US',
  mimetypeGroups: ['image', 'video'],
  min: 1,
  max: 10,
})

console.log('Selected:', assets.length, 'assets')
```

### Confirm Dialog

```typescript
const confirmed = await sdk.dialogs.openConfirm({
  title: 'Delete Entry',
  message: 'Are you sure you want to delete this entry?',
  intent: 'negative',
  confirmLabel: 'Delete',
  cancelLabel: 'Cancel',
})

if (confirmed) {
  // Delete entry
}
```

### Alert Dialog

```typescript
await sdk.dialogs.openAlert({
  title: 'Success',
  message: 'Entry published successfully',
  confirmLabel: 'OK',
})
```

### Prompt Dialog

```typescript
const value = await sdk.dialogs.openPrompt({
  title: 'Enter Name',
  message: 'Please enter your name',
  defaultValue: '',
  confirmLabel: 'Submit',
  cancelLabel: 'Cancel',
})

if (value) {
  console.log('User entered:', value)
}
```

## Notifier

Show notifications to users.

### Success Notification

```typescript
sdk.notifier.success('Entry published successfully')
```

### Error Notification

```typescript
sdk.notifier.error('Failed to publish entry')
```

### Warning Notification

```typescript
sdk.notifier.warning('Please check your configuration')
```

### Info Notification

```typescript
sdk.notifier.info('New version available')
```

### Loading Notification

```typescript
const closeLoading = sdk.notifier.loading('Publishing entry...')

// Later, close the notification
setTimeout(() => {
  closeLoading()
  sdk.notifier.success('Published!')
}, 2000)
```

## Access

Check user permissions.

### Can Read

```typescript
const canRead = await sdk.access.can('read', 'Entry')
console.log('Can read entries:', canRead)
```

### Can Write

```typescript
const canWrite = await sdk.access.can('create', 'Entry')
console.log('Can create entries:', canWrite)
```

### Can Archive

```typescript
const canArchive = await sdk.access.can('archive', 'Entry')
console.log('Can archive entries:', canArchive)
```

### Can Publish

```typescript
const canPublish = await sdk.access.can('publish', 'Entry')
console.log('Can publish entries:', canPublish)
```

### Resource Types

```typescript
'Asset'
'ContentType'
'Entry'
'Environment'
'EnvironmentAlias'
'Extension'
'Locale'
'Organization'
'Role'
'Space'
'User'
'Webhook'
```

### Actions

```typescript
'create'
'read'
'update'
'delete'
'publish'
'unpublish'
'archive'
'unarchive'
```

## Window

Control window behavior.

### Start Auto-Resizer

```typescript
sdk.window.startAutoResizer()
```

### Stop Auto-Resizer

```typescript
sdk.window.stopAutoResizer()
```

### Update Height

```typescript
sdk.window.updateHeight(500)
```

### Update Width (Dialogs Only)

```typescript
sdk.window.updateWidth(800)
```

## IDs

Access current context IDs.

```typescript
// Space and environment
sdk.ids.space // 'space-id'
sdk.ids.environment // 'master'
sdk.ids.environmentAlias // 'master' or undefined

// Current user
sdk.ids.user // 'user-id'

// Current app
sdk.ids.app // 'app-definition-id'

// Entry context (if in entry location)
sdk.ids.entry // 'entry-id'
sdk.ids.contentType // 'blogPost'

// Field context (if in field location)
sdk.ids.field // 'title'
```

## User

Access current user information.

```typescript
sdk.user.email // 'user@example.com'
sdk.user.firstName // 'John'
sdk.user.lastName // 'Doe'
sdk.user.avatarUrl // 'https://...'
sdk.user.spaceMembership.sys.id // 'membership-id'
sdk.user.sys.id // 'user-id'
sdk.user.sys.type // 'User'
```

## Locale

Work with locales.

### Get Current Locale

```typescript
const locale = sdk.locales.default
console.log('Default locale:', locale.code)
```

### Get All Locales

```typescript
const locales = sdk.locales.available
for (const locale of locales) {
  console.log(locale.code, locale.name, locale.fallbackCode)
}
```

### Get Locale Names

```typescript
const names = sdk.locales.names
console.log(names) // { 'en-US': 'English (United States)', ... }
```

## Close (Dialog Only)

Close dialog and return result:

```typescript
// Close with result
sdk.close({ result: 'success', data: { id: '123' } })

// Close without result (cancel)
sdk.close(null)

// Close with undefined
sdk.close()
```

## CMA Adapter

Use with contentful-management:

```typescript
import contentful from 'contentful-management'

const cma = contentful.createClient(
  { apiAdapter: sdk.cmaAdapter },
  {
    type: 'plain',
    defaults: {
      spaceId: sdk.ids.space,
      environmentId: sdk.ids.environmentAlias ?? sdk.ids.environment,
    },
  }
)

// Use CMA without exposing token
const entry = await cma.entry.get({ entryId: 'entry-id' })
```

## App Methods (Configuration Only)

### On Configure

```typescript
sdk.app.onConfigure(async () => {
  const currentState = await sdk.app.getCurrentState()

  return {
    parameters: {
      apiKey: 'your-api-key',
    },
    targetState: {
      EditorInterface: {
        blogPost: {
          sidebar: { position: 0 },
        },
      },
    },
  }
})
```

### Set Ready

```typescript
sdk.app.setReady()
```

### Get Parameters

```typescript
const parameters = await sdk.app.getParameters()
console.log('Installation parameters:', parameters)
```

### On Configuration Completed

```typescript
sdk.app.onConfigurationCompleted((parameters) => {
  console.log('App configured with:', parameters)
})
```

### Get Current State

```typescript
const state = await sdk.app.getCurrentState()
console.log('Current editor interfaces:', state.EditorInterface)
```

### Set Installation Status

```typescript
// Mark installation as complete
sdk.app.isInstalled().then((installed) => {
  if (!installed) {
    sdk.app.setReady()
  }
})
```

## Complete Examples

### Field with Asset Picker

```tsx
import { Button } from '@contentful/f36-components'

function AssetField({ sdk }: { sdk: FieldAppSDK }) {
  const [assetId, setAssetId] = useState(sdk.field.getValue()?.sys.id)

  const selectAsset = async () => {
    const asset = await sdk.dialogs.selectSingleAsset({
      locale: sdk.field.locale,
      mimetypeGroups: ['image'],
    })

    if (asset) {
      const link = {
        sys: {
          type: 'Link',
          linkType: 'Asset',
          id: asset.sys.id,
        },
      }
      sdk.field.setValue(link)
      setAssetId(asset.sys.id)
      sdk.notifier.success('Asset selected')
    }
  }

  const removeAsset = () => {
    sdk.field.removeValue()
    setAssetId(null)
    sdk.notifier.success('Asset removed')
  }

  return (
    <div>
      {assetId ? (
        <>
          <img src={`https://images.ctfassets.net/${assetId}/...`} />
          <Button onClick={removeAsset}>Remove</Button>
        </>
      ) : (
        <Button onClick={selectAsset}>Select Asset</Button>
      )}
    </div>
  )
}
```

### Sidebar with Entry Links

```tsx
function RelatedPosts({ sdk }: { sdk: SidebarAppSDK }) {
  const [entries, setEntries] = useState([])

  const addRelatedPost = async () => {
    const entry = await sdk.dialogs.selectSingleEntry({
      locale: 'en-US',
      contentTypes: ['blogPost'],
    })

    if (entry) {
      const field = sdk.entry.fields.relatedPosts
      const current = field.getValue() || []
      const link = {
        sys: {
          type: 'Link',
          linkType: 'Entry',
          id: entry.sys.id,
        },
      }
      field.setValue([...current, link])
      sdk.notifier.success('Related post added')
    }
  }

  const openEntry = (entryId: string) => {
    sdk.navigator.openEntry(entryId, { slideIn: true })
  }

  return (
    <div>
      <h3>Related Posts</h3>
      <ul>
        {entries.map((entry) => (
          <li key={entry.sys.id}>
            <button onClick={() => openEntry(entry.sys.id)}>
              {entry.fields.title}
            </button>
          </li>
        ))}
      </ul>
      <Button onClick={addRelatedPost}>Add Related Post</Button>
    </div>
  )
}
```

### Page with Bulk Operations

```tsx
function BulkPublish({ sdk }: { sdk: PageAppSDK }) {
  const [entries, setEntries] = useState([])
  const [publishing, setPublishing] = useState(false)

  const publishAll = async () => {
    setPublishing(true)
    const close = sdk.notifier.loading('Publishing entries...')

    try {
      const cma = createCMAClient(sdk)

      for (const entry of entries) {
        await cma.entry.publish(
          { entryId: entry.sys.id },
          { sys: entry.sys }
        )
      }

      close()
      sdk.notifier.success(`Published ${entries.length} entries`)
    } catch (error) {
      close()
      sdk.notifier.error('Failed to publish entries')
    } finally {
      setPublishing(false)
    }
  }

  return (
    <div>
      <h1>Bulk Publish</h1>
      <p>{entries.length} entries selected</p>
      <Button onClick={publishAll} isDisabled={publishing || entries.length === 0}>
        Publish All
      </Button>
    </div>
  )
}
```

## Best Practices

1. **Use typed SDK** - Import location-specific SDK types
2. **Handle dialog cancellation** - Check if result is null
3. **Show loading states** - Use loading notifier for async operations
4. **Check permissions** - Use sdk.access before operations
5. **Use slide-in navigation** - Better UX for entry/asset navigation
6. **Limit dialog size** - Keep dialogs focused and appropriately sized
7. **Provide feedback** - Use notifier for all user actions
8. **Handle errors** - Catch and display errors appropriately
9. **Close dialogs properly** - Always return a value or null
10. **Use CMA adapter** - Never expose management tokens
