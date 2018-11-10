---
title: "Why I Love Hugo"
date: 2018-11-10T09:53:26-05:00
draft: false
---
I've tried a few static site generators in the past, but I think [Hugo](https://gohugo.io) has been the most pleasant experience I've had with one to date.  One of the biggest strengths of Hugo is that it's **fast**.  Being written in Go obviously has its advantages.  

On top of that, Hugo leverages Go's templating syntax, which is extremely easy and intuitive.  For example, you can just set a variable in the following file like so:

```
// <root>/data/var.toml
a_var = "test"
```

and reference it in your template like so:

```
<p class="{{ .Site.Data.var.a_var }}">Hugo is the best</p>
```

##### TOML Configuration Files #####

Hugo supporst TOML configuration files.  The toml syntax is easy to compose and super clear to understand.  For example, a complex data structure like a list of key/value pairs is very nice to declare:

```
# <root>/data/artists.toml

[[albums]]
name: Michael Jackson
album: Thriller
genre: Pop

[[albums]]
name: Pink Floyd
album: The Dark Side of the Moon
genre: Classic Rock
```

You can loop through the above key/value pairs in your template in the following manner:

```
{{ range .Site.Data.albums }}
<div class="album">
    <p class="name">{{ .name }}</p>
    <p class="album">{{ .album }}</p>
    <p class="genre">{{ .genre }}</p>
</div>
{{ end }}
```

If you really would rather use YAML or JSON, you can use those too.  You can even mix and match between all those file formats without having to fiddle with a single setting, Hugo automagically reads all of them!


##### PIPES #####

One of the other neat features is [pipes](https://gohugo.io/news/0.43-relnotes/).  This makes it super easy to process your css/js files with or do any other crazy transformations you can think of:

``` 
<head>
    {{ $styles := resources.Get "scss/main.scss" | toCSS | postCSS | minify | fingerprint }}
    <link rel="stylesheet" href="{{ $styles.Permalink }}" integrity="{{ $styles.Data.Integrity }}" media="screen">
</head>
```

##### SHORTCODES #####
Finally, I have to mention shortcodes.  Often times, when you're using a static site generator and writing markdown, you invariable need to start throwing in some custom html just to do some more advanced layout stuff, like specifying image sizes, etc.  Through Hugo shortcodes, you can essentially define your own html/css as custom markdown!  You can find an example [here](https://gohugo.io/content-management/shortcodes/)
.



