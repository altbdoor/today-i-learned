Vercel deploy prebuilt with SPA routes
===

June 20, 2024

Part of the project I was involved in was using [Vercel](https://vercel.com/) for preview deployments.
We would build our Angular project, and then use the [Build Output API](https://vercel.com/docs/build-output-api/v3) to push into Vercel.
Everything was pretty good, but recently we started seeing a new issue popping out.

### Single Page Application (SPA) fun!

When we visit the preview deployment with a pathname, we will get the 404 page. Let's take
<https://nice-url-bro.fake-vercel.app/manage/potatoes> as an example.
The `/manage/potatoes` is a valid route which we have already configured in the Angular router,
but it still shows a 404!

It looks like an old case of SPA-based routes. Angular used to provide [a bunch of examples](https://v17.angular.io/guide/deployment#fallback-configuration-examples)
on how to configure the server to always redirect to `index.html` when no files are matched
("used", because it is missing in their new [angular.dev](https://angular.dev/) website).

There is a `vercel init angular`, but that would mean the build process would be done by Vercel.
Again, we already have our built files, and just looking to upload into Vercel. Surely it is easy... right?

### What do you mean there's two config files?

In the Build Output API, the most basic `config.json` required would be `{ "version": 3 }`. This is all
that is needed for a purely static website build. Here was a rough idea of how it worked in our CI before:

```yml
run: |
  echo '{ "version": 3 }' > ./.vercel/output/config.json
  npm i -g vercel
  vercel pull --environment=preview
  vercel deploy --prebuilt
```

So we need to fix this with `routes`... right?

```diff
-  echo '{ "version": 3 }' > ./.vercel/output/config.json
+  echo '{ "version": 3, "routes": [{ "src": "/(.*)", "dest": "/index.html", "status": 200 }] }' > ./.vercel/output/config.json
```

This would redirect any routes to `index.html`... but it is literally _any_ routes. We will get an `index.html`,
where its CSS and JS are also from `index.html`!

Looking at most of the discussions out there, they seem to make changes to the `vercel.json` configuration.
But we don't have one, because the project was not initialized as a Vercel project.

But okay, lets create a `vercel.json`, dump our config in... but how do we know `vercel` will pick up `vercel.json`?
Lucky for us, there seems to be an argument for `--local-config`, which should really help... or not?

```diff
-  vercel pull --environment=preview
-  vercel deploy --prebuilt
+  vercel pull --local-config=vercel.json --environment=preview
+  vercel deploy --local-config=vercel.json --prebuilt
```

Nope. Nothing changed, and we're still looking at the mess of an `index.html`.

### One true config in `config.json`

By now, I have probably exhausted every Google search term I could think of:

- vercel deploy prebuilt spa
- vercel prebuilt spa
- vercel spa route
- vercel angular spa

And many other permutations we could think of. One particular blog post was interesting to me,
with how they described a migration to Vercel.

> When using an external CI (GitHub Actions for this site) to build before deploying to Vercel,
> I expected `vercel deploy --prebuilt` to still pick up the `vercel.json` at root (including the
> redirects and rewrites within), but it doesn't seem to do that. Switching to setting up the
> environment and building through `vercel pull` and `vercel build` (still on GitHub Actions, using
> [Vercel's guide for GitLab CI as a reference][gitlab-ref]) allowed the redirect rules to be applied, implying
> one of those (I guess `vercel build`) is propagating `vercel.json` settings to
> `.vercel/output/config.json` … I guess.
>
> &mdash; Kisaragi Hiu <https://kisaragi-hiu.com/switching-to-vercel/>

[gitlab-ref]: https://vercel.com/guides/how-can-i-use-gitlab-pipelines-with-vercel

Hmm, alright. Assuming the statement is right, there should be some magic that needs to happen,
and that would be `vercel build`. But how does it really affect `config.json`?

So let's create a minimal reproducible example!

```
/le-foobar-folder
  +- vercel.json
  +- public
  |  `- index.html        // just a simple lorem ipsum would do
  `- .vercel
      +- .env.development.local
      +- README.txt
      +- project.json     // vercel project settings
      `- output
          `- config.json  // this would only have version 3
```

> [!NOTE]
> To note, a bunch of project settings had to be pulled first with `vercel pull`.
> This would include the `buildCommand`, which has instructions on how to build the project.
> Since we're only doing a super basic example, we will be overriding `buildCommand` in `vercel.json`.

Here is how we configure our `vercel.json`:

```json
{
    "buildCommand": "node --version",
    "rewrites": [
        {
            "source": "/(.*)",
            "destination": "/index.html"
        }
    ]
}
```

`node --version` is to just get a 0 exit code. Feel free to use whatever you like.
The `rewrites` part is recommended based on the [Legacy SPA Fallback](https://vercel.com/docs/projects/project-configuration#legacy-spa-fallback).
In hindsight, it took an embarassingly long for me to find that documentation page.
And with that, we run `vercel build`.

```console
$ vercel build
Vercel CLI 34.2.7
v18.19.0
✅  Build Completed in .vercel/output [55ms]

$ cat .vercel/output/config.json
{
  "version": 3,
  "routes": [
    {
      "handle": "filesystem"
    },
    {
      "src": "^(?:/(.*))$",
      "dest": "/index.html",
      "check": true
    },
    {
      "handle": "error"
    },
    {
      "status": 404,
      "src": "^(?!/api).*$",
      "dest": "/404.html"
    }
  ],
  "crons": []
}
```

Well well well, what do we have here?

### So the `rewrites` became `routes` with `filesystem`

Interesting, so the configuration from `vercel.json` was translated into `config.json` during `vercel build`.
If we dissect the `config.json` further, we can see:

- The [`handle: "filesystem"`](https://vercel.com/docs/build-output-api/v3/configuration#handler-route)
  dictates that the following rules will take effect when the file is not found, which is what we want,
  "go to `index.html` if you cannot find the file".

- The [`check: true`](https://vercel.com/docs/build-output-api/v3/configuration#source-route) says that
  it triggers both `handle: "filesystem"` and `handle: "rewrite"`, but I don't have enough IQ to
  understand this.

- There is a continuation of handling error routes... which we didn't really need for a preview deployment,
  right? Or at least it is not a worry now.

So with that... we just have to tweak our final commands to update the `config.json` properly.

```yml
env:
  VERCEL_CONFIG: |
    {
      "version": 3,
      "routes": [
        { "handle": "filesystem" },
        {
          "src": "^(?:/(.*))$",
          "dest": "/index.html",
          "check": true
        }
      ]
    }

run: |
  echo "${VERCEL_CONFIG}" > ./.vercel/output/config.json
  npm i -g vercel
  vercel pull --environment=preview
  vercel deploy --prebuilt
```

### Done

And we're done. That's a lot of exploration, just to make a plain boring SPA work in Vercel, indeed.

---

#### References

- <https://vercel.com/docs/build-output-api/v3>
- <https://v17.angular.io/guide/deployment#fallback-configuration-examples>
- <https://kisaragi-hiu.com/switching-to-vercel/>
- <https://vercel.com/docs/projects/project-configuration#legacy-spa-fallback>
- <https://vercel.com/docs/build-output-api/v3/configuration#handler-route>
- <https://vercel.com/docs/build-output-api/v3/configuration#source-route>
