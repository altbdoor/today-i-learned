Basic Jekyll building with GitHub Actions
===

February 9, 2021

I've been working on a [HTML/JS editor](https://github.com/altbdoor/trainstream-editor) for a website that I am helping out. While working in private, I used a local Jekyll binary, and had everything worked out pretty well. I wanted to make a monorepo of sorts, so I housed both the GitHub OAuth stuff (which is based on Cloudflare Workers because [CORS](https://stackoverflow.com/questions/62298627/github-api-cors-policy)), and the website code.

```
/
|- auth
|   |- index.js
|   |- package.json
|   `- wrangler.toml
|
`- website
    |- _config.yml
    |- Gemfile
    `- index.html
```

And I thought this worked well, since I have set the `source` in `_config.yml`.

```yml
source: website
```

But well... GitHub Pages have other plans.

> You can publish your site from any branch in the repository, either from the root of the repository on that branch, `/`, or from the `/docs` folder on that branch
> 
> &mdash; [About GitHub Pages](https://docs.github.com/en/github/working-with-github-pages/about-github-pages#publishing-sources-for-github-pages-sites)

So once GitHub Pages deploys, everything is broken.

Since I cannot depend on the automatic GitHub Pages deployment, I wrote a Bash script to automate some of the stuff. But I see how some people are actively using GitHub Actions to do things now. So I started to look into it a little bit, and find out even the Jekyll docs recommends GitHub Actions.

> When building a Jekyll site with GitHub Pages, the standard flow is restricted for security reasons and to make it simpler to get a site setup. For more control over the build and still host the site with GitHub Pages you can use GitHub Actions.
> 
> &mdash; [Jekyll docs on GitHub Actions](https://jekyllrb.com/docs/continuous-integration/github-actions/)

Basing a lot of my scripts from the [jekyll-action-ts](https://github.com/limjh16/jekyll-action-ts) repository, I came up with the following.

```yml
jobs:
  build:
    # run on the latest ubuntu github has
    runs-on: ubuntu-latest

    steps:
      # checkout to the master branch
      - uses: actions/checkout@v2

      # setup ruby with 2.7
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: 2.7

      # i'm too lazy to maintain a Gemfile.lock, so i generated the lock file on the go
      - name: generate gem file lock
        working-directory: website
        run: |
          bundle lock
      
      # use this action to... run jekyll and build the site
      - uses: limjh16/jekyll-action-ts@v2
        with:
          enable_cache: true
          jekyll_src: website

      # deploy!
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site
          force_orphan: true
```

With this series of workflows, I can generate the Jekyll site whenever the action gets triggered, which is whenever a commit goes into `master`.

I also fixed the version of Jekyll in the `Gemfile`, for... some sort of stability I guess, since I am completely missing a `Gemfile.lock`.

```rb
source 'https://rubygems.org'

gem 'jekyll', '~> 3.9'
gem 'kramdown-parser-gfm', '~> 1.1'
```

With Jekyll 4.x, everything seems to be fine. But I prefer to use the older Jekyll version to match the local binaries I have. With Jekyll 3.9.x, we need to [explicitly set the `kramdown-parser-gfm` dependency](https://stackoverflow.com/questions/63335953/jekyll-error-building-page-related-to-kramdown-parser), or the build will error out.

---

#### References

- https://stackoverflow.com/questions/62298627/github-api-cors-policy
- https://docs.github.com/en/github/working-with-github-pages/about-github-pages#publishing-sources-for-github-pages-sites
- https://jekyllrb.com/docs/continuous-integration/github-actions/
- https://github.com/limjh16/jekyll-action-ts
- https://stackoverflow.com/questions/63335953/jekyll-error-building-page-related-to-kramdown-parser
