---
title: "Building React Apps in Deno using Aleph.js and Ruck"
datePublished: Wed Nov 02 2022 03:22:43 GMT+0000 (Coordinated Universal Time)
cuid: clfvzbxhp000509ladev143cb
slug: building-react-apps-in-deno-using-alephjs-and-ruck
canonical: https://blog.logrocket.com/ruck-aleph-building-react-apps-deno/
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/Pihl8kTtX-s/upload/2c51b1fb82f1d369a9cd0e3eb48417b1.jpeg
tags: reactjs, deno, import-maps

---

Building a frontend in the modern era is tough because of the plethora of choices you must make. Developers will often reach for a popular framework, React, and find themselves needing more tools to get the job done. These could include a bundler, test runner, linter more. Not only that but they need to consider SEO, styling assets, routing, data-fetching, and the list goes on. Developers should consider these when creating a production-ready, performant React app. Projects like [create-react-app](https://create-react-app.dev/) and [Next.js](https://nextjs.org/) have gained popularity for providing features that were tedious to put in place on their own. Deno is a new JavaScript runtime that is gaining support from the community. Deno aligns with web standards by supporting [ES Modules](https://flaviocopes.com/es-modules/), [import maps](https://github.com/WICG/import-maps), and [the fetch API](https://deno.land/manual@v1.26.0/runtime/web_platform_apis#fetch-api). Most React frameworks today are only supported on Node.js but now some are released and built on Deno. Deno can do things these frameworks must put in place on their own while supporting Node.js. Due to Deno supporting ES Modules and TypeScript, frameworks can avoid building steps like transpilation. Deno has a large standard library, developer tools for common tasks like linting, formatting and testing, and a package manager. Some are weary of Deno because it not supporting NPM and not being compliant with all Node.js third-party packages. In my experience, there are many workarounds to these limitations. Ruck and Aleph.js are Deno-native React web frameworks and support features like server-side rendering, data-fetching, routing and modifying the HTTP server response. There are key similarities and differences with both Ruck and Aleph.js that are important to distinguish when choosing which one to use.

## Ruck

[Ruck](https://github.com/jaydenseric/ruck#installation) is a minimal framework for building React apps with Deno. It leans into Deno-specific features like ES Modules and import maps which makes it a great showcase for the new runtime. It doesn’t use a bundler so it does not support writing React components in JSX and all configuration defines itself in code. Using `createElement` everywhere is not the best developer experience. I could see another framework adopting Ruck under the hood to solve a lot of the problems for it. Ruck is what you are looking for if you want a framework in which you have control of what is going on and don’t like the “magic” of other frameworks.

An example component is written with Ruck:

```tsx
import { createElement as h } from "react";
import useOnClickRouteLink from "ruck/useOnClickRouteLink.mjs";
import useRoute from "ruck/useRoute.mjs";

export const css = new Set([
  "/components/ExampleComponent.css",
]);

export default function ExampleComponent({ href, children }) {
  return createElement("a", 
		{ className: "NavLink__a", href, onClick: () => console.log('Hello World!') },
		children
	);
}
```

## Aleph.js

Aleph.js is a full-stack web framework for building React apps with Deno. Second, to [Fresh](https://github.com/denoland/fresh), it is the most popular Deno-native React framework. It leans into Deno for some of its features but also provides much more. Aleph.js is inspired by Next.js even giving some of the same syntaxes for some features. Aleph.js supports server-side rendering as well as static-site generation, creating standalone APIs, file-base routing, and React Hot Module Reloading. To support separate file types such as JSX and CSS it doesn’t use webpack but instead uses [esbuild](https://github.com/evanw/esbuild).

An example component is written in Aleph.js:

```tsx
import React from 'react';
import Logo from '../components/logo.tsx'

export default function ExampleComponent() {
  return (
    <div>
      <Logo />
      <h1>Hello World!</h1>
    </div>
  )
}
```

## Similarities

There are similarities between Ruck and Aleph.js. One of those similarities is the support for [import maps](https://github.com/WICG/import-maps#the-basic-idea). Without the usage of NPM or another package manager, Deno depends on [HTTP imports](https://deno.land/manual/linking_to_external_code). This means imports usually look like this:

```tsx
import React from "https://esm.sh/stable/react@18.2.0/es2021/react.js”;
```

Deno recommends putting all module imports into a single `deps.ts` file to be re-exported. The issue with this approach is that imports are still not compatible with Node.js/webpack counterparts. A better way (and browser-compliant way) to do this is with import maps. Import maps are a recent browser feature that instructs the browser where dependencies for a module are located.

An example import map:

```tsx
{
  "imports": {
    "react": "https://esm.sh/stable/react@18.2.0/es2021/react.js",
  }
}
```

A component that uses the import map:

```tsx
import React from "react";

export default function ExampleComponent() {
  return <div />;
}
```

To use an import map in Aleph.js, one needs to define one file named `import_map.json` in the root directory. Using one in Ruck is also simple, define the file and pass it into Deno at runtime:

```tsx
deno run \
    --allow-env \
    --allow-net \
    --allow-read \
    --import-map=importMap.json \
    scripts/ruck-serve.mjs
```

The issue with import maps is that browser support is still poor with Safari and Firefox not supporting it out of the box. The good news is that Ruck uses a shim to provide support for older browsers.

Another similarity is their focus on server-side rendering (SSR) React components. SSR can provide performance, SEO and other benefits over client-side rendering. If a React component depends on fetched data, opting to do so on the server means a component can render before sending data to the client. This means no loading states to show to the user and generally better performance. Ruck supports data fetching on the server at a component level whereas other frameworks usually only support this at the page level. Aleph.js lets you define [a ssr function](https://alephjs.vercel.app/docs/basic-features/ssr-and-ssg) inside a page component file to achieve this. Aleph.js also supports a special hook, `useDeno`, for use in a component.

Example of using `useDeno` to fetch data on the server side in Aleph.js:

```tsx
import React from 'react'
import { useDeno, useRouter } from 'aleph'

export default function Post() {
  const { params } = useRouter()
  const post = useDeno(async () => {
    return await (await fetch(`https://.../post/${params.id}`)).json()
  })

  return (
    <h1>{post.title}</h1>
  )
}
```

When it comes to styling your React app with CSS, both Ruck and Aleph.js support component-level CSS imports. This allows for sending CSS to the browser that requests it (i.e. when a component renders). Ruck allows for this via an exported component variable named css. You can achieve the same behavior in [a variety of ways](https://alephjs.vercel.app/docs/basic-features/built-in-css-support) with Aleph.js but the recommended approach is to use [CSS modules](https://github.com/css-modules/css-modules).

Example of using the `css` function in Ruck:

```tsx
import React from 'react'
import Heading, { css as cssHeading } from "./Heading.mjs";
import Para, { css as cssParagraph } from "./Para.mjs";

export const css = new Set([
  ...cssHeading,
  ...cssParagraph,
  "/components/ExampleComponent.css",
]);

export default function ExampleComponent() {
   ...
}
```

Example of using a CSS module in Aleph.js:

```tsx
import React from 'react'
import styles from './exampleComponent.module.css'

export default function ExampleComponent() {
  return (
    <>
      <h1 className={styles.title}>Hi :)</h1>
    </>
  )
}
```

A perk of being a server-side rendered application is having access to the HTTP request during the rendering lifecycle. This can be helpful if you need to access headers or change the response. With Ruck, the HTTP response is available in a React context, `TransferContext`. In Aleph.js we can use the `ssr` function.

Example of modifying the HTTP response in Ruck:

```tsx
import React from 'react';
import TransferContext from "ruck/TransferContext.mjs";

export default function PageError({ errorStatusCode, title, description }) {
  const ruckTransfer = useContext(TransferContext);

  if (ruckTransfer) ruckTransfer.responseInit.status = errorStatusCode;

  ...
}
```

Example of modifying the HTTP response in Aleph.js:

```tsx
import React from 'react';
import { useDeno } from 'aleph';

export default function ExampleComponent() {
  const isLoggedIn = useDeno(req => {
    return req.headers.get('Auth') === 'XXX'
  }, { revalidate: true })

  return (
    <p>isLoggedIn: {isLoggedIn}</p>
  )
}
```

## Differences

There are notable differences between the two frameworks to be aware of. Popularity and developer experience are the two largest. Ruck is new so doesn’t have the community backing that a framework like Aleph.js has. By looking at [Deno X Ranking](https://yoshixmk.github.io/deno-x-ranking/), Aleph.js is the second most popular React framework by Github star count with 4.8k compared to Ruck with only 120. Star Count isn’t the best metric but it gives you a good idea about developer intent.

Ruck will favor the developers who like a high level of control over exactly how their application functions. Ruck has configuration set in code, for example routing you must define yourself while Aleph.js handles this for you. Aleph.js can be run with zero configs and has project templates to get developers started. You can opt-in to features based on config. In Ruck, you must spend time setting up the basics of the application yourself.

Static websites are desirable if your web application has all the data it needs at build time. This can simplify deployments as there needs to be no running Deno server. Place the built folder of HTML, CSS and JS to a deployment target like Github Pages or Cloudflare. [Aleph.js supports static-site generation](https://alephjs.vercel.app/docs/basic-features/ssr-and-ssg) which is helpful for these situations while Ruck does not. Like `getStaticPaths` in Next.js, you can define a paths key in the `ssr` function inside a component file to specify the paths this route can handle:

```tsx
import type { SSROptions } from 'aleph/types';

export const ssr: SSROptions = {
  paths: async () => {
    const posts = await (await fetch('https://.../api/posts')).json()
    return posts.map(({ id }) => `/post/${id}`)
  }
}
```

And then run `aleph build` , simple as that.

## Final Thoughts

With the popularity of Deno continuing to increase, Ruck and Aleph.js are two Deno-based React web frameworks catering to two different sets of developers. Ruck being a newcomer, doesn’t have the same level of polish Aleph.js has but offers more control. Aleph.js offers a great developer experience with zero config needed and lots of powerful features. These minimal frameworks bake in a lot of built-in modern browser features which can lead to a minimal and lean tech stack that contrasts a lot of the complexity in the frontend ecosystem seen today. Deno’s large amount of built-in features leads to less work being done by third-party tools. React frameworks can focus on developing innovative and interesting new features while developers are at ease knowing they made a great choice for their web application tech stack.