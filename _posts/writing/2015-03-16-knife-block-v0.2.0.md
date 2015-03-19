---
layout: post
title: "knife-block v0.2.0 released"
excerpt: ""
categories: writing
tags: [chef, knife, knife-block]
comments: true
share: true
---

# knife-block v0.2.1 released.

_Important Update:_ v0.2.0 has been yanked because it wasn't actually working on Chef12/ChefDK.

[v0.2.1](https://github.com/knife-block/knife-block/releases/tag/v0.2.1) is out and on [rubygems.org](https://rubygems.org/gems/knife-block/versions/0.2.1)

This release includes:

- A fix so knife-block now works on Chef12
- Some updates to the documentation
- Some travis build improvements, including
  - Multiple 2.x Ruby builds
  - Building on 2.1 with chefdk
- A shiny logo

# The Future

Some future improvements I'm hoping to make in the coming months are:

- Get the code base happy with Rubocop
- Get CI builds going on OSX (w/ rvm and chefdk) and Windows builds
- Start working on any Windows related bugs that crop up
- Better integration with Berkshelf config stuff
