# Going Further with Cloudflare

You've built and deployed a website on GitHub Pages. That site is **static** — every visitor sees the same pre-built HTML files. This is fast and simple, but it means your site can't respond to individual users, store data, or run any logic at the moment someone visits.

This guide introduces the other side of web development. You'll deploy your site to a global network, add a visitor counter that remembers its state between visits, and connect your own domain name. Along the way, you'll work with serverless computing, key-value storage, APIs, and DNS — the building blocks that make web applications dynamic.


## What is Cloudflare?

Cloudflare is one of the world's largest web infrastructure companies. They operate a global network of data centres in over 300 cities, and a significant proportion of all web traffic passes through their infrastructure. Among many other things, they offer:

- **Cloudflare Pages** — website hosting with a global **CDN** (Content Delivery Network). Where GitHub Pages serves your site from a single location, a CDN caches copies of your site on servers around the world, so visitors load it from whichever server is geographically nearest to them. This is how large-scale websites achieve fast load times globally.
- **Workers** — **serverless functions** that run your code on Cloudflare's network. "Serverless" doesn't mean there are no servers — it means *you* don't manage them. You write a function, deploy it, and Cloudflare handles everything else: provisioning, scaling, uptime, and distribution. You're charged per request rather than per server, which means you pay nothing if nobody uses it.
- **KV Storage** — a **key-value database** distributed across Cloudflare's network. Unlike a traditional relational database (which stores data in structured tables with rows and columns), a key-value store is much simpler: you store a value against a key, and retrieve it later by that key. It's ideal for simple, fast lookups — things like configuration settings, session data, or the visitor counter you'll build below.
- **Domain Registration** — buy and manage your own domain names, with DNS management built in.

All of these have generous free tiers. You won't need to pay anything to complete these exercises.


## Getting Started

1. Sign up at [dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)
2. Verify your email address


---


## Exercise 1: Deploy Your Site to Cloudflare Pages

In the main session, you enabled GitHub Pages and your site was built and deployed automatically. Cloudflare Pages works on the same principle, but it makes the process more explicit — and more powerful.

When you connect Cloudflare Pages to your GitHub repository, you're setting up a **CI/CD pipeline** (Continuous Integration / Continuous Deployment). Every time you push a change to your repository, Cloudflare automatically pulls the latest code, runs the build command, and deploys the result. This is the same pattern used by professional development teams — the only difference is that at scale, teams add stages for automated testing, code review checks, and staging environments before changes reach production.

1. In the Cloudflare dashboard, go to **Compute & AI** then **Workers & Pages**
2. Click **Create application** — you'll see several options for creating Workers, but below these look for the message **"Looking to deploy Pages? Get started"** and click **Get started**
3. Choose **Import an existing Git repository**
4. Authorise Cloudflare to access your GitHub account — this uses **OAuth**, an industry-standard protocol that lets you grant Cloudflare limited access to your GitHub repositories without sharing your password
5. Select your website repository
6. Configure the build settings:
   - **Framework preset**: Jekyll
   - **Build command**: `jekyll build` — this is the command Cloudflare will run on its servers to convert your Markdown into HTML. It's the same process GitHub Pages runs, but here you can see and control it explicitly.
   - **Build output directory**: `_site` — when Jekyll builds your site, it writes the finished HTML, CSS, and assets into a folder called `_site`. This tells Cloudflare where to find the files it should actually serve to visitors.
7. Click **Save and Deploy**

Your site will be available at `https://YOUR-PROJECT.pages.dev` within a few minutes. You now have your website running on two independent platforms — GitHub Pages and Cloudflare Pages — both deploying automatically from the same repository.

**Why bother when GitHub Pages already works?** Several reasons. Cloudflare's CDN means your site loads faster for visitors around the world. Cloudflare Pages also supports **preview deployments** — when you create a branch in your repository (a parallel version of your code), Cloudflare automatically builds and deploys it to a temporary URL so you can review changes before merging them into your main site. This is how professional teams test changes safely. And critically, having your site on Cloudflare's platform gives you access to Workers and KV Storage for the dynamic features in the next exercise.


---


## Exercise 2: Add a Visitor Counter

So far, everything you've built is **static** — your site serves the same pre-built files to every visitor. To do anything dynamic (respond to user actions, store data, personalise content), you need code running on a server somewhere. Traditionally, that meant renting a server, installing an operating system, configuring a web server, managing security patches, and handling traffic spikes yourself. **Serverless computing** eliminates all of that.

With serverless, you write a function and deploy it. The platform (in this case, Cloudflare) handles everything else — it runs your code when a request arrives, scales it automatically if thousands of requests arrive at once, and charges you only for what you actually use. You don't think about servers at all.

In this exercise, you'll build a simple **API** (Application Programming Interface) — a URL that, when visited, executes your code and returns data. Your website's front end will call this API to fetch and display a visitor count. This is the **client-server model** in action: your HTML/CSS/JavaScript runs in the visitor's browser (the client), and your Worker runs on Cloudflare's servers (the server).


### Step 1: Create a KV Namespace

Before your Worker can count anything, it needs somewhere to store the count. KV (Key-Value) storage is the simplest type of database: you store a value against a key, and retrieve it later by that key. Think of it like a dictionary or a lookup table — `"count" → "42"`.

A **namespace** is an isolated container for your key-value pairs. This keeps data from different projects separate, just as separate databases would in a larger system.

1. In the Cloudflare dashboard, go to **Compute & AI** then **Workers & Pages** then **KV**
2. Click **Create a namespace**
3. Name it `VISITS`
4. Click **Add**


### Step 2: Create a Worker

1. Go to **Compute & AI** then **Workers & Pages** and click **Create application**
2. Name it `visitor-counter`
3. Click **Deploy** (we'll edit the code next)
4. Click **Edit code** and replace everything with:

```javascript
export default {
  async fetch(request, env) {
    // Handle CORS preflight requests so browsers
    // allow your site to call this worker
    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "GET",
        },
      });
    }

    // Get the current count from KV storage (or start at 0)
    const current = await env.VISITS.get("count");
    const count = (parseInt(current) || 0) + 1;

    // Save the updated count back to KV storage
    await env.VISITS.put("count", count.toString());

    // Send the count back as JSON
    return new Response(JSON.stringify({ visits: count }), {
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*",
      },
    });
  },
};
```

5. Click **Save and Deploy**

Take a moment to read through the code — there's a lot happening in a small number of lines, and each part introduces an important concept:

**The request/response model.** The entire web is built on this pattern: a client sends a **request**, a server processes it and sends back a **response**. Your Worker's `fetch` function receives a request object (containing the URL, HTTP method, headers, and any body) and must return a response object. Every website you've ever visited works this way.

**CORS (Cross-Origin Resource Sharing).** Browsers enforce a security policy called the **same-origin policy** — by default, JavaScript running on one domain (like `your-username.github.io`) cannot make requests to a different domain (like `visitor-counter.your-subdomain.workers.dev`). This prevents malicious websites from making requests to your bank's API using your logged-in session. The `Access-Control-Allow-Origin` headers tell the browser "it's fine, I explicitly allow requests from other origins." The `OPTIONS` check handles the browser's "preflight" request — a safety check it sends before the actual request to confirm the server permits cross-origin access.

**Async/await.** The `async` and `await` keywords handle **asynchronous operations** — things that take time, like reading from a database or making a network request. Rather than blocking everything while waiting for KV storage to respond, `await` pauses just this function and lets the server handle other requests in the meantime. This is how modern JavaScript handles concurrency, and it's the reason Node.js and serverless platforms can handle thousands of simultaneous connections efficiently.

**JSON (JavaScript Object Notation).** The response `{ visits: count }` is formatted as JSON — a lightweight data format that has become the universal standard for APIs. It's human-readable, language-agnostic (virtually every programming language can parse it), and simple enough to be understood at a glance. When you see API documentation, the examples are almost always in JSON.

**Environment bindings.** Notice that the KV namespace is accessed through `env.VISITS`, not by importing a library or hardcoding a connection string. This is the **bindings** pattern — external resources are injected into the function's environment at runtime. This is a deliberate security practice: your code never contains credentials or connection details, which means it's safe to store in a public repository.


### Step 3: Connect KV Storage to Your Worker

The Worker needs to be explicitly connected to your KV namespace through a **binding**. This is the principle of **least privilege** — your Worker only has access to the specific resources you grant it, nothing more. If you created other KV namespaces for other projects, this Worker couldn't access them unless you explicitly bound them.

1. Go to your Worker's **Settings** then **Bindings**
2. Click **Add** then **KV Namespace**
3. Set the **Variable name** to `VISITS` — this is the name your code uses to reference the binding (`env.VISITS`), and it must match exactly. Think of it as a local alias for the external resource.
4. Select the `VISITS` namespace you created earlier
5. Click **Save**


### Step 4: Test Your Worker

Visit your Worker's URL directly in a browser:

```
https://visitor-counter.YOUR-SUBDOMAIN.workers.dev
```

You should see something like `{"visits": 1}`. Refresh the page and the number should increase. What you're seeing is the raw JSON response from your API — this is exactly what your website's JavaScript will receive when it calls this URL. You've just built and tested a working API endpoint.


### Step 5: Add the Counter to Your Website

Now you'll connect the front end (your website) to the back end (your Worker). Edit your `index.md` and add this HTML snippet wherever you'd like the count to appear:

```html
<p id="visitor-count"></p>

<script>
  fetch("https://visitor-counter.YOUR-SUBDOMAIN.workers.dev")
    .then(function(response) { return response.json(); })
    .then(function(data) {
      document.getElementById("visitor-count").textContent =
        "This site has been visited " + data.visits + " times.";
    });
</script>
```

Replace `YOUR-SUBDOMAIN` with your Cloudflare Workers subdomain (you'll find this in your Workers dashboard).

One thing worth noting: Markdown files can contain raw HTML, and Jekyll will pass it through to the final page unchanged. This is by design — Markdown handles the common cases elegantly, but when you need something Markdown can't express (like a `<script>` tag), you can drop down to HTML without leaving the file.

The `fetch()` function is the browser's built-in way of making HTTP requests from JavaScript. It's the modern replacement for an older technique called **AJAX** (Asynchronous JavaScript and XML) — you may still see that term in documentation and job descriptions. The `.then()` chain handles the asynchronous response: first parsing the JSON, then updating the page content. This pattern of fetching data from an API and updating the page without a full reload is fundamental to how modern web applications work — it's the basis of every single-page application framework like React, Vue, and Angular.

**What you've just built:**

You now have a complete, working web application with a genuine **client-server architecture**:

- A **front end** (your GitHub Pages site) that runs in the visitor's browser
- A **back-end API** (your Cloudflare Worker) that runs on a server
- A **database** (KV Storage) that persists data between requests
- **Cross-origin communication** between two different platforms, secured by CORS

These are the same architectural building blocks used by professional web applications at companies of all sizes. The visitor counter is simple, but the pattern — front end calls API, API reads/writes database, API returns response, front end updates — is the foundation of virtually every dynamic web application.


---


## Exercise 3: Connect Your Own Domain Name

A custom domain name like `www.yourname.co.uk` makes your site feel truly yours. This is the one step that costs money — but domain names are relatively inexpensive, and understanding how they work is genuinely valuable knowledge.

When someone types a URL into their browser, the first thing that happens — before any page loads — is a **DNS lookup** (Domain Name System). DNS is essentially the phone book of the internet: it translates human-readable domain names (like `www.yourname.co.uk`) into the IP addresses (like `104.21.32.1`) that computers use to find each other on the network. Without DNS, you'd need to remember numerical addresses for every website you visit.

DNS records come in several types, but the most relevant for your purposes are:

- **A records** — map a domain name directly to an IP address
- **CNAME records** (Canonical Name) — map a domain name to *another* domain name, which then resolves to an IP address. This is what you'll use, because you're pointing your domain to GitHub's or Cloudflare's infrastructure rather than to a specific server.
- **TXT records** — store arbitrary text, commonly used to verify domain ownership


### Buying a Domain

A **domain registrar** is a company authorised to sell domain names. Cloudflare operates as a registrar and sells domains at cost (they don't mark up the wholesale price), which makes them one of the most affordable options.

1. In the Cloudflare dashboard, go to **Domain Registration** then **Register Domains**
2. Search for a domain name you'd like
3. Prices vary — `.co.uk` domains are typically around £5–8 per year, `.com` around £8–10 per year
4. Complete the purchase

You can also buy domains from other registrars (like Namecheap) and point them to Cloudflare. It's worth understanding that the registrar (where you buy the domain), the DNS provider (where the domain's records are managed), and the hosting provider (where your site's files live) can all be different companies. In this setup, Cloudflare would handle the first two, and either Cloudflare Pages or GitHub Pages handles the third.


### Connecting to Cloudflare Pages

If you deployed your site to Cloudflare Pages in Exercise 1:

1. Go to your Cloudflare Pages project
2. Click **Custom domains** then **Set up a custom domain**
3. Enter your domain name
4. Cloudflare handles the DNS configuration automatically

Your site is now live at your own domain. Cloudflare also automatically provisions a **TLS/SSL certificate** — this is what puts the padlock icon in the browser's address bar and enables `https://` (the secure version of HTTP). The certificate proves to visitors' browsers that they're genuinely connected to your site and not an impostor, and it encrypts all data in transit. A few years ago, setting up HTTPS required purchasing and manually installing certificates — Cloudflare handles this entirely for you.


### Connecting to GitHub Pages

If you'd prefer to keep using GitHub Pages as your host:

1. In your GitHub repository, create a file called `CNAME` containing just your domain name:
   ```
   www.yourname.co.uk
   ```
2. In the Cloudflare dashboard, go to **DNS** and add a **CNAME record** pointing to `YOUR-USERNAME.github.io` — this tells DNS that when someone looks up your domain, they should be directed to GitHub's servers
3. In GitHub, go to your repository's **Settings** then **Pages** and enter your custom domain — this tells GitHub to respond to requests for your domain name, not just the `github.io` address

Either approach works. Using Cloudflare Pages gives you the additional benefit of serverless functions (like the visitor counter) running on the same platform, and the CDN means your site is served from the nearest data centre to each visitor.


---


## What You've Covered

Through these extension exercises, you've moved from static web publishing into genuine full-stack web development. Here's a summary of what you've worked with and where each concept leads:

| What you did | The concept | Where it leads |
|---|---|---|
| Deployed to Cloudflare Pages | CI/CD pipelines | Automated testing, staging environments, release management |
| Configured build settings | Build processes | Bundlers (Webpack, Vite), transpilers, optimisation pipelines |
| Created a Worker | Serverless computing | Cloud functions (AWS Lambda, Azure Functions), microservices |
| Used KV Storage | Data persistence | Relational databases (SQL), document stores (MongoDB), caching (Redis) |
| Built an API endpoint | Client-server architecture | REST APIs, GraphQL, WebSockets, API design |
| Handled CORS | Web security | Authentication, authorisation, OWASP, security headers |
| Used fetch() in the browser | Asynchronous JavaScript | React, Vue, Angular, single-page applications |
| Connected a domain | DNS and networking | Load balancing, CDNs, TLS/SSL, network architecture |

Everything you've built here uses the same tools, platforms, and architectural patterns that professional developers work with daily. The difference between this and a production application is scale and complexity, not kind.


---


## Useful Links

- [Cloudflare Pages Documentation](https://developers.cloudflare.com/pages/)
- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)
- [Cloudflare KV Documentation](https://developers.cloudflare.com/kv/)
- [Cloudflare Domain Registration](https://developers.cloudflare.com/registrar/)
