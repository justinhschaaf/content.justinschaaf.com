---
title: "Website V5"
desc: "So the blog's back in town. Naturally, starting off with one of these updates. The rest of the website may be slightly different too."
created: 1711103235
cover: "https://content.justinschaaf.com/common/blog/2024/03/22/v5Hero.webp"
---

So the blog's back in town. Naturally, starting off with one of *these* updates. I managed to copy over all the blog posts from when I used Squarespace, but I've gone ahead and hidden most of them since they're written horribly (in spite of what my English grade at the time would make you think). I sadly wasn't able to find the blog posts from before Squarespace, but most of them were probably so horrible I'd disable the posts anyways.

Oh yeah, the rest of the website also received a massive facelift.

It doesn't look much different when you're just staring at the hero image on the home page, but there's more content when you scroll down now: a little introductory paragraph and an overview of some of the projects I'm responsible for bringing into this world. I've taken the opportunity to polish this section a bit, notably I've added some features to the backend enabling more creative splashes, and multiline splashes no longer cause the logo to shift upwards.

Each project gets its own page too, featuring a demo video and some text about it. These are all WebP images so they have better quality than a GIF would. I also found that when using a WebM video, it either didn't work or my adblock would prevent it from autoplaying, so the WebP stays even if the quality isn't perfect. Thankfully, I had some prerecorded clips for the [Llama Steeds](/projects/llama_steeds) and [Killer Snow](/projects/killer_snow) demos, so I only had to start from scratch for [Infinitifall](/projects/infinitifall) and [Stylesheet](/projects/stylesheet). Ironically, the Stylesheet demo has the largest file size at 99MB, probably because in spite of just being a slow scroll of some basic colored boxes on a white background, it's 2 minutes long.

The footer is thankfully no longer a list of links to various places. Social icons have moved here along with a theme switcher, which was surprisingly easy to implement persistence for.

## Development

I initially planned to call this version v4.5 since my main motivation was to modernize the site's backend and make it work on Firefox again; however, considering the sheer amount of new content added and that it's been rewritten nearly from the ground up, v5 is far more fitting in my opinion.

![A piece of white printer paper on a transluscent green clipboard. The paper has the general layout of the website's home page drawn out with a Sharpie marker and multiple notes. One points at the navigation menu and says "folder tabs, sticky once scrolled past. Replace recipes with Tools to start." Another points at the tiles for the projects and says "fit width. There should probably be 4 for 1080+". A third points to a Theme switcher in the footer (placed where the Source section currently is) and says "'Manually overriding this stores a cookie'". An arrow points from the project tiles to a mockup of the current project page layout.](https://content.justinschaaf.com/common/blog/2024/03/22/Sharpie.webp)

##### Many of my ideas start off as Sharpie marker on a sheet of paper. This project managed to escape the filing cabinet and actually got made.

Svelte is still used for the backend; as Sapper[^1] was killed just a few months after I first made my website in it, this time around everything is built with SvelteKit--Sapper's successor. I already quite liked working with Svelte + Sapper before, but using SvelteKit this time around made the development process even more of a breeze. Once I got my mind warmed up, SvelteKit made it really easy to define routes, share logic for multiple pages, and fetch dynamic content server-side for improved performance. The whole experience was smooth and surprisingly fun for web dev, and if you make websites, I *highly* recommend you try it. I'll save more details about it for another post though.

Most of the core functionality only took a couple weeks starting mid-February. Once the core features were in, polishing and getting all the new assets done was a slog for the weeks between then and now. Naturally, when everything was working and it was time to show off the new website, shit hit the fan.

## Deployment

For a cleaner deployment solution, I wanted to take advantage of GitHub Actions to build and deploy the website for me instead of having all the production files stored in the repo's `docs/` directory. I had never written a GitHub Workflow before, and the wall of nondescriptive text that is GitHub's documentation for it scared me; however, finding a few good examples online, it turned out to be a breeze and worked flawlessly when publishing the website.

What didn't work flawlessly was GitHub Pages itself. The first time I tried to connect to the website, the routing system was completely broken, meaning everywhere you went was a 404 page--a 404 page that didn't load because the footer's social icons weren't fetching properly and as such crashed the layout.

The latter issue was easy enough to fix. I got the 404 page to at least show up, but the router was still broken for reasons only God knows why. I've ran into issues with GitHub Pages in the past, but nothing this bad before. I really didn't feel like troubleshooting the router when I knew I was trying to deploy a dynamic content system to a static website host, so I knew I needed to try something different, ***fast***.

I didn't want to bother configuring an OS and web server from the ground up, and I figured the old CPanel backend that powered my website before Squarespace was outdated before I started using it. Not understanding much about it, but seeing it in SvelteKit's docs, I ended up settling on [Vercel](https://vercel.com/). 

Vercel specializes in distributed deployment of frontend web frameworks (perfect for my use case), and has a pretty generous free tier. Best of all, deployments are incredibly fast and all of them are saved, so *if you're constantly updating the production version of your website because something specific to the production environment isn't working (foreshadowing)*, you're not sitting around waiting for the new GitHub Pages version to go live and can easily roll back when it breaks. I didn't fully understand what the platform was capable of at the time or how you're meant to best utilize it, and I still don't.

Signing up for Vercel and partially rewriting the GitHub Workflow to deploy there was pretty easy, and I ended up with a fully working build of my website within an hour. My domain name was working with it too, up and running less than 2 minutes after changing the DNS records over.

Sorry, did I say a fully working build? I meant it was *almost* fully working. While to my surprise, the router, splashes, projects, blog posts, RSS feed, and most dynamic stuff was working seamlessly, there was one small problem. ...at the very end of the website. ...in the footer.

Take a look down there and scroll back up here. Go on, I'll wait.

> [!TIP]
> If you just scrolled down to the footer, this is where you left off.

Did you notice the **Source** section, where the repository, branch, and commit SHA are? For non-technical readers, this is a bit of information displaying the version number of the website and where you can go to view the source code. It's the *one* thing that didn't work when moving to Vercel.

Ironically, it worked without issue on the GitHub Pages deployment by setting some environment variables in the workflow script. These same environment variables should have been set for the Vercel build, but because Vercel makes you use their special snowflake build command instead of just being able to upload your own files, it couldn't see them. I spent the next 3 hours fighting over getting these environment variables set.

- **Using the `env` variables in the GitHub Workflow didn't work.** These were just ignored as Vercel takes control of the build environment and all its environment variables. You have to build a Vercel project using it instead of being able to use your build tool directly.
- **Vercel's [System Environment Variables](https://vercel.com/docs/projects/environment-variables/system-environment-variables) didn't work.** Vercel conveniently has environment variables for the information I wanted--it's how [Modrinth](https://modrinth.com/) has commit info in its footer as well (and where I got the idea from). Because the GitHub Workflow was building the website instead of Vercel, these were unset.
- **Using the [Deploy to Vercel Action](https://github.com/marketplace/actions/deploy-to-vercel-action) instead of running the commands directly didn't work.** This broke more things as it tried to install Vercel again in the Nix shell that already had Vercel. I'm pretty sure it made no difference on the system environment variables despite claiming to provide commit metadata to the deployment; I can't say for certain though since there's no way to manually view the system environment variables for an individual build--all my searches for how to do so gave irrelevant results.
- **Using `vercel env` to set the environment variables didn't work.** This command was designed by a genius who never could've imagined someone wanting to set the value of an environment variable via an argument to the command. Instead, your options are to read the value from a file or pipe it from `stdout`. The recommended workaround of piping the output from `echo` ([I really wish I was joking](https://vercel.com/docs/cli/env#extended-usage)) didn't work.

If you remember scrolling down to the footer when I asked you to, it *clearly* works now, so what fixed it? Introducing the [Vercel Set Environment Variables GitHub Action](https://github.com/marketplace/actions/vercel-set-environment-variables). You can't just give it the names and values of the environment variables you want to set. ***Oh no. That would make too much sense.*** For *every* variable you want to set, you have to declare 3 for it: the variable itself, what environments it applies to (production, preview, and development, of course), and whether to encrypt the value. So you end up with this:

```yml
- name: Set Vercel Environment Variables
  uses: dkershner6/vercel-set-env-action@v3
  with:
    token: ${{ secrets.VERCEL_TOKEN }}
    projectName: justinschaaf-com
    envVariableKeys: PUBLIC_GITHUB_REPOSITORY,PUBLIC_GITHUB_REF_NAME,PUBLIC_GITHUB_SHA
  env:
    PUBLIC_GITHUB_REPOSITORY: ${{ github.repository }}
    TARGET_PUBLIC_GITHUB_REPOSITORY: production,preview,deployment
    TYPE_PUBLIC_GITHUB_REPOSITORY: plain
    PUBLIC_GITHUB_REF_NAME: ${{ github.ref_name }}
    TARGET_PUBLIC_GITHUB_REF_NAME: production,preview,deployment
    TYPE_PUBLIC_GITHUB_REF_NAME: plain
    PUBLIC_GITHUB_SHA: ${{ github.sha }}
    TARGET_PUBLIC_GITHUB_SHA: production,preview,deployment
    TYPE_PUBLIC_GITHUB_SHA: plain
```

It's a dirty solution in my opinion and takes up an unnecessary amount of space in the workflow file. Good news is--unlike anything else--*it actually works*.

## Outcome

I honestly really like what I managed to make here. It's probably my best website developed from the ground up, and most of it took only two weeks. There's finally proper metadata for thumbnails on social media sites and SEO, it feels alive without the animations being laggy or weird to interact with, it has a proper theme switcher, the code is easy to read and maintain, there's [privacy-friendly analytics](https://counter.dev/), and with Vercel + SvelteKit, it's really fast.

![A screenshot of GTmetrix showing the latest performance report for justinschaaf.com. The overall GTmetrix Grade is an A, with 98% Performance score and 93% Structure score. The Web Vitals show 538ms for Largest Contentful Paint, 125ms for Total Blocking Time, and 0 for Cumulative Layout Shift. The speed visualization graph at the bottom shows 538ms for both the first and largest contentful paints, 838ms time to interactive, 899ms onload time, and 5.2s fully loaded time.](https://content.justinschaaf.com/common/blog/2024/03/22/GTmetrix.webp)

##### I don't have the exact numbers for the old website anymore since GTMetrix snaps them after a month, but I have a feeling this is just as fast--or faster.

There's still room for improvement; for instance, on the home page, each project icon's ASCII version is actual HTML. These numbers are still insane to me though, and most of the heavy lifting for it is simply Svelte. I'm definitely proud of the CLS[^2] score though, as I did my best to manually reduce it as much as possible since it's one of my biggest pet peeves nowadays.

It may just be wishful thinking, but I'm feeling really good about the website after the redesign and bringing the blog back. With the new backend and content, I want to try and actually keep it up-to-date, give it the love and care it deserves, maybe add more sections to show off more of my passions. That's all assuming I keep my motivation and don't get pulled away by other projects, commitments, and life in general. 

Here's to hoping I won't let this place rot again.

[^1]: Sapper was the original routing library for building websites using Svelte components, as Svelte itself is less of a comprehensive website toolkit and more a framework for building the individual components to make up a website.

[^2]: Cumulative Layout Shift is how much visitors see the website content move while the page is loading. It makes it harder to interact with the content on the page and do what you're actually trying to, and as such, it's generally a bad thing.
