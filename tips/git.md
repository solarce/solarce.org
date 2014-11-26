---
layout: page
title: tmux tricks
excerpt: "tmux tips"
modified: 2014-11-09
---

## nicer (imo) git log

A nicer look for git log

First run `git config --global alias.l "log --graph --decorate --all --max-count 5 --name-status"`

Then run `git l`

It should look something like

```
* commit 7110cbba9dad7fffbf3bdf137d39f09fec277f16 (HEAD, tag: v1.0.5, origin/master, origin/HEAD, master)
| Author: Brandon Burton <brandon@inatree.org>
| Date:   Mon Oct 27 14:42:19 2014 -0700
|
|     version 1.0.5
|
|     - This is a pretty substantial rewrite of this cookbook
|     - It now uses
|     [YAMLConfig](ttps://github.com/jmxtrans/jmxtrans/wiki/YAMLConfig) for
|     config generations.
|     - You should review the README and see
|     ttps://github.com/solarce/jmxtrans-wrapper
|
| M     Berksfile.lock
| A     CHANGELOG.md
| M     README.md
| D     Rakefile
| M     attributes/default.rb
| A     files/default/templates_yaml/example_yaml_template.yaml.erb.sample
| A     libraries/config_render.rb
| M     metadata.rb
| A     recipes/config_render.rb
| D     recipes/default.rb
| A     recipes/install.rb
| A     recipes/service.rb
| M     templates/default/jmxtrans.upstart.conf.erb
| D     templates/default/jmxtrans_config.json.erb
| M     templates/default/jmxtrans_default.erb
| D     test/.chef/knife.rb
| D     test/support/Gemfile
| D     test/support/Gemfile.lock
|
* commit 3866e7cdc535e757d153d48d40ced1a03f53c734
| Author: Brandon Burton <brandon@inatree.org>
| Date:   Fri Oct 10 08:28:16 2014 -0700
|
|     adding .gitignore
|
| A     .gitignore
|
* commit 0db90a064a1eb03181868ab084f0c6f8588bbf9a (tag: v0.0.5)
| Author: Brandon Burton <brandon@inatree.org>
| Date:   Fri Sep 26 23:20:37 2014 -0700
|
|     moar fixes
|
| M     Berksfile.lock
```

_added 2014-11-03_

---

