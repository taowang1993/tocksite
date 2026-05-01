<script setup lang="ts">
const colorMode = useColorMode()

const color = computed(() => colorMode.value === 'dark' ? '#020618' : 'white')

const themeAwareFavicons = useState('tocksite-theme-favicons', () => ({
  dark: false,
  light: false,
}))

if (import.meta.server) {
  const [{ existsSync }, { resolve: resolvePath }] = await Promise.all([
    import('node:fs'),
    import('node:path'),
  ])

  const publicDir = resolvePath(process.cwd(), 'public')

  themeAwareFavicons.value = {
    dark: existsSync(resolvePath(publicDir, 'favicon-dark.svg')),
    light: existsSync(resolvePath(publicDir, 'favicon-light.svg')),
  }
}

const faviconLinks = computed(() => {
  const links: Array<{ key: string, rel: string, href: string, type: string, media?: string, sizes?: string }> = []

  if (themeAwareFavicons.value.dark) {
    links.push({
      key: 'favicon-dark',
      rel: 'icon',
      href: '/favicon-dark.svg',
      type: 'image/svg+xml',
      media: '(prefers-color-scheme: light)',
      sizes: 'any',
    })
  }

  if (themeAwareFavicons.value.light) {
    links.push({
      key: 'favicon-light',
      rel: 'icon',
      href: '/favicon-light.svg',
      type: 'image/svg+xml',
      media: '(prefers-color-scheme: dark)',
      sizes: 'any',
    })
  }

  links.push({
    key: 'favicon',
    rel: 'icon',
    type: 'image/svg+xml',
    href: '/logo.svg',
    sizes: 'any',
  })

  return links
})

useHead({
  meta: [
    { charset: 'utf-8' },
    { name: 'viewport', content: 'width=device-width, initial-scale=1' },
    { key: 'theme-color', name: 'theme-color', content: color }
  ],
  link: faviconLinks,
  htmlAttrs: {
    lang: 'en'
  }
})

useSeoMeta({
  titleTemplate: '%s - Tockbot',
  twitterCard: 'summary_large_image'
})

const { data: navigation } = await useAsyncData('navigation', () => queryCollectionNavigation('docs'), {
  transform: data => data.find(item => item.path === '/docs')?.children || []
})
const { data: files } = useLazyAsyncData('search', () => queryCollectionSearchSections('docs'), {
  server: false
})

const links = [{
  label: 'Docs',
  icon: 'i-lucide-book',
  to: '/docs/getting-started'
}, {
  label: 'Pricing',
  icon: 'i-lucide-credit-card',
  to: '/pricing'
}, {
  label: 'Blog',
  icon: 'i-lucide-pencil',
  to: '/blog'
}, {
  label: 'Changelog',
  icon: 'i-lucide-history',
  to: '/changelog'
}]

provide('navigation', navigation)
</script>

<template>
  <UApp>
    <NuxtLoadingIndicator />

    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>

    <ClientOnly>
      <LazyUContentSearch
        :files="files"
        shortcut="meta_k"
        :navigation="navigation"
        :links="links"
        :fuse="{ resultLimit: 42 }"
      />
    </ClientOnly>
  </UApp>
</template>
