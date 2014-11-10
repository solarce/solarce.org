---
layout: page
title: tmux tricks
excerpt: "tmux tips"
modified: 2014-11-09
---

## zooming a pane

The recently released in **tmux 1.8** includes a new feature, **zoomed panes**, that allows temporarily expanding a pane to the full size of the tmux window to see more of its contents.

In the man page for tmux(1), the feature is described as follows, under the details for the resize-pane command:

> With -Z, the active pane is toggled between zoomed (occupying the
> whole of the window) and unzoomed (its normal position in the layout).

This command is bound to `<prefix> z` by default

_via [http://blog.sanctum.geek.nz/zooming-tmux-panes/](http://blog.sanctum.geek.nz/zooming-tmux-panes/)_

_added 2011-11-09_

---


## working with windows

**create a new window**

`tmux new-window (prefix + c)`

**move to the window based on index**

`tmux select-window -t :0-9 (prefix + 0-9)`

**rename the current window**

`tmux rename-window (prefix + ,)`


_via [http://robots.thoughtbot.com/a-tmux-crash-course](http://robots.thoughtbot.com/a-tmux-crash-course)_

_added 2011-11-09_