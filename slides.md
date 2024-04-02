---
theme: ./theme
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
title: Performant Websites with Remix
layout: intro
---

# Performant Websites with Remix

<!--
These are some techniques to make your website or web app load and run faster. 

These techniques are inherent to how the web and browsers work. 

We’ll talk about the technique, why it works, and which levers you can adjust in Remix to take advantage of these technique in your Remix app or site.
-->

---
layout: presenter
presenterImage: /profile.jpg
---

# Hi! I'm Mark 👋

I'm a UX designer and developer who makes exploratory prototypes with code. My pronouns are they/them.

<div class="flex flex-row items-center gap-3">

<img src="/mastodon.svg" alt="mastodon-logo" class="w-10 h-10">

[@markmalstrom@universeodon.com](https://universeodon.com/@markmalstrom)

</div>

<div class="flex flex-row items-center gap-3">

<picture>
    <source srcset="/github-dark.svg" media="(prefers-color-scheme: dark)">
    <img src="/github.svg" class="w-10 h-10">
</picture>

https://github.com/markmals

</div>

<!--
I'm a UX designer and developer who makes exploratory prototypes with code. My pronouns are they/them.
-->

---
transition: none
---

## Holotypes

<div class="flex flex-col gap-4">
    <Holotype 
        emojiIcon="🤳" 
        title="Social Media Applications"     
        holotype="Instagram"
        examples="YouTube, Twitter"
        characteristics="rich media, infinite scrolling content, user contribution, realtime updates, notifications, embeddability, embedded content"
        constraints="extended session depth, realtime updates, resource contention from embedded content, uninterruptible media playback, SEO"
        idealImplementation="Single-Page Application with app shell prerendering & caching."
        idealDelivery="PWA in standalone display mode."
    />
    <Holotype 
        emojiIcon="🛍" 
        title="Storefronts"     
        holotype="Amazon"
        examples="BestBuy, Newegg, Shopify(-based stores)"
        characteristics="search, payments, discoverability, filtering & sorting"
        constraints="shallow to medium session depth, small interactions, high cart/checkout dropoff, SEO"
        idealImplementation="Server-rendered site with CSR/SPA takeover or turbolinks-style transitions."
        idealDelivery="PWA in default display mode."
    />
</div>

<!--
I do want to say that these techniques aren’t always applicable or necessary for every website or web app. 

Every website has its own constraints that may dictate something unchangeable about its architecture, how it’s loaded, where it needs to run, where and how its data can be fetched, and more. 

There’s this great blog post by Jason Miller titled “Application Holotypes: A Guide to Architecture Decisions” that breaks down the major website categories you’ll see across the modern web landscape and talks about the ideal strategies for implementing them.

The techniquies I'm going to talk about apply more to content-heavy sites with sprinkles of app-like interactions — things like marketing sites, social media sites, reference and index sites, storefronts, and news sites.
-->

---

## Holotypes

<div class="flex flex-col gap-4">
    <Holotype 
        emojiIcon="🎨" 
        title="Graphical Editors"     
        holotype="Figma"
        examples="AutoCAD, Tinkercad, Photopea, Polarr"
        characteristics="3D rendering & GPU, image manipulation, fullscreen & pointer capture, MDI, storage, offline, filesystem, threads, wasm"
        constraints="long session length, sensitivity to input & rendering latency, large objects/files"
        idealImplementation="Single Page App. Separate lighter browsing UI from editor."
        idealDelivery="PWA in standalone display mode."
    />
    <Holotype 
        emojiIcon="🎮" 
        title="Immersive / AAA Games"     
        holotype="Stadia"
        examples="Heraclos, Duelyst, OUIGO"
        characteristics="3D rendering & GPU, P2P, audio processing, fullscreen & storage, offline, threads, device integration (gamepad), wasm"
        constraints="long session length (highly interactive), extremely sensitive to input and rendering latency, requires consistent FPS, extreme asset sizes"
        idealImplementation="Single Page App"
        idealDelivery="PWA in fullscreen display mode."
    />
</div>

<!--
As you get farther down the list, the holotypes get more and more interactive and these techniques apply less and less.

For web apps with heavy client side interactions that require things like WebAssembly and WebGL rendering, such as Figma or gaming, none of these techniques may be right. 
-->

---

## What Are We Optimizing For?

<v-clicks class="text-2xl mt-4">

* Simplicity
* Discoverability (SEO)
* Initial page load

</v-clicks>

---

## How Do We Achieve These Goals?

<v-clicks class="text-2xl mt-4">

1. Do as much work on the server as possible
2. Put your servers as close to your users as possible
3. Send as little data over the network as possible
4. Ship HTML to your users
5. Make pages work before JavaScript loads
6. Embrace web standards and browser defaults

</v-clicks>

---

## Do as much work on the server as possible

<div class="flex flex-col p-3 items-center">
    <img src="/wpt-hkg-search-3G.gif" width="500">
</div>

* If you fetch your data on the client, you can't load images until it loads data, you can't load data until you load JavaScript, and you can't load JavaScript until the document loads

<v-clicks>

* The user's network is a multiplier for every single step in that chain 😫
* Always fetching on the server removes the user's network as a multiplier everywhere else

</v-clicks>

<!--
When you do your work on the server, you can start fetching immediately when a request is received

You don't have to wait for the browser to download the document and then the JavaScript

It doesn't matter how slow the user's network is, your server's network is always consistent
-->

---

## Remix on the Server

<div class="flex flex-row gap-4 w-full">

<div class="w-full">

```ts
// Loaders only run on the server and provide data
// to your component on GET requests
export async function loader() {
    return json(await db.projects.findAll());
}
```

</div>
<div class="w-full">

```ts
// Actions only run on the server and handle POST
// PUT, PATCH, and DELETE. They can also provide data
// to the component
export async function action({ request }) {
    const form = await request.formData();
    const errors = validate(form);
    if (errors) {
        return json({ errors });
    }
    await db.projects.create({ title: form.get("title") });
    return json({ ok: true });
}
```

</div>
</div>

<!-- Remix fetches on the server by default -->

---

## Put your servers as close to your users as possible

<v-clicks>


* Long running servers and serverless functions tend to run their code in a single region

<div>

* Depending on your users, the one region could be a long way from where they are, which could mean a long time before the server can start fetching the data

</div>
<div>

* By utilizing an edge network, you can deploy your site on servers in dozens of regions around the world:


<div class="flex flex-col p-3 items-center">
    <img src="https://www.cloudflare.com/network-maps/cloudflare-pops-2O04nulSdNrRpJR9fs9OKv.svg" width="450" class="dark:bg-slate-200 p-4">
</div>
</div>

</v-clicks>

<!-- Maybe yopu can't deploy to the edge if you have certain restrictions, like in education where data must be centralized (but maybe you can still render at the edge?) -->

---

## Remix at the Edge

<v-clicks>

<div>

* Adapters!

```ts
import { createRequestHandler, handleAsset, } from "@remix-run/cloudflare-workers";
import * as build from "../build";

const handleRequest = createRequestHandler({ build });

const handleEvent = async (event: FetchEvent) => {
    let response = await handleAsset(event, build);
    if (!response) response = await handleRequest(event);
    return response;
};

addEventListener("fetch", (event) => {
    event.respondWith(handleEvent(event));
});
```

</div>

<div>

* You can deploy your entire site to the edge with something like Cloudflare Workers or Pages, or...

</div>

</v-clicks>

<!--
Because Remix is built on web standards, adapters can be built for almost any JS runtime

Including most edge runtimes, it's easy to deploy Remix to the edge
-->

---

* You can utilize hybrid options with something like Vercel's config export:

<div class="flex flex-col small">
<div class="flex flex-row gap-2">

`app/routes/concerts.tsx`

`/concerts`

</div>


````md magic-move
```tsx
export const config = { runtime: 'edge' };
export function loader() { ... }
 
export default function Component() {
    const { featured } = useLoaderData();
    return (
        <>
            <h1>Concerts</h1>
            <p>Featured concert: {featured}</p>
            <Outlet />
        </>
    );
}
```
```tsx
export const config = { runtime: 'nodejs' };
export function loader() { ... }
 
export default function Component() {
    const { featured } = useLoaderData();
    return (
        <>
            <h1>Concerts</h1>
            <p>Featured concert: {featured}</p>
            <Outlet />
        </>
    );
}
```
````

<div class="flex flex-row gap-2">

`app/routes/concerts.trending.tsx`

`/concerts/trending`

</div>

```tsx
export const config = { runtime: 'edge' };
export function loader() { ... }

export default function NestedComponent() {
    const { trending } = useLoaderData();
    return <ul>{trending.map(trend => <li>{trend.name}</li>)}</ul>;
}
```

</div>

---
transition: none
---

## Send as little data over the network as possible

* Mobile network

<div class="flex justify-center">
    <img src="/us_smartphone_dependence_pew.webp" class="h-[26rem] border border-gray-200 dark:border-none">
</div>

<!--
As smartphone ownership and use grow, the frontends we deliver remain mediated by the properties of those devices. The inequality between the high-end and low-end is only growing, even in wealthy countries. What we choose to do in response defines what it means to practice UX engineering ethically.
-->

---
transition: none
---

## Send as little data over the network as possible

* Nested routing

<div class="flex justify-center">
    <img src="/nested-routing.png" class="h-[26rem]">
</div>

<!-- 
Instead of fetching and rendering all of the data for the entire page every time the URL changes, if you nest routes, you only have to send the data for the portion of the page that has changed.

Remix does nested routing by default, take advantage of it!
-->

---
transition: none
---

## Send as little data over the network as possible

<div class="border-l-4 border-yellow-400 bg-yellow-50 dark:bg-yellow-950 dark:border-yellow-700 p-2">
    <div class="flex items-center">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="h-5 w-5 text-yellow-400 dark:text-yellow-500">
  <path fill-rule="evenodd" d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25ZM12.75 6a.75.75 0 0 0-1.5 0v6c0 .414.336.75.75.75h4.5a.75.75 0 0 0 0-1.5h-3.75V6Z" clip-rule="evenodd" />
</svg>
        <div class="ml-2">
            <span class="text-sm text-yellow-700 dark:text-yellow-500">
                Coming Soon:
            </span>
            <span class="text-sm text-yellow-800 dark:text-yellow-600">
                Partial hydration with React Server Components
            </span>
        </div>
    </div>
</div>

```tsx {all|4,5|13,16|12,15,17,18}
function loader() {
    const { title, content } = await loadArticle();
    return {
        articleHeader: <Header title={title} />,
        articleContent: <AsyncRenderMarkdownToJSX makdown={content} />,
    };
}

export default function Component() {
    const { articleHeader, articleContent } = useLoaderData();
    return (
        <main>
            {articleHeader}

            <React.Suspense fallback={<LoadingArticleFallback />}>
                {articleContent}
            </React.Suspense>
        </main>
    );
}
```

<!--
Partial hydration (though it's more like lakes than islands) is coming to Remix in the future with React Server Components: https://github.com/remix-run/remix/discussions/8048 
-->

---

## Ship HTML to your users

* Websites should work as soon as you can see them: progressive enhancement!

<v-clicks>
<div>

* Apps that are rendered entirely via JavaScript require waiting for the script(s) to download and execute before anything is displayed to users

<div class="my-4 ml-5 w-1/2">

```
HTML        |---|
JavaScript      |---------|
Data                      |---------------|
                            page rendered 👆
```

</div>
</div>
<div>

* HTML will download and display quicker than JavaScript and allows more parallel execution

<div class="mt-4 ml-5 w-1/2">

```
               👇 first byte
HTML        |---|-----------|
JavaScript      |---------|
Data        |---------------|
              page rendered 👆
```

</div>
</div>
</v-clicks>

<!-- </div> -->

---

## HTML in Remix

* By default, Remix server-renders all routes
* You can opt out of this on a per-route basis using the new `clientLoader` and `clientAction` exports along with `HydrateFallback`... but you're still shipping HTML!

```tsx {all|7}
export async function clientLoader() {
    const data = await loadSavedGameOrPrepareNewGame();
    return data;
}

export function HydrateFallback() {
    return <p>Loading Game...</p>;
}

export default function Component() {
    const data = useLoaderData<typeof clientLoader>();
    return <Game data={data} />;
}
```

<!-- If you have a site or page restricted to client-only rendering, you can pre-render your app's shell using HydrateFallback: https://remix.run/docs/en/main/route/hydrate-fallback -->

---
transition: none
---

## `HydrateFallback` User Experience

<v-clicks>

* Displaying your app's shell UI the moment it loads and quickly replacing it with your first screen allows you to give your users the impression that your experience is fast and responsive
* The ideal `HydrateFallback` is effectively invisible to people, because it simply provides a context for your initial content
* Your fallback should not an onboarding experience or a splash screen; its primary function should be to enhance the perception of your experience as quick to launch and immediately ready to use
* **The HTML you export from your `HydrateFallback` should be nearly identical to the HTML on the initial screen of your app.** If you include elements that look different when the data finishes loading, people may experience an unpleasant flash between the fallback and the initial screen of the app.

</v-clicks>

---
transition: none
---

## `HydrateFallback` User Experience

<div class="flex justify-center">
    <img src="/app-shell-initial.png" class="h-[30rem]">
</div>

---

## `HydrateFallback` User Experience

<div class="flex justify-center">
    <img src="/app-shell-loaded.png" class="h-[30rem]">
</div>

---

## Make pages work before JavaScript loads

<v-clicks class="text-xl mt-4">

* Why? [Everyone has JavaScript, right?](https://www.kryogenix.org/code/browser/everyonehasjs.html)
* The data layer of your site should function with or without JavaScript on the page
* It will make your website *feel* faster
* This is part of progressive enhancement 

</v-clicks>

---

## Progressive Enhancement in Remix

```tsx
/**
 * The public API for rendering a history-aware `<a>`.
 */
export const Link = ({ to: href, onClick, ref, target, ...props }) => {
    // ...
    return (
        <a
            href={href}
            target={target}
            onClick={isExternal || reloadDocument ? onClick : handleClick}
            ref={ref}
            {...props}
        />
    );
}
```

<!-- Progressive enhancement in Remix is built into the philosophy. Using `<Form>` and `<Link>` progressively enhance the existing HTML `<form>` and `<a>` elements -->

---
transition: none
---

## Progressive Enhancement in Remix

```tsx
/**
 * A `@remix-run/router`-aware `<form>`.
 */
export const Form = ({ method, action, onSubmit, ref, ...props }) => {
    // ...
    return (
        <form
            method={method}
            action={action}
            onSubmit={reloadDocument ? onSubmit : handleSubmit}
            ref={ref}
            {...props}
        />
    );
}
```

---

## Embrace web standards and browser defaults

<div class="text-xl">

* `<form>`
* `<link rel="prefetch">`
* URLs for assets
* `Cache-Control`

</div>

<!-- Remix does this for you by default! No need to change anything about your Remix app. If you're employing Remix best practices then you're already using web standards. -->

---

## `<link rel="prefetch">` in Remix

```tsx {all|5}
export default function Component() {
    return (
        <>
            <h1>See New Concerts in Your Area</h1>
            <Link to="concerts/new" rel="prefetch" />
        </>
    );
}
```

```tsx {all|4}
export default function Component() {
    return (
        <>
            <PrefetchPageLinks page="/concerts/trending" />
            <h1>All Concerts</h1>
            <ul>...</ul>
        </>
    );
}
```

<!-- 
This component enables prefetching of all assets for a page to enable an instant navigation to that page. It does this by rendering `<link rel="prefetch">` and `<link rel="modulepreload"/>` tags for all the assets (data, modules, css) of a given page. 
-->

---

## `Cache-Control` in Remix

<div class="large-code">

```ts
import type { HeadersFunction } from "@remix-run/node"; // or cloudflare/deno

export const headers: HeadersFunction = () => ({
    "Cache-Control": "max-age=604800, stale-while-revalidate=86400",
});
```

</div>

<!--
HTTP stale-while-revalidate caching directive 

The result: a static document at the edge

Instead of fetching all of the data and rendering the pages to static documents at build/deploy time, the cache is primed when you're getting traffic

Documents are served from the cache and revalidated in the background for the next visitor

Like SSG, no visitor pays the download + render cost when you're getting traffic

SWR is a great alternative to SSG and many CDNs support it
-->

---
layout: new-section
---

<!--
This page intentionally left blank

That's what I've got for you today. Does anyone have any questions?
-->
