# Tocksite

## Overview

The Official Website for Tockbot.com.

This repository is a Nuxt 4 marketing site with content-driven pages for the landing page, pricing, blog, changelog, docs, and auth demo flows. `public/logo.svg` is the shared brand asset for the logo and favicon.

## Tech Stack

| Tech                            | Use case        |
| ------------------------------- | --------------- |
| Nuxt 4                          | routing / SSR   |
| Vue 3                           | UI              |
| TypeScript                      | type safety     |
| Nuxt UI v4                      | components      |
| Tailwind CSS v4                 | styling         |
| Nuxt Content v3                 | content         |
| nuxt-og-image                   | social images   |
| @nuxt/image                     | images          |
| @vueuse/nuxt                    | composables     |
| Zod                             | validation      |
| Iconify (Lucide + Simple Icons) | icons           |
| better-sqlite3                  | local store     |
| ESLint / @nuxt/eslint           | linting         |
| GitHub Actions                  | CI              |
| pnpm                            | package manager |
| nuxt-skill-hub                  | agent skills    |

## Architecture

```text
Build time
┌──────────────────────┐      ┌──────────────────────────┐
│ Developer / CI       │ ───▶ │ Nuxt prepare / skill hub │
│ install / lint /     │      │ generation               │
│ typecheck            │      └─────────────┬────────────┘
└──────────┬───────────┘                    │
           │                                ▼
           │                     ┌──────────────────────────┐
           │                     │ Generated skill files    │
           │                     │ .codex/skills/nuxt       │
           │                     │ .nuxt/skill-hub/nuxt     │
           │                     └──────────────────────────┘
           │
           ▼ HTTP/SSR
┌──────────────────────┐
│ Browser / crawlers   │
└──────────┬───────────┘
           ▼
┌──────────────────────────────────────────────────────────────┐
│ Nuxt 4 app                                                   │
│ app/app.vue                                                  │
│ layouts: default • docs • auth                               │
│ pages: landing • pricing • blog • changelog • docs • auth    │
└──────────┬────────────────────────────┬──────────────────────┘
           │ renders                    │ reads
           ▼                            ▼
┌──────────────────────────┐   ┌──────────────────────────────┐
│ Nuxt UI + Tailwind v4    │   │ Nuxt Content                 │
│ header/footer/forms      │   │ YAML / Markdown / Zod        │
└──────────┬───────────────┘   └───────────┬──────────────────┘
           └───────────────┬───────────────┘
                           ▼
                Shared services
                SEO • search • images • auth
```

- `app/app.vue` sets global head tags, provides docs navigation, and loads the search index on the client.
- `default`, `docs`, and `auth` layouts keep the public site, docs, and auth pages consistent.
- Pages stay thin: they fetch content with `queryCollection*` helpers and pass it into Nuxt UI components.
- `ContentRenderer`, `UContentToc`, and `UContentSurround` render docs, blog posts, and changelog entries.
- `public/logo.svg` powers both the header logo and the favicon.
- `nuxt-og-image` handles shareable social previews.
- Login, signup, and newsletter forms are demo-only until a backend is wired in.

## Project Structure

```text
.
├── app/
│   ├── app.vue
│   ├── app.config.ts
│   ├── assets/css/main.css
│   ├── components/
│   │   ├── AppFooter.vue
│   │   ├── AppHeader.vue
│   │   ├── AppLogo.vue
│   │   ├── HeroBackground.vue
│   │   ├── ImagePlaceholder.vue
│   │   ├── PromotionalVideo.vue
│   │   ├── StarsBg.vue
│   │   ├── TemplateMenu.vue
│   │   ├── content/
│   │   └── OgImage/
│   ├── error.vue
│   ├── layouts/
│   ├── pages/
│   └── types/
├── content/
│   ├── 0.index.yml
│   ├── 1.docs/
│   ├── 2.pricing.yml
│   ├── 3.blog/
│   └── 4.changelog/
├── public/
│   ├── logo.svg        # Logo + favicon
│   └── favicon.ico     # Legacy asset
├── content.config.ts
├── nuxt.config.ts
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
├── eslint.config.mjs
├── .github/
│   └── workflows/
│       └── ci.yml
├── .agents/
│   └── skills/
│       └── nuxt         # symlink to the stable wrapper
├── .codex/
│   └── skills/
│       └── nuxt/
│           └── SKILL.md  # stable wrapper for generated Nuxt guidance
├── .nuxtrc
├── README.md
└── .context/
    └── architecture.md
```

## Local Agent Skills

- Use the `nuxt` skill for Nuxt-specific work in this repo.
- `.agents/skills/nuxt` points to `.codex/skills/nuxt` so agent tooling finds the same entrypoint.
- `.codex/skills/nuxt/SKILL.md` is the stable wrapper that points at `.nuxt/skill-hub/nuxt/SKILL.md`.
- The generated content in `.nuxt/skill-hub/nuxt` is ephemeral; refresh it with `pnpm exec nuxt prepare` if it ever goes missing.

## Development Guidelines

- Use `pnpm install`, `pnpm dev`, `pnpm build`, `pnpm preview`, `pnpm lint`, and `pnpm typecheck`.
- Keep content in `content/` and update `content.config.ts` when schemas change.
- Fetch data with `queryCollection*` helpers instead of hardcoding copy in components.
- Reuse Nuxt UI components and shared layouts before adding custom CSS.
- Keep SEO and social metadata aligned with `useSeoMeta`, `defineOgImageComponent`, and `public/logo.svg`.
- Follow `.editorconfig`: 2-space indentation, LF endings, no trailing whitespace.
- Keep numeric prefixes in content paths so ordering stays deterministic.
- Treat auth, newsletter, and similar flows as demos until real services exist.
- When rebranding, use `public/logo.svg` for the logo and favicon and keep the content pipeline intact.
- When opening or updating a PR, use `.github/PR.md` as the source template for the PR body.
