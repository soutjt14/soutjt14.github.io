+++
title = "Get (got?) It Working"
+++

# How the site works

Everything is hosted on my personal
[github](https://github.com/soutjt14/soutjt14.github.io).

To have a successful zola build, I added the standard necessary directories:
- content
- templates
- themes

Zola projects can also have `static` and `sass` directories, but I didn't try
creating anything that would go in one of those, so my project doesn't have
them.

My development environment leaves something to be desired. I just have a
Windows laptop to play with, so I didn't try to get access to `zola` locally.
Oh well, I'll just push lots of bad commits and learn on-the-fly in CI. (And
[bad CI](https://github.com/soutjt14/soutjt14.github.io/commits/main/) is
exactly what I got.)

Something else to be desired: I don't have the ability to run CLI git. Sure, I
could probably figure this out if I put in epsilon effort. But I didn't. So my
only way to get content from other projects is to copy-paste themes into my own
themes directory and run commits with Visual Studio. No submodules for me.
(Worth noting that I could have also moved the git clone step for the themes
into my CI process, but as you'll see in the next section, I'm using pre-built
CI steps and didn't try too hard to make anything myself.)

Some notes about the files I added:
- My config.toml is pretty basic, but might still have some artifacts from when
I was trying to use the `after-dark` theme. Now I'm using `just-the-docs`.
- I couldn't get zola to build with `index.md` in my content directory. Maybe
serving a page as the root...page? on a static site is a zola no-no. Or maybe
I'm doing something dumb elsewhere that's telling zola to expect a section as
the root page.
- Early on I had errors where zola complained about an empty templates
directory. It's unclear if I could have gotten around this by removing the
templates directory entirely, but I went the route of adding a `file.txt` to
the directory. Well, it's not a template by any stretch of the imagination. But
zola is happy.

## Github Actions

To get zola builds in CI, I followed the instructions
[here](https://www.getzola.org/documentation/deployment/github-pages/) to
configure my github actions, i.e., the workflow(s) that run every time I push
to the project. Github Actions can be configured with yaml files in a project's
[.github/workflows](https://github.com/soutjt14/soutjt14.github.io/tree/main/.github/workflows)
directory. My workflow looks like this:

```yaml
# On every push this script is executed
on: push
name: zola-build-and-deploy
jobs:
  build:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: checkout
        uses: actions/checkout@v4
      - name: build_and_deploy
        uses: shalzz/zola-deploy-action@v0.18.0
        env:
          # Target branch
          PAGES_BRANCH: gh-pages
          # Provide personal access token
          #TOKEN: ${{ secrets.TOKEN }}
          # Or if publishing to the same repo, use the automatic token
          TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

This effects the build in a series of steps:
- First `actions/checkout` pulls a new copy of my project's main branch.
- Then `shalzz/zola-deploy-action` does a few things:
  - It runs `zola build`, which creates the html pages you're reading right now.
  - It `git push`es the created website to my project's `gh-pages` branch.

Really what's going on is these two `steps` are running some `docker run`
commands with a bunch of env variables set to use my project. The source code
for all this is hosted publicly in github, so I poked around in the deploy action's
[entrypoint](https://github.com/shalzz/zola-deploy-action/blob/master/entrypoint.sh)
at various points to help myself debug problems I encountered.

After the above workflow runs, I still don't have pages - nothing has been
published yet. So the last step is to configure the project to build pages off
of the `gh-pages` branch. This is extra nice because the pages job is watching
the branch my workflow is pushing to, so there's built-in dependency without
needing to configure anything. If the zola build fails, `gh-pages` doesn't
update and the pages job never runs.