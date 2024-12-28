+++
title = "How To Pages"
paginate_by = 4
+++

# How the site works

Everything is hosted on my personal
[github](https://github.com/soutjt14/soutjt14.github.io).

To have a successful zola build, I added the standard necessary directories:
- content
- templates
- themes

For my Christmas card, I also added a `static` directory to hold some images
that `zola` integrates into the webpages.

Zola projects can also have a `sass` directory, but I didn't attempt to create
one of those since the theme I'm using provides reasonable defaults.

I do development on my Windows laptop. I downloaded zola with `winget` following
the `getzola.org` installation
[instructions](https://www.getzola.org/documentation/getting-started/installation/#windows).
Then in a terminal I have `zola serve` running, which rebuilds the site as I
make changes to the source code, letting me see how the site looks in my browser
locally.

I use Visual Studio as my IDE. Its git interactions leave a bit to be desired
(more likely I juse use it badly), so I also installed CLI git to let myself do
things like `git mv <file> <file>` without breaking the git history tracking.

For site theming, I hardcoded the `just-the-docs` theming into my `themes`
directory. I've made a few edits to it, including font and font size tweaks.

Some notes about the files I added:
- My config.toml is pretty basic, but might still have some artifacts from when
I was trying to use the `after-dark` theme. Now I'm using `just-the-docs`.
- I couldn't get zola to build with `index.md` in my content directory, but
  `_index.md` works. Maybe this is just how `zola` wants it to be.
- I noticed that the menu links weren't rendering correctly. I dug into the
`just-the-docs` templates and noticed that the `href` being rendered didn't
have the base URL, just the page name. Rather than figure out the correct zola
thing to do, I copied the template macro into my own project and re-worked the
[nav function](https://github.com/soutjt14/soutjt14.github.io/blob/main/templates/macros/nav.html#L24)
to use an `href` with my pages URL. Now the menu links work, but the `index`
link at the top of the page seems to make my browser go haywire. Progress?
  - Recently I noticed this tweak I made means when I'm serving the site locally
    I can't use the nav bar links to get around, since they have the github
    pages URL hardcoded.

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