# Documentation Platform Migration Notes

*Date: February 11, 2026*

## Context

The `multitechsystems.github.io` GitHub Pages site was a pilot for migrating away from the WordPress-based `multitech.net/developer/` site. The pilot proved out markdown-based content and a versioned URL structure (`/v1/`, `/v2/`).

## Current State

- **Legacy site**: https://www.multitech.net/developer/ — WordPress, 300+ pages, deep hierarchy (Products, Software, mLinux, mDot/xDot libraries, LoRaWAN, DeviceHQ API)
- **Pilot site**: https://multitechsystems.github.io — Jekyll on GitHub Pages, ~40 pages, versioned (`/v1/` guides-only snapshot, `/v2/` full docs with API schemas)
- **Pain point**: WordPress UI for article writing is painful compared to markdown

## Why Move Away from GitHub Pages

- No built-in search (hand-rolled JSON index doesn't scale to 300+ pages)
- No CMS for non-developer contributors
- No access control (everything public)
- Single-site limit per GitHub org
- Versioning had to be built manually (redirect stubs, version-aware sidebar, etc.)

## Recommended: MkDocs Material

- **Markdown-native** — write in VS Code/Cursor, git workflow
- **Built-in search** — lunr.js, handles hundreds of pages well
- **Versioning** — `mike` plugin provides version dropdown and `/vN/` paths out of the box
- **Navigation** — auto-generated or explicit in `mkdocs.yml`
- **Material theme** — polished, responsive, dark mode, admonitions, tabs, code highlighting
- **Deploy anywhere** — Netlify, Cloudflare Pages, GitHub Pages, or self-hosted
- **PDF export** — plugin available

### Migration Path

1. Existing markdown files move almost unchanged
2. `mkdocs.yml` replaces Jekyll `_config.yml` + navigation YAML files
3. Custom sidebar/layout HTML goes away — Material theme handles it
4. `mike deploy v2` replaces manual versioning setup
5. Could host at `docs.multitech.net` via Cloudflare Pages for professional URL

### Other Options Considered

| Platform | Pros | Cons |
|----------|------|------|
| **Docusaurus** | React-based, Algolia search, versioning | Heavier, Node.js ecosystem |
| **Read the Docs** | Free OSS hosting, PDF, versioning | Less control over styling |
| **GitBook** | Polished UI, good for mixed teams | Commercial, less git-native |
| **Keep WordPress** | Already works, CMS for non-devs | Writing experience is the problem |

## Decision

Pending review. No action taken yet.
