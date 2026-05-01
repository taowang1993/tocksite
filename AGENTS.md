# Tocksite Website

## Overview

The Official Website for Tocksite.

This repository is a Nuxt 4 marketing site with content-driven pages for the landing page, pricing, blog, changelog, docs, and auth demo flows. `public/logo.svg` is the shared brand asset for the logo and favicon.

## Tech Stack

| Tech                            | Use Case                   |
| ------------------------------- | -------------------------- |
| Nuxt 4                          | Routing, SSR, prerendering |
| Vue 3                           | Reactive UI                |
| TypeScript                      | Type safety                |
| Nuxt UI v4                      | App components             |
| Tailwind CSS v4                 | Styling                    |
| Nuxt Content v3                 | Markdown/YAML content      |
| nuxt-og-image                   | Social images              |
| @nuxt/image                     | Image handling             |
| @vueuse/nuxt                    | Shared composables         |
| Zod                             | Validation                 |
| Iconify (Lucide + Simple Icons) | Icons                      |
| better-sqlite3                  | Local content store        |
| ESLint / @nuxt/eslint           | Linting                    |
| GitHub Actions                  | CI checks                  |
| pnpm                            | Package manager            |

## Architecture

```text
Browser
  └─ Requests: /, /pricing, /blog, /docs, /changelog, /login, /signup
        │
        ▼
┌──────────────────────────────────────────────────────────────────────┐
│ Nuxt 4 Runtime (Nitro + Vue)                                         │
│ app/app.vue                                                          │
│ - <UApp> Root Shell                                                  │
│ - Global Head, SEO, Theme Color                                      │
│ - Docs Navigation Provider                                           │
│ - Client Content Search Bootstrap                                    │
└───────────────┬──────────────────────────────────────────────────────┘
                │
                │                          ┌──▶ Client-Only Features:
                │                          │    Search, Color Mode,
                │                          │    Animations, Toasts
                ▼                          │
┌──────────────────────────────────────────────────────────────────────┐
│ Layout Selection                                                     │
│ default -> Header + Footer                                           │
│ docs    -> Sidebar + Search + TOC                                    │
│ auth    -> Centered Card                                             │
└───────────────┬──────────────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────────────┐
│ Pages                                                                │
│ /            -> content/0.index.yml                                  │
│ /pricing     -> content/2.pricing.yml                                │
│ /blog        -> content/3.blog.yml + content/3.blog/**/*             │
│ /blog/[slug] -> queryCollection('posts') + surroundings              │
│ /docs/**     -> queryCollection('docs') + TOC + surroundings         │
│ /changelog   -> content/4.changelog.yml + content/4.changelog/**/*   │
│ /login, /signup -> UAuthForm Demo Flows                              │
└───────────────┬──────────────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────────────┐
│ Nuxt Content Collections                                             │
│ index | pricing | docs | blog | posts | changelog | versions         │
│ Markdown/YAML + Zod Schemas in content.config.ts                     │
│ Custom Shortcodes: Pictures, PictureAndText                          │
└──────────────────────────────────────────────────────────────────────┘
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
├── README.md
└── .context/
    └── architecture.md
```

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
