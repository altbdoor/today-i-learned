Colima and Docker fun
===

February 25, 2025

### Docker CLI is free!

Docker as a company has been somewhat controversial. But business is business.
And for me who is working in a very serious company right now, I might step on the wrong foot of business by running Docker Desktop.

> The Docker Engine is licensed under the Apache License, Version 2.0. See [LICENSE](https://github.com/moby/moby/blob/master/LICENSE) for the full license text.
>
> However, for commercial use of Docker Engine obtained via Docker Desktop within larger enterprises (exceeding 250 employees OR with annual revenue surpassing $10 million USD), a [paid subscription](https://www.docker.com/pricing/) is required.
>
> &mdash; [Docker licensing](https://docs.docker.com/engine/#licensing)

It is important to note, that Docker CLI is licensed under Apache license,
which as far as [tldr](https://www.tldrlegal.com/license/apache-license-2-0-apache-2-0) is concerned,
is completely valid to be used for commercial purposes.

So Docker Inc. really just made it an inconvenience to most developers,
by putitng Docker Desktop behind a paid subscription. For context,
I mostly build some experimental Docker images to test stuff by my own.


### There's Podman Desktop

One of the more popular alternatives is [Podman Desktop](https://podman-desktop.io/),
which attempts to replace both Docker CLI and Docker Desktop. For most parts,
Podman intends to be a drop-in replacement for Docker, and for most parts, it works.

I mostly struggle with two main aspects:

1. Some build problems with Alpine images and musl, and
2. Podman Desktop is very geared towards Kubernetes, while I really just needed Docker CLI

I tried it out for a week or so, before heading back to online forums for other recommendations.


### There's Colima

I briefly read through what [Colima](https://github.com/abiosoft/colima) does a couple of years back,
but I was too comfortable with Docker Desktop in a small company to be bothered.
A couple of online posts recommended using Colima, and so:

> what the hell, sure
>
> &mdash; me debugging a failed Docker build

The good news is, Colima is mostly on the CLI, which works for me
since I mostly worked with Docker CLI in... the CLI.

Installation was simple, since Colima is available in Homebrew.

```console
$ # docker package is docker cli btw
$ brew install colima docker
```

And with that, I can start building Docker images! Or can I...?

```console
$ colima start
INFO[0000] starting colima
INFO[0000] runtime: docker
INFO[0002] starting ...                                  context=vm
INFO[0013] provisioning ...                              context=docker
INFO[0014] starting ...                                  context=docker
INFO[0015] done

$ docker build -t jekyll-dev:latest .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/
```

So installing Docker alone from Homebrew, is not enough. Its really just plain Docker CLI,
without all its amazing plugins.

Docker build has been using Buildkit as the default since version 23.0 (February 2023).
Foolish me who had been using Docker Desktop all these time, did not even know that
Buildkit and Compose are separate plugins, because it is all bundled in!

Fortunately, Homebrew is here to save us once more.

```console
$ # i don't use compose for now, but you get the idea
$ brew install docker-buildx docker-compose
```

After installing, Homebrew included a post-installation message on how to setup Buildkit properly.

```
==> docker-buildx
docker-buildx is a Docker plugin. For Docker to find the plugin, add "cliPluginsExtraDirs" to ~/.docker/config.json:
  "cliPluginsExtraDirs": [
      "/opt/homebrew/lib/docker/cli-plugins"
  ]
```

...which I did, and failed for a couple of times. I'm not exactly sure why Docker wouldn't pick up Buildkit,
even when configured in the `config.json` directly. After some searching around the internet,
I decided to symlink Buildkit directly to where Docker would look.

```console
$ ln -s $HOMEBREW_PREFIX/bin/docker-buildx ~/.docker/cli-plugins/docker-buildx

$ docker info
Client: Docker Engine - Community
 Version:    28.0.0
 Context:    colima
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.21.1
    Path:     $HOME/.docker/cli-plugins/docker-buildx
```

I haven't tried using Compose plugin, but I assume things would be more or less the same.
Not too sure why the `config.json` didn't work, but symlinking two or three plugins is not too bad.


### Wrap it up

And that was it. I have to remember to `colima start` every time I want to Docker,
but I currently don't use Docker often enough for that to annoy me. Even if it does,
its most probably going to be a `colima start &` somewhere when the OS starts.

That was pretty fun. Maybe I'll really have to consider fully using Podman or something,
when Docker Inc. continues making [somewhat poor decisions](https://news.ycombinator.com/item?id=43125089).

But business is business.

---

#### References

- https://docs.docker.com/engine/#licensing
- https://www.tldrlegal.com/license/apache-license-2-0-apache-2-0
- https://podman-desktop.io/
- https://github.com/abiosoft/colima
- https://news.ycombinator.com/item?id=43125089
