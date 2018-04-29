# The Architect theme

[![Build Status](https://travis-ci.org/pages-themes/architect.svg?branch=master)](https://travis-ci.org/pages-themes/architect) [![Gem Version](https://badge.fury.io/rb/jekyll-theme-architect.svg)](https://badge.fury.io/rb/jekyll-theme-architect)

*Architect is a Jekyll theme for GitHub Pages. You can [preview the theme to see what it looks like](http://pages-themes.github.io/architect), or even [use it today](#usage).*

![Thumbnail of Architect](thumbnail.png)

## Usage

To use the Architect theme:

1. Add the following to your site's `_config.yml`:

    ```yml
    theme: jekyll-theme-architect
    ```

2. Optionally, if you'd like to preview your site on your computer, add the following to your site's `Gemfile`:

    ```ruby
    gem "github-pages", group: :jekyll_plugins
    ```

## Customizing

### Configuration variables

Architect will respect the following variables, if set in your site's `_config.yml`:

```yml
title: [The title of your site]
description: [A short description of your site's purpose]
```

Additionally, you may choose to set the following optional variables:

```yml
show_downloads: ["true" or "false" to indicate whether to provide a download URL]
google_analytics: [Your Google Analytics tracking ID]
```

### Stylesheet

If you'd like to add your own custom styles:

1. Create a file called `/assets/css/style.scss` in your site
2. Add the following content to the top of the file, exactly as shown:
    ```scss
    ---
    ---

    @import "{{ site.theme }}";
    ```
3. Add any custom CSS (or Sass, including imports) you'd like immediately after the `@import` line

*Note: If you'd like to change the theme's Sass variables, you must set new values before the `@import` line in your stylesheet.*

### Layouts

If you'd like to change the theme's HTML layout:

1. [Copy the original template](https://github.com/pages-themes/architect/blob/master/_layouts/default.html) from the theme's repository<br />(*Pro-tip: click "raw" to make copying easier*)
2. Create a file called `/_layouts/default.html` in your site
3. Paste the default layout content copied in the first step
4. Customize the layout as you'd like

### Overriding GitHub-generated URLs

Templates often rely on URLs supplied by GitHub such as links to your repository or links to download your project. If you'd like to override one or more default URLs:

1. Look at [the template source](https://github.com/pages-themes/architect/blob/master/_layouts/default.html) to determine the name of the variable. It will be in the form of `{{ site.github.zip_url }}`.
2. Specify the URL that you'd like the template to use in your site's `_config.yml`. For example, if the variable was `site.github.url`, you'd add the following:
    ```yml
    github:
      zip_url: http://example.com/download.zip
      another_url: another value
    ```
3. When your site is built, Jekyll will use the URL you specified, rather than the default one provided by GitHub.

*Note: You must remove the `site.` prefix, and each variable name (after the `github.`) should be indent with two space below `github:`.*

For more information, see [the Jekyll variables documentation](https://jekyllrb.com/docs/variables/).

### Previewing the theme locally

If you'd like to preview the theme locally (for example, in the process of proposing a change):

1. Clone down the theme's repository (`git clone https://github.com/pages-themes/architect`)
2. `cd` into the theme's directory
3. Run `script/bootstrap` to install the necessary dependencies
4. Run `bundle exec jekyll serve` to start the preview server
5. Visit [`localhost:4000`](http://localhost:4000) in your browser to preview the theme

### Running tests

The theme contains a minimal test suite, to ensure a site with the theme would build successfully. To run the tests, simply run `script/cibuild`. You'll need to run `script/bootstrap` one before the test script will work.


kevinkelley.github.io
=====================

engineer@large

> “What did I want? I wanted a Roc's egg.  ...I wanted raw red gold in nuggets the size of your fist... I wanted to hear the purple water chuckling against the skin of the Nancy Lee in the cool of the morning watch and not another sound, nor any movement save the slow tilting of the wings of the albatross that had been pacing us the last thousand miles.  I wanted the hurtling moons of Barsoom. I wanted Storisende and Poictesme, and Holmes shaking me awake to tell me, "The game's afoot!"  I wanted to float down the Mississippi on a raft and elude a mob in company with the Duke of Bilgewater and Lost Dauphin.  I wanted Prester John, and Excalibur held by a moon-white arm out of a silent lake.  I wanted to sail with Ulysses and with Tros of Samothrace and to eat the lotus in a land that seemed always afternoon.  I wanted the feeling of romance and the sense of wonder I had known as a kid.  I wanted the world to be the way they had promised me it was going to be, instead of the tawdry, lousy, fouled-up mess it is.  I had had one chance ... Maybe one chance is all you ever get.”
― Robert A. Heinlein, Glory Road

I am a freelance programmer, always interested in new challenges.  I've been all up and down the stack.  I like modern type-safe languages... Rust is a big current love of mine.  When I'm on JVM I like Kotlin, I think it's what Java wanted to be.

I'm probably available for work, contact me if you've got something interesting needs done, maybe I can help.

best!
