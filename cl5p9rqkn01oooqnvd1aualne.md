## Using static site generation in Next.js, Gatsby.js, and Remix

If you are writing a web application in 2022, you are likely using modern frontend technologies like [React](https://reactjs.org/), [Vue](https://vuejs.org/), and [Svelte](https://svelte.dev/). You are also likely using an API to get the data necessary to render pages. 

Using API network requests is easily one of the slowest steps required to render pages, and a slow-running app can mean a poor user experience. [Having great performing pages](https://blog.logrocket.com/optimizing-performance-react-application/) can also improve your [search engine optimization](https://en.wikipedia.org/wiki/Search_engine_optimization) (SEO) dramatically. 

If you own the API and know how to make it faster, great! But if not, you don’t have control over the performance and speed of your app. You may even be using [server-side generation](https://vercel.com/blog/nextjs-server-side-rendering-vs-static-generation), which passes data fetching and page rendering to the server, and yet, your app still may not be fast enough. This is where static site generation comes into play. There are different ways to implement SSG, but first an explanation into how it works.


## What is static site generation?

One of the ways software developers can optimize their applications is by doing the work ahead of time or by using caching. 

Static site generation is the process of building pages (pre-rendering) into static assets and serving them to users instead of doing it per request, especially when our data is static or doesn’t change often. We can also create as many builds as we want, and most of this work can be done on a hosted server, making the process easy.

Thankfully, there are many static site generation tools based on React.js, with some of the most interesting ones being [Gatsby](https://www.gatsbyjs.com/), [Next.js](https://nextjs.org/), and [Remix.js](https://www.gatsbyjs.com/https://remix.run/). All three achieve [serving performant web applications in their own way](https://blog.logrocket.com/react-remix-vs-next-js-vs-sveltekit/). 

For example, Gatsby has a great ecosystem of plugins to get you started quickly. Next.js is flexible, allowing you to opt into SSG on a per-page basis. [Remix.js](https://blog.logrocket.com/remix-guide-newly-open-sourced-react-framework/), while it doesn’t currently support SSG in the traditional sense, strives for page performance through caching assets on edge servers.

After understanding how Gatsby, Next.js, and Remix achieve performance through caching and SSG, choosing the best one for the job becomes easier, and we can take the techniques used and apply them to other frameworks and tools.

## Using Gatsby for static site generation

Gatsby is a React-based framework made for building statically generated web applications. It is one of the most popular ways developers perform SSG on the web today and has a huge community and [plugin](https://www.gatsbyjs.com/plugins/) [](https://www.gatsbyjs.com/plugins/)[ecosystem](https://www.gatsbyjs.com/plugins/). 

Here’s how it works. Developers write functions in Node.js to fetch data, create pages, and fill them with content. Matching these pages with React components (Gatsby calls these “templates”) is how Gatsby knows what to render and when. 

Builds can be created on a developer’s local machine but also on many SaaS hosting solutions like Gatsby Cloud, Netlify, and others. Once a build is complete, it can then be deployed to a CDN for ultra-fast serving to users.

![Example Gatsby architecture](https://paper-attachments.dropbox.com/s_C0546FE5A8F64775D8E87F12FEE36C2DA7164D789CC627F62DE935B22BA16C10_1648729126619_gatsby-deployment.drawio.png)


One of the best aspects of Gatsby is that its developer ecosystem is extensive. It has data-source plugins that make it incredibly easy to fetch data from an external API (take Shopify, for example) and image optimization plugins, just to name a few. 

Gatsby seems to have a plugin for everything — over 2,000 and counting! Not only does it have several plugins to choose from, but the documentation is extensive and there are many [starter templates to take inspiration from](https://www.gatsbyjs.com/starters/). 

With Gatsby, you get a very quick website by default while you’re developing at a quick rate. If you are building a frontend for a CMS or creating a commerce backend, Gatsby is a great choice.

Here is an example of the code used to generate pages in Gatsby:


    exports.createPages = async function ({ actions }) {
      const { data } = await graphql(`
        query {
          allMarkdownRemark {
            nodes {
              fields {
                slug
              }
            }
          }
        }
      `)
      data.allMarkdownRemark.forEach(node => {
        const slug = node.fields.slug
        actions.createPage({
          path: slug,
          component: require.resolve(`./src/templates/blog-post.js`),
          context: { slug: slug },
        })
      })
    };

Of course, all frameworks have their downsides, and Gatsby is no exception to this rule. Here are some cons to using Gatsby.

### Cons to using Gatsby for static site generation


- Server-side rendering (SSR) is relatively new to Gatsby, so the docs aren’t thorough
- The build process is tough to debug
- Devs must use GraphQL to get data into React components, which may make the learning curve steep for some
- Many aspects of Gatsby, including data fetching, are Gatsby-framework specific, so it’s challenging to switch to a different framework in the future
- Builds can become slow depending on the amount of data processing you need — like most SSG-based technologies, the more your data increases, the number of pages to create and images to process also increase

Considering the trade-offs, however, Gatsby can still be a great choice for your project.

## Static site generation with Next.js

[Next.js is a React-based framework](https://blog.logrocket.com/creating-website-next-js-react/) for building both dynamically and statically rendered web applications. It’s very popular and has many developers using it daily. 

There is now a relatively new feature that added SSG support to Next.js, with the unique proposition that it can be done on a per-page basis. The build process is similar to traditional SSG frameworks like Gatsby, but you can choose the pages the SSG occurs on.

Developers also have the choice of using incremental static generation (ISR), which is an advanced method that builds pages at runtime if they are not present or need to be rebuilt due to developer-set invalidation. 

If you have only chosen to use SSG for your pages, deployment is similar to Gatsby and can be done with a variety of vendors. It uses Node.js to fetch data and build your pages. If you use [any of the SSG unsupported features](https://nextjs.org/docs/advanced-features/static-html-export#unsupported-features), such as using a server to fetch data and build pages at runtime, you’ll have a more involved deployment process.

![Example Next.js architecture](https://paper-attachments.dropbox.com/s_C0546FE5A8F64775D8E87F12FEE36C2DA7164D789CC627F62DE935B22BA16C10_1648729160089_next-deployment.drawio.png)


One of the nicest aspects of Next.js is that it gives you the freedom to do what you want. As mentioned, you get the choice on a per-page basis to use SSG, SSR, or the new method, ISR. Better yet, it doesn’t prescribe anything like GraphQL when it comes to getting the data into your React components.

Here is an example of the code required for Next.js to statically generate a page:

    function Blog({ posts }) {
      return (
        <ul>
          {posts.map((post) => (
            <li>{post.title}</li>
          ))}
        </ul>
      )
    }
    
    export async function getStaticProps() {
      const res = await fetch('<https://example.com/posts>')
      const posts = await res.json()
    
      return {
        props: {
          posts,
        },
      }
    }
    
    export default Blog

While Next.js is a great option, it’s not for everyone.

### The cons to using Next.js for static site generation


- The freedom it allows can be daunting for new developers who don’t have much experience writing web applications 
- Unlike Gatsby, there aren’t any out-of-the-box plugins, so there is boilerplate code you may need to write if the data you are fetching is from a popular vendor like Shopify or WordPress
- The deployment and infrastructure can be cumbersome if you mix and match features and don’t use the Vercel, the company behind Next.js, deployment solution. 

However, if you want the freedom Next.js provides and don’t mind using Vercel’s deployment platform, Next.js is a great choice for web applications that need SSG.

## Using Remix.js with modern React tools

Remix.js is a React-based framework for building primarily dynamically rendered web applications. It’s a very new framework to many, as it was closed-source until recently.

Remix is interesting because it does not provide an option to pre-compile pages into a static asset bundle as Gatsby or Next.js do. There is no concept of SSG or “builds”, data-fetching is on-demand by default and HTML is compiled on the server per request (SSR). 

[Instead Remix depends on distributed computing to perform the same features one would get with SSG.](https://remix.run/docs/en/v1/guides/performance) The Remix server is compatible with edge computing vendors, such as Cloudflare Workers and [Fly.io](http://Fly.io). 

The server that fetches the data and compiles the HTML is deployed to these edge servers close to your users, resulting in much faster response times than a traditional web application setup. Our data, by default, is still fetched per request, which may be still too slow for your needs. The Remix teams recommends you create custom cache code for this purpose. 

There are edge server compatible databases that are usually in-memory, like SQLite, Redis, and LRU caches. When the data changes, you can evict data inside these caches manually or automatically, just like you would trigger a new build in a traditional SSG setup. 

It’s also recommended to use [HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) and serve static content on a CDN to serve web pages where the data doesn’t change often. This means you’ll get similar performance to a framework that uses SSG without it being coupled to a specific framework or having to worry about builds.

![Example Remix.js architecture](https://paper-attachments.dropbox.com/s_C0546FE5A8F64775D8E87F12FEE36C2DA7164D789CC627F62DE935B22BA16C10_1648729207919_remix-deployment.drawio.png)


Like Next.js, Remix gives the developer more freedom than prescribed frameworks like Gatsby when it comes to providing a fast web application experience. One of the great aspects of Remix.js is that it relies heavily on platforms like edge computing and browsers to accomplish most of its features. 

This means when you learn Remix, you learn more about the web, not how to develop in a specific framework. There is a surprisingly large amount of deployment options for Remix despite being new, so you can shop around for the best one.

Here is an example of a React component that fetches data and compiles the HTML on an edge server and displays it on the page:


    export const loader: LoaderFunction = async ({ request }) => {
      const userId = await requireUserId(request);
      const noteListItems = await getNoteListItems({ userId });
      return json<LoaderData>({ noteListItems });
    };
    
    export default function NotesPage() {
      const data = useLoaderData() as LoaderData;
    
      return data.noteListItems.map((note) => (
        <li key={note.id}>
          <NavLink to={note.id}>📝 {note.title}</NavLink>
        </li>
      ));
    }

### The cons of using Remix

- Like Next.js, the freedom Remix provides can be daunting for developers just starting, but the documentation is great so far, so you can learn what it has to offer
- Creating custom caching mechanisms for your web content could result in boilerplate code, which isn’t great
- The framework is still new and just now providing examples of how to perform data-caching on the server

In general, Remix is a great option if you aren’t afraid to take on a relatively new option in the space.

## Conclusion

There is plenty to consider when choosing between Gatsby, Next.js, and Remix.js to build your application. Which one you should choose depends on your ideal setup, your experience level, the vendor you want to host on, and how much code you want to write. No matter what you choose, all three options provide ways to serve web pages fast to your users.

