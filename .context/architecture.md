# Tocksite Website Architecture

This repository is a Nuxt 4, content-driven marketing site for Tocksite. The active brand uses `public/logo.svg` for the logo and favicon, while the underlying app keeps the Nuxt UI + Nuxt Content architecture from the template foundation.

## System Architecture

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
└───────────────┬──────────────────────────┬───────────────────────────┘
                │                          │
                │                          └──▶ Client-Only Features:
                │                               Search Overlay, Color Mode,
                │                               Animations, Toasts
                ▼
┌──────────────────────────────────────────────────────────────────────┐
│ Layout Selection                                                     │
│ default -> Header + Footer                                           │
│ docs    -> Sidebar + Search + TOC                                    │
│ auth    -> Centered Card                                             │
└───────────────┬──────────────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────────────┐
│ Pages (app/pages/)                                                   │
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

## Request And Render Flow

1. Nitro receives the request and Nuxt resolves a page file under `app/pages/`.
2. The page fetches content via `queryCollection(...)`, `queryCollectionNavigation(...)`, or `queryCollectionItemSurroundings(...)`.
3. A layout wraps the page in the correct shell from Nuxt UI.
4. Content-heavy pages render markdown with `ContentRenderer`, TOC links, and previous/next navigation.
5. Browser-only features hydrate after mount: the content search overlay, color mode, pricing tabs, auth forms, animated backgrounds, and toast notifications.
6. `useSeoMeta` sets page-specific metadata and `nuxt-og-image` generates shareable Open Graph images.

## Route Map

| Route | Page File | Content Source | Notes |
| --- | --- | --- | --- |
| `/` | `app/pages/index.vue` | `content/0.index.yml` | Landing page with hero, feature sections, testimonials, and CTA |
| `/pricing` | `app/pages/pricing.vue` | `content/2.pricing.yml` | Monthly/yearly pricing switch and FAQ |
| `/blog` | `app/pages/blog/index.vue` | `content/3.blog.yml` + posts | Blog listing page |
| `/blog/[slug]` | `app/pages/blog/[slug].vue` | `content/3.blog/**/*` | Article body, authors, date, TOC, and surround navigation |
| `/docs/**` | `app/pages/docs/[...slug].vue` | `content/1.docs/**/*` | Docs shell with sidebar navigation, search, TOC, and surrounding links |
| `/changelog` | `app/pages/changelog/index.vue` | `content/4.changelog.yml` + `content/4.changelog/**/*` | Reverse-chronological release notes |
| `/login` | `app/pages/login.vue` | UI-only | Auth demo form with validation |
| `/signup` | `app/pages/signup.vue` | UI-only | Auth demo form with validation |

## Content Model

- `content.config.ts` defines the canonical content schema with Zod.
- The app uses page-type collections for all public-facing sections, which keeps routing and content aligned.
- Numeric prefixes in folders and files control ordering, e.g. `1.getting-started`, `2.essentials`, `1.index.md`, `2.installation.md`.
- Docs navigation is driven by folder metadata (`.navigation.yml`) and page frontmatter (`navigation.icon`).
- Blog and changelog entries use frontmatter-heavy markdown so the pages can stay thin and mostly bind data to Nuxt UI components.
- Custom content components live in `app/components/content/` and are used directly from markdown, such as `::pictures` and `::picture-and-text`.

## Layouts And Shared Shell

- `app/app.vue` is the orchestration point. It sets global head tags, provides the docs navigation tree, lazy-loads the docs search index on the client, and wraps the app in `<UApp>`.
- `app/layouts/default.vue` is the public shell used by the home, pricing, blog, and changelog pages.
- `app/layouts/docs.vue` adds the docs aside, `UContentNavigation`, and the docs search button.
- `app/layouts/auth.vue` keeps login and signup centered inside a card.
- `app/error.vue` mirrors the public shell so 404 states still have navigation and search.
- `app/pages/blog.vue` is a thin route shell that keeps `/blog` and `/blog/[slug]` under the same namespace.

## UI And Design System

- Nuxt UI provides the component library used throughout the site.
- `app/assets/css/main.css` imports Tailwind CSS v4 and Nuxt UI, defines a small theme token set, and tweaks dark-mode background variables.
- `app/app.config.ts` sets the brand palette (`primary: blue`, `neutral: slate`).
- VueUse powers shared state such as color mode and loading indicators.
- `public/logo.svg` is the single brand asset for the logo and favicon.
- Decorative components like `HeroBackground`, `PromotionalVideo`, `ImagePlaceholder`, and `StarsBg` are presentation-only and help the marketing pages feel polished.

## Delivery And Operational Notes

- `nuxt.config.ts` prerenders `/` and crawls links, so the site behaves like a static-first application with selective client-side hydration.
- `/docs` redirects to `/docs/getting-started`.
- There are no custom API routes or server handlers in the repository; the login, signup, and newsletter flows are UI demos until real services are wired in.
- `.github/workflows/ci.yml` runs linting and type checking on every push.

## Rebranding Notes

The architecture is reusable as-is. For a Tocksite-specific launch, keep the Nuxt Content + Nuxt UI pipeline intact, replace template copy where needed, and continue using `public/logo.svg` for the logo and favicon.
