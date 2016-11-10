Building Jekyll for Windows
===

May 11, 2016

Before anything, I would like to get one thing straight. I do not know how to use [Ruby](https://www.ruby-lang.org/en/), at all. But I do have a lot of pleasant experience with [Jekyll](http://jekyllrb.com/) while working on GitHub. So when a project popped out which is mostly static, Jekyll comes to my mind.

My original setup on Windows did have [Ruby and RubyDevKit](http://rubyinstaller.org/) installed, but personally speaking, I find it rather hassling to download, install, `gem install`, and so on on another device (or for my colleagues). `py2exe` already exists for Python, so I believe there should be something for Ruby as well. If only I could plop a `jekyll.exe` and just start working... if only...

Fortunately, there are people who had researched on this matter. From a simple Google, I chanced upon [a blog post](http://www.nickw.it/jekyll-dot-exe/) where a man named [Nick Williams](https://github.com/nilliams) had actually built Jekyll for Windows. The first build can be found in his [GitHub repository](https://github.com/nilliams/jekyll.exe), which is v1.2.0. The file size is only 2.62 MB, which is really impressive. At this point, I pretty much consider it an end to my research, and started using Jekyll v1.2.0. Its old, but `./jekyll.exe serve --watch` does its job.

As the project grew a little more complex, I find the need to use `_data` folder. To no surprise, Jekyll v1.2.0 was not able to render the `site.data` content. I faintly remember it being introduced in v2.x, and from Nick's blog and repository, it does not appear that he maintains any other versions. Since the steps are all written down on the blog, I figured I should give it a shot as well.

I started by installing the Jekyll and [OCRA](http://ocra.rubyforge.org/) gems. I have to specify the version for Jekyll (`gem install jekyll:2.5.3`), or it will end up fetching v3.x. Why not build v3.x instead? Well, v2.5.3 has served me well enough, so I thought it would suffice. Plus, its nice to get all the stable versions covered. From here on, its a bit of hit and miss.

The command to build Jekyll with OCRA is like so:

```
ocra jekyll-2.5.3/bin/jekyll \
    jekyll-2.5.3/lib/jekyll/mime.types \
    jekyll-2.5.3/lib/site_template/**/* \
    jekyll-2.5.3/lib/site_template/*
```

After building, I tried it with the many subcommands (`serve`, `build`, `new`, etc). If it throws up an error, usually its verbose enough to let you know what went wrong. I found out I still need `webrick` and `jekyll-watch`, which I promptly install and include within `jekyll-2.5.3/bin/jekyll`. Now v2.5.3 pretty much works for me, but I found some issues with the syntax highlighter.

Firstly, v2.5.3 still uses [Pygments](http://pygments.org/), which is dependant on Python. Nope, not going to include Python in this. Using `./jekyll.exe new`, it looks like the default site template still uses the `{% highlight %}` block. Instead of using the block, I replaced them with the code fence blocks. By default, [Kramdown uses a syntax highlighter called CodeRay](http://kramdown.gettalong.org/rdoc/Kramdown/Options.html). Despite installing CodeRay and bundling it in, Jekyll still complains that it was not able to find CodeRay.

I later found out that it can be solved with an OCRA parameter `--gem-all`, but [Jekyll appears to be](https://jekyllrb.com/docs/templates/#code-snippet-highlighting) [moving towards using](https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0) [Rouge](http://rouge.jneen.net/) from v3.x onwards.

> Jekyll has built in support for syntax highlighting of over 60 languages thanks to Rouge. Rouge is the default highlighter in Jekyll 3 and above. To use it in Jekyll 2, set `highlighter` to `rouge` and ensure the `rouge` gem is installed properly.

Therefore, I decided to bundle Rouge in instead. However, I still use `--gem-all` with OCRA, to ensure that I did not miss out any libraries again. Just a tweak on `_config.yml` to make things good.

```yml
markdown: kramdown
kramdown:
  input: GFM # if you like GitHub Flavored Markdown
  syntax_highlighter: rouge
```

With all that done, I got myself a working Jekyll v2.5.3 at 3.86 MB. A tad larger than v1.2.0, but still acceptable. For more technical details, you can check out the [release page for my fork](https://github.com/altbdoor/jekyll-exe/releases/tag/stable-v2.5.3). I will hopefully build v3.x as well in the future (`--incremental` sounds good to have), but for now v2.5.3 serves me well.

---

#### References

- http://www.nickw.it/jekyll-dot-exe/
- http://kramdown.gettalong.org/rdoc/Kramdown/Options.html
- https://jekyllrb.com/docs/templates/#code-snippet-highlighting
- https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0
