# Building The Site

- Initially tried Jekyll and could not get it to theme correctly
- Switch to [Hugo](https://gohugo.io) where I have had some prior experience

## Hugo Notes
- Items placed in `static` get copied in their directory structure to `public`
    - so for images that neeed to go in `public/images` put them in `static/images`

## Notes

- How to add a theme
    - from the root of your hugo site:
    - `git submodule add https://github.com/{user}/{theme} themes/{theme-name}`
    - add `theme = '{theme-name}' to `hugo.toml` file

- AI Based Image Generation via [krea.ai](https://www.krea.ai/)

## Notes

https://github.com/tomowang/hugo-theme-tailwind
