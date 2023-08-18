---
title: "Image Hosting Service and How to implement MVC-ish architecture in Cloudflare Workers"
date: 2023-08-18T14:10:32+07:00
description: "A writeup of my Image Hosting Service project"
categories:
    - Project
tags:
    - Serverless
    - Cloudflare
---
in this post, I will try to explain my own interpretation of MVC architecture and how I implement it in my Cloudflare Workers project. I'll be using my Image Hosting Service project as an example. This project will be fully hosted on **Cloudflare Workers** and **Cloudflare R2**. No static hosting is used.

## But Why?

NodeJS is my go-to stack when developing websites; I typically use React for the front-end and Express for the back-end REST API. But when I had to use the MVC framework Django to create a web application for my internship, I fell in love with it right away.

However, working with Python can feel a bit "magical" in a bad way because of its dynamic nature. Built-in arguments frequently lack typehints, and Django ORM methods and arguments are difficult to comprehend without spending a lot of time reading through the documentation and source code. Additionally, Django did not perform well with method completion. All these issues are resolved by TypeScript.

As a result, I tried right away to implement MVC using NodeJS and the Express library using EJS. That stack performed flawlessly throughout. It works up until the point where you try to implement it using serverless.


### The Problem

Workers cannot be effectively used with EJS because of its close relationship to NodeJS functions. Mainly, the ability to do an evaluation function is not supported. So, after doing extensive research, I discovered two potential solutions for EJS replacement in workers. That is:

1. Use the HTMLRewriter class and webpack (which is supported by default) to load HTML and add dynamic content, or 
2. Create the HTML manually using template literals and return it with the necessary headers.

The issue with the first method is that I have a really difficult time configuring Webpack in Workers because serverless cannot access filesystem functionality. Aside from that, compared to another MVC framework that I am already familiar with, using HTMLRewrite to inject a dynamic value seems a little counterintuitive even if I can get it to work. So I chose the latter.


# Solution
## Disclaimer
I won't go deep to how my image hosting services work, because it really is just a simple listen to POST request with `multipart/form-data;` and upload it to R2 bucket. You can find the source code in [GitHub](https://github.com/reverseon/img.thr.fi) if you want to look closer.
## The Idea

The idea is simple, it is expanded on this Workers [example](https://developers.cloudflare.com/workers/examples/return-html/). So, you can return an HTML page with this code:

```ts
return new Response(html_string, {
    headers: {
        "Content-Type": "text/html; charset=UTF-8"
    }
})
```

The next logical question is then, how to pass a dynamic variable and inject it to an HTML string? My solution is to use javascript template literals for this. The one with `${}` syntax.

For example, if i want to display dynamic heading, I first create a function that constructs a desired HTML strings from given arguments. For example.

```ts
const index__html = (ctx: any): string => {
    return `
        <h1>${ctx.title}</h1>
    `
};
```

and then, you can return that page by using this syntax in your workers.

```ts
return new Response(index__html(
    {
        title: "Dynamic Heading"
    },
    {
        headers: {
            "Content-Type": "text/html; charset=UTF-8"
        }
    }
));
```

Looks a lot more like Django or Laravel, doesn't it? With template literals, you can expand almost any basic features in MVC templating engine. For loop? just make a function that construct the desired string using for-loop and call it inside template literals. Include? make a function that returns included HTML strings and call that function inside string literals. Just be careful with your function namings to prevent confusion.

## Elephant in the Room

Ok, this works with HTML or any string-based content. Then, how about assets serving that requires bytestream, for example, images? Workers doesn't have access to filesystem, so static serving router that utilizes something like:

```ts
router.get('/:filename', async (req, res, ctx) => {
    return new Response(fs.arrayBuffer(req.params.filename))
})
```

cannot work. My own band-aid solution is to utilize base64 representation since it is already a string. This may be inefficient, but I can't think of any better solution at the time of this writing.

For example, if you want to serve `image.jpg` in route `/static/img/`, you can first create a function that returns the base64 representation of the image. i.e.

```ts
const image__jpg = (): string => {
    return `
    // base64 here
    `
}
```

and then, in your router, you can hard-code the endpoint to match the name of the file (or not, its entirely up to you) and then return the byte-converted value and set the appropriate headers like this.

```ts
router.get('/img/image.jpg', async (
    request: Request, env: Env, ctx: ExecutionContext
) => {
    const base64 = image__jpg();
    const bytes = Uint8Array.from(atob(base64), c => c.charCodeAt(0));
    return new Response(
        bytes, {
            headers: {
                'Content-Type': 'image/jpeg',
                'Content-Length': bytes.length.toString(),
                'Content-Disposition': 'inline; filename="image.jpg"',
                'Cache-Control': 'public, max-age=31536000',
            },
        }
    );
});
```

# Conclusion
To sum up, this might be a really hacky and inefficient way to implement MVC-ish architecture in a serverless environment. Of course there is a tradeoff by using serverless. For example, you need to rely on Cloudflare's rate-limiting to rate limit the API, and it's hard to create CSRF token without delving deep into storage medium like KV and it's synchronization problem. But overall, it's a project well(-enough) done, and I'm proud of it.