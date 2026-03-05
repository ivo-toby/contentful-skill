# Contentful API Skill for Claude Code

A Claude Code skill that provides comprehensive Contentful REST API reference — covering the Content Management API (CMA), Content Delivery API (CDA), Preview API, Images API, and GraphQL API. All examples use curl/HTTP, making them language-agnostic.

## What's a Skill?

Skills are curated reference files that Claude Code can read during conversations to give accurate, up-to-date guidance for specific tools and APIs. Instead of relying on training data (which may be outdated), Claude reads the skill files on demand and gives you answers grounded in the actual API documentation.

## What This Skill Covers

| API | Description | Reference |
|-----|-------------|-----------|
| **Content Management (CMA)** | Create/update content, manage content types, assets, environments | [references/content-management/](references/content-management/) |
| **Content Delivery (CDA)** | Fetch published content for production apps | [references/content-delivery/](references/content-delivery/) |
| **Content Preview** | Fetch draft + published content | [references/content-preview/](references/content-preview/) |
| **Images API** | On-the-fly image transformations via URL params | [references/images/](references/images/) |
| **GraphQL API** | Query content via GraphQL | [references/graphql/](references/graphql/) |
| **Authentication** | Token types, auth headers, API base URLs | [references/authentication.md](references/authentication.md) |
| **HTTP Conventions** | Version locking, rate limits, pagination, errors | [references/http-conventions.md](references/http-conventions.md) |

## Install

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated

### Add the Skill

From any project where you want Contentful API guidance available:

```bash
claude skill add --from https://github.com/ivo-toby/contentful-skill
```

This clones the skill into your project's `.claude/skills/` directory.

### Verify

Start a Claude Code session and ask something like:

```
How do I create an entry in Contentful?
```

Claude will read the relevant reference files and give you a curl-based answer with the correct headers, URL structure, and payload format.

### Update

To pull the latest version of the skill:

```bash
claude skill update contentful-api
```

## Project Structure

```
.
├── SKILL.md                          # Entry point — SDK routing & quick reference
├── references/
│   ├── authentication.md             # Token types, auth headers, US/EU base URLs
│   ├── http-conventions.md           # Version locking, rate limits, pagination, errors
│   ├── content-management/
│   │   ├── overview.md               # CMA quick start & core patterns
│   │   ├── entries.md                # CRUD, publish/unpublish, versioning
│   │   ├── content-types.md          # Content models, field types, validations
│   │   ├── assets.md                 # Upload, process, publish media
│   │   ├── environments.md           # Create, clone, manage environments & aliases
│   │   └── bulk-actions.md           # Bulk publish/unpublish/validate
│   ├── content-delivery/
│   │   ├── overview.md               # CDA quick start & authentication
│   │   ├── querying.md               # Filters, search operators, ordering
│   │   ├── includes-links.md         # Include parameter, link resolution
│   │   ├── localization.md           # Locale parameter, fallback chains
│   │   └── sync.md                   # Incremental content synchronization
│   ├── content-preview/
│   │   └── overview.md               # Preview API setup & usage
│   ├── images/
│   │   └── overview.md               # Image transformation URL parameters
│   └── graphql/
│       └── overview.md               # GraphQL queries with CDA tokens
└── CLAUDE.md                         # Development instructions (not part of the skill)
```

## Contributing

1. Fork the repo
2. Create a feature branch
3. Keep reference files under 500 lines
4. Add a table of contents to files over 100 lines
5. Use curl/HTTP examples (language-agnostic)
6. Open a PR

## License

MIT
