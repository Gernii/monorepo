![Paraglide JS header image](https://cdn.jsdelivr.net/gh/inlang/monorepo@latest/inlang/source-code/paraglide/paraglide-js/assets/paraglide-js-header.png)

Attention: This package is in pre-release. It is not yet ready for production use.

# SolidStart Adapter for Paraglide JS

## Example project

Before setting up the adapter in your own project, you can take a look at the [example project](./example) to see how it works.

## Getting started

### 1. Initialize paraglide-js

If you haven't already, initialize paraglide-js in your project. To do so, follow the [getting started guide](https://github.com/inlang/monorepo/tree/main/inlang/source-code/paraglide/paraglide-js#getting-started) to get familiar with the basic paraglide concepts.

Then come back here to learn how the adapter helps you to integrate paraglide-js into your SolidStart project.

### 2. Install the adapter

```bash
npm install @inlang/paraglide-js-adapter-solidstart
# or
pnpm add @inlang/paraglide-js-adapter-solidstart
# or
yarn add @inlang/paraglide-js-adapter-solidstart
```

### 3. Use the adapter to wrap paraglide

Pass the runtime generated by paraglide to the adapter to create SolidStart-bound functions.

```ts
// i18n.tsx (or where you want to use the adapter)

import * as paraglide from "./paraglide/runtime.js" // generated by paraglide
import { createI18n } from "@inlang/paraglide-js-adapter-solidstart"

const { LanguageTagProvider, languageTag, setLanguageTag } = createI18n(paraglide)
```

Take a look at (./example/src/i18n/index.tsx) to see how the adapter is used in a example project. With an addition to some convenience functions.

### 4. Provide language tag to your app, and Solid Router.

```tsx
// root.tsx

import { Component, ErrorBoundary, Suspense } from "solid-js"
import { Body, FileRoutes, Head, Html, Routes, Scripts } from "solid-start"
import { LanguageTagProvider, useLocationLanguageTag } from "./i18n.ts"
import { sourceLanguageTag, availableLanguageTags } from "./paraglide/runtime.js"

const Root: Component = () => {
	// get language tag from URL, or use source language tag as fallback
	const url_language_tag = useLocationLanguageTag(availableLanguageTags)
	const language_tag = url_language_tag ?? sourceLanguageTag

	// 1. provide language tag to your app
	// 2. set html lang attribute
	// 3. make sure the routing doesn't treat the language tag as part of the path

	return (
		<LanguageTagProvider value={language_tag}>
			<Html lang={language_tag}>
				<Head />
				<Body>
					<ErrorBoundary>
						<Suspense>
							<Routes base={url_language_tag}>
								<FileRoutes />
							</Routes>
						</Suspense>
					</ErrorBoundary>
					<Scripts />
				</Body>
			</Html>
		</LanguageTagProvider>
	)
}
export default Root
```

Take a look at (./example/src/root.tsx) to see how it is used in a example project.

### 5. Use message functions

The language tag of current request is provided by the adapter via a context. All the messgee functions used with this context will be translated to the current language.

```tsx
// uses the context's language tag to translate the message
<h1>{m.greeting({ name: "Loris" })}</h1>

// pass language tag explicitly when used outside of the available context
const language_tag = languageTag() // in component body

<button onClick={e => {
	e.preventDefault()
	alert(
		m.greeting({ name: "Loris" }, { languageTag: language_tag })
	)
}}>
	{m.show_greeting()}
</button>
```

### 6. Switch language
You can switch languages by calling the `setLanguageTag` function provided by the adapter. This will navigate to the translated variant of the current route.

If you want to navigate to a different route in a specific language, you can use the `translateHref` function provided by the adapter to generate the correct href. 

```tsx
<A href={translateHref("/about", "en")}>{m.about()}</A>
```

> ⚠️ Don't use the `translateHref` function on links that point to external websites. It will break the link.