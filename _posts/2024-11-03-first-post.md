---
title: Booting . . .
description: "Running startup script"
date: 2024-03-20
image: assets/lain-rig.jpeg
comments: false
toc: false
---

<!-- Insert a loading kind of thingy here markdown -->

```log
❯ cd ~/projects/blog
❯ ./tools/run.sh
> bundle exec jekyll s -l -H 127.0.0.1
You can add logger to your Gemfile or gemspec to silence this warning.
Configuration file: /Users/avramdj/projects/avramdj.github.io/_config.yml
            Source: /Users/avramdj/projects/avramdj.github.io
       Destination: /Users/avramdj/projects/avramdj.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.179 seconds.
 Auto-regeneration: enabled for '/Users/avramdj/projects/avramdj.github.io'
LiveReload address: http://127.0.0.1:35729
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

### Alright, lgtm.

```log
^C%
❯ git add _posts/2024-11-03-first-post.md
❯ git commit -m "First post"
❯ git push
```
