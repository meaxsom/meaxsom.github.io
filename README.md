# Building The Site

## Getting Started

I started off using the standard GitHub tutorials for hosting pages on github.io and made good progress.

Using VS Code I built a Jekyll [dev container](https://code.visualstudio.com/docs/devcontainers/containers) so I didn't pollute my local Mac with installing the Ruby tools.

It was pretty easy to get the templated site up and running with the default "how to's" but when I went to install a theme it got intresting. I ended up with a "white screen of death". Along the way, I stumbling my way through github "actions" watching the output of the deploy pipeline to see what went wrong. Eventually I ended up backing the theme out to get the skeleton site back to a stable state.

While the auto-build on the github end is nice and all, it's really just deploying a static site out of the Jekyll build process, which means this could be built locally and then just upload the `_site` directory for deployment. In fact I could use just about any [static site generator](https://jamstack.org/generators/), e.g [Hugo](https://gohugo.io/) if I wanted to.

I ended up following this tutorial on [Making GitHub Pages Work With Jekyll 4+ and Any Theme and Plugin ](https://www.moncefbelyamani.com/making-github-pages-work-with-latest-jekyll/). Because I had started my repo before this I had to juggle the deployment settings a bit and alter some of the specific versions of things in the Gemfile. Othewise it went fairly well.

