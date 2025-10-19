# Building Contentful SDK Skill - Instructions for Claude Code

## Context

You're building a comprehensive Contentful SDK skill that covers three main SDKs:

1. Management SDK (CMA) - creating/updating content, managing schema
2. Delivery SDK (CDA) - fetching published content
3. App Framework SDK - building Contentful apps

This skill follows the progressive disclosure pattern where SKILL.md is minimal (~50 lines) and detailed content lives in domain-specific reference files.

## Current State

The `/Users/ivo/workspace/contentful-skill` directory exists with:

- `SKILL.md` - Current monolithic version (needs to be split)
- `.git` - Already initialized

## Target Structure

```
contentful-skill/
├── SKILL.md (~50 lines: overview + SDK selection routing)
├── references/
│   ├── management/
│   │   ├── overview.md (quick start, client setup, core patterns)
│   │   └── [additional files based on documentation analysis]
│   ├── delivery/
│   │   ├── overview.md (client setup, authentication)
│   │   └── [additional files based on documentation analysis]
│   └── app-framework/
│       ├── overview.md (initialization, SDK setup)
│       └── [additional files based on documentation analysis]
└── scripts/ (only if needed for common operations)
```

**Structure Guidelines:**

- Each SDK subdirectory MUST have `overview.md` as the entry point
- Analyze the documentation to determine logical topic splits
- Keep each reference file under 500 lines
- Split by domain concepts, not by API endpoints
- Name files descriptively (e.g., `querying.md`, `authentication.md`, `content-types.md`)
- Aim for 3-7 topic files per SDK (plus overview.md)

## Task Breakdown

### Phase 1: Split Management SDK (Current Content)

1. **Analyze current SKILL.md** to identify logical domain groupings
2. **Create directory structure**: `references/management/`
3. **Determine topic splits** based on the existing content:
   - Look for major sections (Environments, Content Types, Entries, Assets, etc.)
   - Group related concepts together
   - Aim for 5-8 reference files plus overview.md
   - Each file should be focused on one domain concept
4. **Create `overview.md`**:
   - Quick start with client setup
   - Core patterns (version locking, locale structure, publish workflow)
   - Table of contents linking to all other management files
5. **Split content into topic files**:
   - Extract each major section into its own file
   - Add table of contents at top of each file if >100 lines
   - Ensure code examples are complete and runnable
6. **Suggested splits** (adapt based on actual content):
   - entries.md, content-types.md, assets.md, environments.md, error-handling.md
   - But analyze the content first and adjust as needed
7. Commit: "Split Management SDK into reference files"

### Phase 2: Build Delivery SDK

Documentation URL: https://www.contentful.com/developers/docs/references/content-delivery-api/

1. **Fetch and analyze** the Delivery SDK documentation
2. **Identify major topics** from the documentation:
   - Look for distinct concepts (querying, filtering, includes, preview, etc.)
   - Group related functionality together
   - Aim for 3-6 topic files plus overview.md
3. **Create directory structure**: `references/delivery/`
4. **Create `overview.md`**:
   - Client initialization patterns
   - Basic usage example
   - Authentication approaches
   - Table of contents linking to all delivery topic files
5. **Create topic files** based on documentation analysis:
   - Each file covers one major concept
   - Include practical examples
   - Focus on common use cases
   - Add table of contents if >100 lines
6. **Possible topics to look for** (adapt based on actual docs):
   - Querying and search
   - Link resolution and includes
   - Preview API
   - Localization
   - Content delivery patterns
7. Commit: "Add Delivery SDK documentation"

### Phase 3: Build App Framework SDK

Documentation URL: https://www.contentful.com/developers/docs/extensibility/app-framework/

1. **Fetch and analyze** the App Framework SDK documentation
2. **Identify major topics** from the documentation:
   - Look for distinct concepts (locations, UI components, SDK APIs, etc.)
   - Group related functionality together
   - Aim for 4-7 topic files plus overview.md
3. **Create directory structure**: `references/app-framework/`
4. **Create `overview.md`**:
   - SDK initialization (`init` pattern)
   - Basic app structure
   - Available SDK capabilities overview
   - Table of contents linking to all app-framework topic files
5. **Create topic files** based on documentation analysis:
   - Each file covers one major concept
   - Include practical examples for each location/feature
   - Focus on integration patterns
   - Add table of contents if >100 lines
6. **Possible topics to look for** (adapt based on actual docs):
   - App locations (where apps can render)
   - SDK methods and APIs
   - Parameters and configuration
   - CMA integration
   - UI components and extensions
   - State management
7. **Note**: If there's App SDK Integration content in the current SKILL.md, move it to the appropriate file here
8. Commit: "Add App Framework SDK documentation"

### Phase 4: Create Final SKILL.md

Create a new minimal SKILL.md (~50-70 lines) that dynamically reflects the actual reference structure created.

**Template structure**:

```markdown
---
name: contentful-sdk
description: Comprehensive Contentful SDK guide for TypeScript/JavaScript. Covers Management SDK (CMA) for content/schema management, Delivery SDK (CDA) for fetching content, and App Framework SDK for building Contentful apps. Use for any Contentful API integration work.
---

# Contentful SDK Guide

Comprehensive guide for Contentful SDKs in TypeScript/JavaScript.

## Which SDK Do You Need?

- **Management SDK (CMA)**: Creating/updating content, managing content types, assets, environments → Start at [references/management/overview.md](references/management/overview.md)
- **Delivery SDK (CDA)**: Fetching published content for production apps → Start at [references/delivery/overview.md](references/delivery/overview.md)
- **App Framework**: Building Contentful Apps that extend the UI → Start at [references/app-framework/overview.md](references/app-framework/overview.md)

## Management SDK (CMA)

For creating, updating, and managing content, content types, assets, and environments.

**Start here**: [references/management/overview.md](references/management/overview.md)

**Topics**:
[LIST ALL ACTUAL FILES IN references/management/ except overview.md, with brief descriptions]

## Delivery SDK (CDA)

For fetching published content in production applications.

**Start here**: [references/delivery/overview.md](references/delivery/overview.md)

**Topics**:
[LIST ALL ACTUAL FILES IN references/delivery/ except overview.md, with brief descriptions]

## App Framework SDK

For building apps that extend the Contentful UI.

**Start here**: [references/app-framework/overview.md](references/app-framework/overview.md)

**Topics**:
[LIST ALL ACTUAL FILES IN references/app-framework/ except overview.md, with brief descriptions]

## Quick Reference

[2-3 key patterns that apply across SDKs]
```

**Instructions**:

1. List the actual files you created in each section
2. Add 1-line descriptions for each file
3. Keep total length under 70 lines
4. Ensure all links point to files that actually exist
5. Add a "Quick Reference" section with 2-3 universal patterns

Commit: "Create final SKILL.md with navigation"

### Phase 5: Validation

1. Verify all reference files are <500 lines
2. Ensure files >100 lines have table of contents at top
3. Check that SKILL.md is ~50 lines
4. Verify all links in SKILL.md point to existing files
5. Test that progressive disclosure works: SKILL.md → overview.md → specific topic
6. Commit: "Final validation and cleanup"

## Important Guidelines

### Content Extraction

- Focus on **practical patterns** and **working code examples**
- Extract the "how-to" not the "what is"
- Prioritize TypeScript examples
- Keep error handling patterns
- Include version/deprecation notes if present

### File Size Management

- Each reference file should be <500 lines ideally
- If a file approaches 500 lines, consider splitting it further
- Add table of contents for files >100 lines

### Writing Style

- Concise and direct
- Code examples over prose
- Imperative tone ("Use X for Y", not "You can use X for Y")
- Minimal explanations (Claude is already smart)
- Focus on patterns, not exhaustive API coverage

### Progressive Disclosure

- SKILL.md: Overview + navigation only
- overview.md: Quick start + links to detailed topics
- Topic files: Deep dives with examples

### Version Control

- Commit after each major phase
- Use descriptive commit messages
- This allows rollback if needed

## Documentation URLs

Management SDK: Already have the content in current SKILL.md

Delivery SDK: [USER TO PROVIDE]

App Framework SDK: [USER TO PROVIDE]

## Success Criteria

✅ SKILL.md is ~50 lines with clear SDK navigation
✅ All three SDKs fully documented in references/
✅ Each overview.md has table of contents
✅ No file exceeds 500 lines
✅ All examples use TypeScript
✅ Progressive disclosure pattern followed
✅ Git history shows clear progression

## Notes

- You have the skill-creator skill available - use it for guidance on structure
- Current SKILL.md has good content, just needs splitting
- Prioritize code examples and patterns over prose
- Management SDK content is already solid, mainly needs reorganization
- Be critical: if documentation is unclear or missing key info, note it in comments
