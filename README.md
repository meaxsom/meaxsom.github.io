# Building The Site

- Initially tried Jekyll and could not get it to theme correctly
- Switch to [Hugo](https://gohugo.io) where I have had some prior experience
- Finally got it up and working on 2004.03.20 after forking the theme

## Hugo Notes
- Items placed in `static` get copied in their directory structure to `public`
    - so for images that neeed to go in `public/images` put them in `static/images`

## How-To

- [How To](https://gohugo.io/hosting-and-deployment/hosting-on-github/) on build process for Hugo on GitHub

- How to add a theme
    - from the root of your hugo site:
    - `git submodule add https://github.com/{user}/{theme} themes/{theme-name}`
    - add `theme = '{theme-name}' to `hugo.toml` file

- A [How-To](https://www.adamormsby.com/posts/000/how-to-set-up-a-hugo-site-on-github-pages-with-submodules/) for Adding a theme as a submodle for hosted pages on github

- [FavIcons in Hugo](https://stackoverflow.com/questions/42043648/where-do-i-put-my-favicon-with-hugo)
    - [refresh](https://stackoverflow.com/questions/2208933/how-do-i-force-a-favicon-refresh)

- AI Based Image Generation via [krea.ai](https://www.krea.ai/)

## Notes

- [Tailwind Theme](https://github.com/tomowang/hugo-theme-tailwind)
