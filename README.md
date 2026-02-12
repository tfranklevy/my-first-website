# Your First Website

A hands-on guide to building and deploying your own website, created for National Apprenticeship Week 2026.

By the end of this session, you'll have a live website with your own content, accessible to anyone in the world via a URL. Not a simulation or a local file on your computer — a real site, hosted on the internet, with a public address.

Along the way, you'll encounter tools and concepts that professional developers use every day: version control, static site generation, configuration files, and the deployment pipeline that takes your content from source files to a live website.


## What You'll Need

- A GitHub account (free) — sign up at [github.com](https://github.com)
- A web browser (this works on your phone)


## What You'll Learn

- How websites work behind the scenes — what happens between writing content and someone viewing it in a browser
- Writing content with Markdown — a lightweight markup language used across the industry
- Version control with Git and GitHub — how developers track changes and collaborate
- The deployment pipeline — how code goes from a repository to a live website
- Separation of concerns — why keeping content, structure, and style independent matters


---


## Step 1: Get Your Own Copy of This Site

Before we start building, it's worth understanding where your code will live. GitHub is built on **Git**, a version control system that tracks every change you make to your files. Think of it as an unlimited undo history — but one that also records *who* changed *what* and *when*, which becomes essential when working in teams.

A **repository** (or "repo") is a project folder managed by Git. It contains your files, your folder structure, and the complete history of every change ever made. This repository is a **template** — a pre-built starting point that you can copy into your own account and make entirely your own.

1. Make sure you're signed in to GitHub
2. Tap the green **"Use this template"** button at the top of this page
3. Choose **"Create a new repository"**
4. Give your repository a name (e.g., `my-website`)
5. Make sure **Public** is selected — this is required for GitHub Pages to work on the free tier, because GitHub serves your site from the repository's contents
6. Tap **"Create repository"**

You now have your own copy of this project. It's a completely independent repository — your changes won't affect the original template, and nobody else's changes will affect yours. This is how professional teams work: everyone has their own copy of the codebase, and changes are merged together deliberately through a process called **pull requests**.


---


## Step 2: Turn On GitHub Pages

GitHub Pages is a free hosting service that turns your repository into a live website. When you enable it, GitHub watches your repository for changes. Each time you make one, it runs a **build process**: your Markdown files are passed through a tool called **Jekyll** (a static site generator), which converts them into the HTML, CSS, and JavaScript that browsers understand. The output is then deployed to GitHub's servers and made publicly available.

This sequence — source files → build → deploy — is called a **deployment pipeline**, and it's the same pattern used by professional development teams, just with more stages and safeguards at scale.

1. In your new repository, tap **Settings** (the gear icon)
2. In the left sidebar, tap **Pages**
3. Under **Source**, select **Deploy from a branch**
4. Under **Branch**, select **main** and **/ (root)** — this tells GitHub which branch of your repository to build from, and that your source files are in the root directory rather than a subfolder
5. Tap **Save**

Give it a minute or two (GitHub needs to run that build process), then visit:

```
https://YOUR-USERNAME.github.io/YOUR-REPO-NAME
```

That's your website. It's live. Anyone with that link can see it.

Notice the URL structure: `github.io` is GitHub's domain, your username identifies your account, and the repository name identifies the project. Later, in the extension exercises, you'll see how to replace this with your own custom domain name.


---


## Step 3: Meet Markdown

The content on your site is written in **Markdown** — a lightweight markup language created by John Gruber in 2004. It was designed to solve a real problem: HTML is powerful but painful to write by hand, especially for content that's mostly text. Markdown lets you express formatting intent — headings, bold, links, lists — using simple, readable syntax that gets converted to HTML automatically.

This matters because it separates the concern of *what you're saying* from *how it's structured*. You focus on content; the tooling handles the markup. It's the same principle behind every content management system (CMS), from WordPress to the systems behind the BBC and GOV.UK.

Here's a quick reference:

| What you type | What you get |
|---|---|
| `# Heading` | A large heading (`<h1>` in HTML) |
| `## Smaller heading` | A smaller heading (`<h2>` in HTML) |
| `**bold text**` | **bold text** (`<strong>` in HTML) |
| `*italic text*` | *italic text* (`<em>` in HTML) |
| `[Link text](https://example.com)` | A clickable link (`<a>` in HTML) |
| `- Item` | A bullet point (`<ul><li>` in HTML) |
| `1. Item` | A numbered list item (`<ol><li>` in HTML) |
| `![Alt text](image-url)` | An image (`<img>` in HTML) |

That's most of what you need. Markdown has become an industry standard — it's used across GitHub, Notion, Slack, Discord, technical documentation, and many other tools you may already use. Learning it once pays off everywhere.


---


## Step 4: Edit Your Homepage

1. Go to your repository's main page
2. Tap on **index.md** — the name `index` is a web convention; it's the default file a server looks for when someone visits a directory. That's why it becomes your homepage.
3. Tap the **pencil icon** to edit
4. Change the content — make it yours
5. Scroll down and tap **"Commit changes"**

When you commit, you're creating a snapshot of your changes in Git's history. Every commit has a message describing what changed and why — this might seem like overhead for a personal site, but on team projects with hundreds of files, a clear commit history is essential for understanding how and why code evolved. Good commit messages are a professional skill worth developing early.

After committing, GitHub detects the change, triggers the Jekyll build process, and deploys the updated site. This typically takes a minute or two — you're witnessing the deployment pipeline from Step 2 in action.


---


## Step 5: Under the Hood — The `<head>` Tag

Your Markdown gets transformed into HTML — the language that web browsers actually understand. This is what Jekyll does during the build process: it takes your `.md` files, applies the theme's layout templates, and outputs complete `.html` files.

You can see the result for yourself: on your live site, right-click (or long-press on mobile) and choose **"View Page Source"**. This shows you exactly what the browser received — the raw HTML that Jekyll generated from your Markdown.

Near the top, you'll find the `<head>` tag. This section is never visible on the page itself, but it's critical — it's where you communicate with the browser, search engines, and social media platforms about your site:

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>My First Website</title>
  <meta name="description" content="Built during National Apprenticeship Week 2026">
  <link rel="stylesheet" href="/assets/css/style.css">
</head>
```

Each element serves a specific purpose:

- **`<meta charset="utf-8">`** — specifies the character encoding. UTF-8 supports virtually every written language and is the universal standard for the web. Without it, characters outside basic ASCII (accented letters, emoji, non-Latin scripts) could display incorrectly.
- **`<meta name="viewport" ...>`** — controls how the page scales on different screen sizes. This single line is what enables **responsive design** — without it, mobile browsers would render your page as if it were being viewed on a desktop and then shrink it down, making text unreadably small.
- **`<title>`** — the text shown in the browser tab, in search engine results, and when someone bookmarks your page. This is pulled from the `title` field in your `_config.yml`. It's one of the most important elements for **SEO** (search engine optimisation).
- **`<meta name="description" ...>`** — the snippet that appears below your page title in search engine results. It's also used by social media platforms when someone shares a link to your site — this is why some shared links show a helpful summary and others don't.
- **`<link rel="stylesheet" ...>`** — loads an external CSS (Cascading Style Sheets) file that controls the visual presentation of your site. This is the theme in action — the same HTML content, styled differently depending on which CSS file is loaded.

Your content — the Markdown you write — appears inside the `<body>` tag, further down in the source. Every HTML page has this same fundamental structure: `<head>` for metadata and configuration, `<body>` for visible content.


---


## Step 6: Change Your Theme

This is where a fundamental principle of software engineering becomes visible: **separation of concerns**. The idea is that different aspects of a system should be handled independently, so you can change one without breaking or even touching the others.

In your site, three concerns are cleanly separated:

- **Content** — your Markdown files (what the site says)
- **Structure** — the HTML templates in the theme (how the content is arranged on the page)
- **Presentation** — the CSS stylesheets in the theme (how everything looks)

Watch what happens when you change only the presentation layer:

1. Go to your repository and open **_config.yml** — this is a **configuration file** written in YAML (a human-readable data format). It holds settings that control how Jekyll builds your site, without being part of the content itself.
2. Tap the pencil icon to edit
3. Change the `theme` line to something different:

```yaml
theme: jekyll-theme-slate
```

4. Commit the change and wait a couple of minutes

Your site now looks completely different — same words, same structure, entirely different visual presentation. You didn't touch your content files at all. This is separation of concerns in practice, and it's the same principle behind every modern web framework, content management system, and design system you'll encounter professionally.

### Available Themes

Try any of these — just replace the theme name in `_config.yml`:

- `jekyll-theme-cayman` — the one you started with
- `jekyll-theme-slate` — dark sidebar, clean layout
- `jekyll-theme-minimal` — stripped back, content-focused
- `jekyll-theme-architect` — blueprint-inspired design
- `jekyll-theme-hacker` — green-on-black terminal aesthetic
- `jekyll-theme-midnight` — dark theme with blue accents
- `jekyll-theme-dinky` — compact, notebook-style
- `jekyll-theme-leap-day` — colourful with a date-inspired header
- `jekyll-theme-merlot` — warm, traditional feel
- `jekyll-theme-modernist` — bold typography
- `jekyll-theme-tactile` — textured, tactile design
- `jekyll-theme-time-machine` — retro, futuristic style


---


## Step 7: Add a New Page

1. Go to your repository's main page
2. Tap **"Add file"** then **"Create new file"**
3. Name it something like `projects.md`
4. Add this at the top of the file, followed by your content:

```markdown
---
layout: default
---

# My Projects

Write about what you've been working on here.
```

5. Commit the file

The block between the `---` lines at the top is called **front matter**. It's written in YAML and it's how you pass metadata to Jekyll — instructions about how to process this particular file. `layout: default` tells Jekyll to wrap your Markdown content in the theme's default page template. Without front matter, Jekyll treats the file as plain static content and won't apply your theme to it.

Your new page will be live at:

```
https://YOUR-USERNAME.github.io/YOUR-REPO-NAME/projects
```

Notice how the URL maps directly to the filename: `projects.md` becomes `/projects`. This is called **convention over configuration** — rather than needing a routing table or settings file to define which URL leads where, Jekyll uses a sensible default based on your file structure. Most web frameworks follow this principle in some form.

To link to it from your homepage, edit `index.md` and add:

```markdown
[See my projects](projects.md)
```

This is a relative link — it points to another file within the same repository. Jekyll will resolve it to the correct URL on your live site. You're now building a multi-page website with navigation, all from Markdown files in a Git repository.


---


## What Next?

You've built and deployed a real website using the same tools and workflows that professional developers use daily. Here's a summary of what you've covered and where each skill leads:

| What you did | The concept | Where it leads |
|---|---|---|
| Created a GitHub repository | Version control | Collaboration, code review, open source |
| Enabled GitHub Pages | Deployment pipelines | CI/CD, automated testing, staging environments |
| Wrote Markdown | Content as structured data | CMS platforms, documentation systems, technical writing |
| Committed changes | Change tracking | Branching, merging, pull requests |
| Inspected the `<head>` tag | HTML metadata | SEO, accessibility, social media integration |
| Changed themes | Separation of concerns | CSS frameworks, design systems, front-end architecture |
| Added pages and links | Site structure | Routing, navigation patterns, information architecture |

**Explore on your own** — add more pages and link them together, try different themes, or add images by uploading them to your repository and referencing them in Markdown.

**Take it further with Cloudflare** — deploy your site to a global content delivery network, add dynamic features like a visitor counter powered by serverless functions, and connect your own custom domain name. The [Cloudflare Extension Guide](extensions/cloudflare.md) in this repository walks through each of these step by step, introducing backend concepts like serverless computing, key-value storage, APIs, and DNS.


---


## Useful Links

- [Markdown Guide — Basic Syntax](https://www.markdownguide.org/basic-syntax/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Supported Jekyll Themes for GitHub Pages](https://pages.github.com/themes/)
- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)
