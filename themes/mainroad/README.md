# Mainroad

**Mainroad** is a responsive, simple, clean and content-focused [Hugo](//gohugo.io/) theme based on the [MH Magazine lite](//wordpress.org/themes/mh-magazine-lite/) WordPress theme by [MH Themes](//www.mhthemes.com/).

**[Alternate Demo (Best)](//hugothemes.gitlab.io/mainroad/)** | [Standart Demo](http://themes.gohugo.io/theme/mainroad/)

![screenshot](https://github.com/Vimux/mainroad/blob/master/images/screenshot.png)

Adaptation has several changes:

+ Include Hugo internal templates (Open Graph, Disqus, Google Analytics)
+ New responsive menu without jQuery
+ SVG icons instead of font icons
+ Theme options are saved (Sidebar position, Author Box, Post Navigation) and available through config.toml file parameters
+ Other small changes

## Installation

In your Hugo site `themes` directory, run:

```
$ git clone https://github.com/vimux/mainroad
```

Next, open `config.toml` in the base of the Hugo site and ensure the theme option is set to `mainroad`:

```
theme = "mainroad"
```

For more information read the official [setup guide](//gohugo.io/themes/installing/) of Hugo.

## Configuration

### Example config.toml

```toml
baseurl = "/"
title = "Mainroad"
languageCode = "en-us"
paginate = "10" # Number of posts per page
theme = "mainroad"
disqusShortname = "" # Enable comments by entering your Disqus shortname
googleAnalytics = "" # Enable Google Analytics by entering your tracking id


[Author]
    name = "John Doe"
    bio = "John Doe's true identity is unknown. Maybe he is a successful blogger or writer. Nobody knows it."
    avatar = "img/avatar.png"


[Params]
    subtitle = "Just another site" # Subtitle of your site
    description = " John Doe's Personal blog about everything" # Description of your site
    opengraph = true
    leftsidebar = false # Move sidebar to the left side if true
    authorbox = true
    post_navigation = true


[Params.widgets]
    search = true # Enable "Search" widget
    recent_articles = true # Enable "Recent arcticles" widget
    categories = true # Enable "Categories" widget
    tags = true # Enable "Tags" widget
```

## TODO

- [ ] Delete unused CSS
- [x] No ID in CSS
- [x] Responsive menu without jQuery
- [ ] Delete unused JS
- [ ] i18n support (Internationalization)
- [ ] Detailed documentation (?)
- [ ] Social media widget (?)
- [ ] Enhanced version of recent posts widget (?)
- [ ] Tabbed widget (?)

## Contributing

Have you found a bug or got an idea for a new feature? Feel free to use the [issue tracker](//github.com/Vimux/mainroad/issues) to let me know. Or make directly a [pull request](//github.com/Vimux/mainroad/pulls).

## License

This theme is released under the [GPLv2 license](//github.com/Vimux/mainroad/blob/master/LICENSE.md) (inherited from the original MH Magazine lite WordPress theme).