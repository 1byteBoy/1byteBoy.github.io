<h1 align="center"> Hi Everyone </h1>
<p align="center" style="font-size:1.5em"> I am 8bitBoy </p>

This is my blog site source code. I built the site using [jekyll](https://jekyllrb.com/), a static site generator.
<img align="right" width="100" src="https://jekyllrb.com/img/logo-2x.png" width=100px/>

## Theme

The theme i am using for my blog is [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy).\
The devs also provides a starter template called [chirpy-starter](https://github.com/cotes2020/chirpy-starter), which i used as my base template 
<img align="right" width="100" src="https://chirpy-img.netlify.app/commons/avatar.jpg" style="border-radius: 25px;">

## Customization

Although the starter template is totally fine, there are still some limitation about various customization options like fonts, colors, site logic etc.

This can be easily avoided by using the original chirpy theme repo [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) and making changes to specific files.

To keep things simple the devs created the [chirpy-starter](https://github.com/cotes2020/chirpy-starter) template but if we want to customize anything, they have also provided an option to do so in the following documentation

https://chirpy.cotes.page/posts/getting-started/#customizing-the-stylesheet

In short we just have to copy and edit the files that holds the core logic or functionality for the specific thing we want to customize

## My Customizations

While the default font is fine, i personally like fonts with bold rounded texts. So as i was writing my first writeup for my blog i stumbled upon this website [vsociety_](https://www.vicarius.io/vsociety/), they were using [IBM Plex Mono](https://fonts.google.com/specimen/IBM+Plex+Mono) from [Google Fonts](https://fonts.google.com/). I liked it and i used it, simple

For the font, i added an entry in `_data/origin/cors.yml` and change the link for `webfonts:`

For the font to take effect, i made two new entries `_sass/main.sass` and `_sass/variables-hook.sass` and copied in the original content and made changes in `variables-hook.sass`

```css
$font-family-base: 'IBM Plex Mono', monospace;
$font-family-heading: 'IBM Plex Mono', monospace;
```

With the font change, i noticed some UI get broken, which i fixed it in `assets/css/jekyll-theme-chirpy.css`

With that done, the site was not building due to some sass syntax issue due to version problem, which i fixed by adding the following in `Gemfile`

```
gem "jekyll-sass-converter", "~> 2.0"
```

Finally run `bundle update` to update the gem configuration and everything should build, run and show properly.

## Github Pages UI Issue

One of the issue i was facing, which i had got no idea for a long time until now, is a total UI getting broken, after github pages build. I though it was due to my customization but it was running and building just fine locally.

The issue was in a environmental variable that was set in github workflow file `.github/workflow/jekyll.yml` while the site was build.

To test how it breaks run (locally) `JEKYLL_ENV=production bundle exec jekyll serve`

The fix is to remove the following from the `jekyll.yml` file 

```yml
env:
  JEKYLL_ENV: production
```

