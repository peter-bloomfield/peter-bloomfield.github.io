# Peter Bloomfield's blog
This repository contains the contents of my personal blog about software engineering. It uses the
[Jekyll](https://jekyllrb.com/) static site generator, with the [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy/).

This repository probably won't be very useful to anyone else, but you're welcome to take a look at
how it's built. If you want to build your own similar blog, please see the Jekyll and Chirpy
documentation at the above links.

# Build instructions
This works best on a Linux-like platform, including Windows Subsystem for Linux. It is possible on
regular Windows though.

Ensure you have Ruby installed. You will need the `gem` command.

Before building for the first time, install Jekyll and bundler like this:

```
gem install jekyll bundler
```

To build the site and immediately host it locally for testing, run this command:

```
bundle exec jekyll serve
```

The command output shows the local server address. It is usually localhost:4000

To build the site without hosting it:

```
bundle exec jekyll build
```

The build output will be in the `_site` folder.
