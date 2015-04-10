---
layout: post
title: "Sidekiq doesn't load Rails code changes automatically, be awared of it"
date: 2015-04-10
image: /images/myself.png
tags: rails sidekiq ruby
---

It's easy for Rails developers to fall into thinking that Sidekiq loads their code automatically because they are too familiar with Rails's autoload feature. However, it's totally wrong. If you fixed a bug in a job code but didn't restart Sidekiq workers, the bug might be still there and you might be like "WHAT".

The reason why Sidekiq doesn't auto load the codes because Rail's autoloading is only in development and it's not **thread-safe** so it got disabled. That means there's no autoloading with sidekiq workers.

## Conclusion

 * Restart sidekiq workers whenever you changed job codes
