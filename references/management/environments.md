# Environments

Environments are isolated content workspaces within a space. Use them for staging, testing, and feature branches.

## Get Environment

```typescript
const env = await client.environment.get({
  spaceId: 'xxx',
  environmentId: 'staging',
})
```

## List Environments

```typescript
const environments = await client.environment.getMany({
  spaceId: 'xxx',
})

for (const env of environments.items) {
  console.log(env.name, env.sys.id, env.sys.status.sys.id)
}
```

## Create Environment

Creates a new environment by cloning from master (default) or a specified source:

```typescript
const newEnv = await client.environment.create(
  { spaceId: 'xxx' },
  {
    name: 'New Environment',
  }
)
```

## Create Environment with ID

```typescript
const newEnv = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'feature-branch',
  },
  {
    name: 'Feature Branch',
  }
)
```

## Clone from Specific Source

```typescript
const newEnv = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'feature-auth',
    sourceEnvironmentId: 'staging', // Clone from staging instead of master
  },
  {
    name: 'Authentication Feature',
  }
)
```

## Wait for Environment to be Ready

Environments are created asynchronously. Poll until ready:

```typescript
let env = await client.environment.get({
  spaceId: 'xxx',
  environmentId: 'new-env',
})

while (env.sys.status.sys.id !== 'ready') {
  console.log('Status:', env.sys.status.sys.id)
  await new Promise(resolve => setTimeout(resolve, 1000))

  env = await client.environment.get({
    spaceId: 'xxx',
    environmentId: 'new-env',
  })
}

console.log('Environment ready!')
```

## Update Environment

```typescript
const env = await client.environment.get({
  spaceId: 'xxx',
  environmentId: 'staging',
})

const updated = await client.environment.update(
  {
    spaceId: 'xxx',
    environmentId: 'staging',
  },
  {
    sys: env.sys,
    name: 'Updated Name',
  }
)
```

## Delete Environment

```typescript
await client.environment.delete({
  spaceId: 'xxx',
  environmentId: 'old-env',
})
```

## Environment Aliases

Environment aliases provide stable identifiers that can point to different environments.

### Get Environment Alias

```typescript
const alias = await client.environmentAlias.get({
  spaceId: 'xxx',
  environmentAliasId: 'master',
})

console.log('Points to:', alias.environment.sys.id)
```

### List Environment Aliases

```typescript
const aliases = await client.environmentAlias.getMany({
  spaceId: 'xxx',
})

for (const alias of aliases.items) {
  console.log(alias.sys.id, '->', alias.environment.sys.id)
}
```

### Update Environment Alias

Point an alias to a different environment:

```typescript
const alias = await client.environmentAlias.get({
  spaceId: 'xxx',
  environmentAliasId: 'master',
})

const updated = await client.environmentAlias.update(
  {
    spaceId: 'xxx',
    environmentAliasId: 'master',
  },
  {
    sys: alias.sys,
    environment: {
      sys: {
        type: 'Link',
        linkType: 'Environment',
        id: 'new-production-env',
      },
    },
  }
)
```

## Complete Workflow Example

```typescript
// 1. Create staging environment from master
const staging = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'staging',
    sourceEnvironmentId: 'master',
  },
  {
    name: 'Staging',
  }
)

// 2. Wait for it to be ready
let env = await client.environment.get({
  spaceId: 'xxx',
  environmentId: 'staging',
})

while (env.sys.status.sys.id !== 'ready') {
  await new Promise(resolve => setTimeout(resolve, 1000))
  env = await client.environment.get({
    spaceId: 'xxx',
    environmentId: 'staging',
  })
}

// 3. Make changes in staging
await client.entry.create(
  {
    spaceId: 'xxx',
    environmentId: 'staging',
    contentTypeId: 'blogPost',
  },
  {
    fields: {
      title: { 'en-US': 'Test Post' },
    },
  }
)

// 4. Test in staging...

// 5. Create production environment from staging
const prod = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'production-v2',
    sourceEnvironmentId: 'staging',
  },
  {
    name: 'Production V2',
  }
)

// 6. Wait for production to be ready
let prodEnv = await client.environment.get({
  spaceId: 'xxx',
  environmentId: 'production-v2',
})

while (prodEnv.sys.status.sys.id !== 'ready') {
  await new Promise(resolve => setTimeout(resolve, 1000))
  prodEnv = await client.environment.get({
    spaceId: 'xxx',
    environmentId: 'production-v2',
  })
}

// 7. Update alias to point to new production
const alias = await client.environmentAlias.get({
  spaceId: 'xxx',
  environmentAliasId: 'master',
})

await client.environmentAlias.update(
  {
    spaceId: 'xxx',
    environmentAliasId: 'master',
  },
  {
    sys: alias.sys,
    environment: {
      sys: {
        type: 'Link',
        linkType: 'Environment',
        id: 'production-v2',
      },
    },
  }
)

// 8. Clean up old environment
await client.environment.delete({
  spaceId: 'xxx',
  environmentId: 'old-production',
})
```

## Environment in Client Defaults

Use environment ID or alias in scoped clients:

```typescript
// Using environment ID
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    defaults: {
      spaceId: 'xxx',
      environmentId: 'staging',
    },
  }
)

// Using environment alias (recommended)
const client = contentful.createClient(
  { accessToken: 'xxx' },
  {
    type: 'plain',
    defaults: {
      spaceId: 'xxx',
      environmentId: 'master', // This is an alias
    },
  }
)
```

## Environment Status

Environment status values during creation:

- `queued` - Environment creation is queued
- `creating` - Environment is being created
- `ready` - Environment is ready to use
- `failed` - Environment creation failed

Check status:

```typescript
const env = await client.environment.get({
  spaceId: 'xxx',
  environmentId: 'new-env',
})

console.log(env.sys.status.sys.id) // 'ready', 'creating', etc.
```

## Best Practices

1. **Use environment aliases** - Avoid hardcoding environment IDs in applications
2. **Always poll after creation** - Environments take time to clone
3. **Clone from appropriate source** - Staging from master, production from staging
4. **Use descriptive names** - Makes management easier
5. **Clean up old environments** - Delete unused environments to save space
6. **Test in non-production** - Make changes in staging before production
7. **Use feature branches** - Create temporary environments for feature development
8. **Monitor status** - Check `sys.status.sys.id` before using a new environment
9. **Handle failed creation** - Check for `failed` status and retry if needed
10. **Limit environment count** - Most plans have environment limits

## Common Patterns

### Feature Branch Workflow

```typescript
// 1. Create feature branch from master
const feature = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'feature-new-layout',
    sourceEnvironmentId: 'master',
  },
  { name: 'New Layout Feature' }
)

// 2. Wait for ready
// ... polling code ...

// 3. Make changes in feature branch
// ... content changes ...

// 4. Test feature branch
// ... testing ...

// 5. Merge to staging (create staging from feature)
const staging = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'staging',
    sourceEnvironmentId: 'feature-new-layout',
  },
  { name: 'Staging' }
)

// 6. Clean up feature branch
await client.environment.delete({
  spaceId: 'xxx',
  environmentId: 'feature-new-layout',
})
```

### Blue-Green Deployment

```typescript
// 1. Create new production environment from staging
const prodNew = await client.environment.createWithId(
  {
    spaceId: 'xxx',
    environmentId: 'production-blue',
    sourceEnvironmentId: 'staging',
  },
  { name: 'Production Blue' }
)

// 2. Wait for ready
// ... polling ...

// 3. Update master alias to point to new production
const alias = await client.environmentAlias.get({
  spaceId: 'xxx',
  environmentAliasId: 'master',
})

await client.environmentAlias.update(
  {
    spaceId: 'xxx',
    environmentAliasId: 'master',
  },
  {
    sys: alias.sys,
    environment: {
      sys: {
        type: 'Link',
        linkType: 'Environment',
        id: 'production-blue',
      },
    },
  }
)

// 4. Keep old production for rollback
// ... after verification ...

// 5. Delete old production
await client.environment.delete({
  spaceId: 'xxx',
  environmentId: 'production-green',
})
```
