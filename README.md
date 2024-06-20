# Image Optimization + Correct Src Paths for Nextjs Static Builds on GitHub Pages

That's a long winded title, I know. Essentially, the story is this.
I created a Nestjs Static (Generated / Exported / Whatever) Application for a client and wanted to host it using GitHub Pages.
The site was super simple, so didn't need SSR or Edge Functions or any of the extra stuff you get with a Nextjs Application.

There are some solutions out there to 'solve' this but most are a half baked solve crammed into an NPM Package.
Rather than do the same thing but maybe slightly better, I just created some stuff and built a minimal reproduction of what I did.
The purpose of this is to give you, the reader, something that you can copy paste into your project, and easily modify if necessary.
The reality is that this requirement is so simple, you really don't need a another dependency with more dependencies and more dependencies still... You really just need get on with it and write some code (or just copy mine).

## Understanding the problems

### Nextjs

Lets start at the application. Nextjs uses image optimization by default, using built in (bundled) libs. However, those don't work with Static Exports. Why? Probably just not a priority to make that work. <br/>

### GitHub Pages

The second part of this issue is not necessarily GitHub's issue, in fairness it's Nextjs not respecting `basePath` in config. <br/>
What does that mean though? Essentially, GitHub Pages hosts on both a subdomain, and a route. <br/>
When you publish to GitHub Pages, the URL created looks something like this.<br/>
https://your-username-or-org-name.github.io/repo-name
<br/>
To make your site load on GitHub Pages, you set this in your next.config.js|mjs file, OR you just let the workflow from the GitHub Action found here [actions/configure-pages@v5](https://github.com/actions/configure-pages) do it for you.

``` mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  basePath: "/repo-name",
};

export default nextConfig;
```

The issue this causes is such: when you source images locally (from your /public/ folder for example), you would reference it like this

```ts
<Image src="/myImage.jpg" ... />
```

BUT, when your site is published, you notice that your images aren't loaded. Upon inspecting the now HTML \<img /> tags, and adding '/repo-name' to the start, they magically work. Surely, setting a basePath in your next.config.js would be respected by the `<Image />` primative right?
Oh you sweet summer child, Triangle Company doesn't care about your Statically Exported Nextjs Site you want to host on GitHub.
<br/>

## The Simple 'Fix'

<b>Just turn off Image Optimization, right?</b><br/>
Sure, if you want to do that. Go ahead. Here's the Config to do so.

``` mjs

/** @type {import('next').NextConfig} */
const nextConfig = {
  basePath: "/<repo-name>",
  images: {
    unoptimized: true,
  },
};

export default nextConfig;
```
<br/>


## Actually Fixing it

There are three main components to resolving this issue. 

- Image Optimization Script
- Custom Loader
- Higher Order Component for `<Image />`

