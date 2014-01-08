---
title: "Capistrano 3: Setting a Default Stage"
date: 2014-01-08 22:50 UTC
tags: rails,capistrano
---

In Capistrano v2, multi-stage support wasn't built in. If you didn't need stages, you could just happily
use `cap deploy` without any problem.

Capistrano v3 is a complete rewrite from v2, and with it comes built-in multi-stage support, even if you don't want it.
So even if you have just a single stage, let's say `production`, you still have to reference the stage every single time
you run your Capistrano tasks:

```
cap production deploy
```

If you try `cap deploy` without a stage, you will be told:

_Stage not set, please call something such as `cap production deploy`, where production is a stage you have defined._

If you used something like the multistage extension in v2, you would expect to be able to add `set :default_stage, "production"`
to your configuration. You will quickly find that this has no effect in v3, and if you try to run your tasks without specifying
a stage, you'll get the same error. The [issue has been brought up](https://github.com/capistrano/capistrano/issues/806)
but apparently having a single stage isn't a "use case." I disagree, so let's make it work.

---

Time to dig into Capistrano a little. Instead of using its own DSL, Capistrano is now a Rake application.
So what's really happening is that Capistrano is expecting a separate task for your stage to be run
__before__ the `deploy` task. In fact, the `deploy.rb` file isn't even loaded when you run `cap deploy`
without a stage.

When we look at Capistrano's `lib/capistrano/setup.rb` file, we find that the stage tasks do a couple things.
For each stage (the list of stages is created from the files in the `config/deploy/` directory), a Rake task is defined to:

1. Set the `:stage`
2. Load the capistrano defaults
3. Load the `deploy.rb` file
4. Load the `deploy/#{stage}.rb` file
5. Do a couple other things like setting SCM and locales

```ruby
# lib/capistrano/setup.rb

namespace :load do
  task :defaults do
    load 'capistrano/defaults.rb'
  end
end

stages.each do |stage|
  Rake::Task.define_task(stage) do
    set(:stage, stage.to_sym)

    invoke 'load:defaults'
    load deploy_config_path
    load stage_config_path.join("#{stage}.rb")
    load "capistrano/#{fetch(:scm)}.rb"
    I18n.locale = fetch(:locale, :en)
    configure_backend
  end
end
```

Now that we know this, we need to approach it a little differently. Rather than thinking _How do I set a default stage_,
we should be asking _How do I make sure that my production stage's task is run every time_.

Because this is just Rake, we already know the answer: simply invoke the `production` task by default.
Normally you would do this from your `Rakefile`, but since we're using Capistrano, you'll add this to the end of your `Capfile` instead:

```ruby
Rake::Task[:production].invoke
```

We can even use the `invoke` method from Capistrano's own DSL:

```ruby
invoke :production
```

__Note:__ We could have simply added a shell alias such as `alias cap='cap production'`, but that has plenty of its own downsides.

This may have some unintended consequences if you actually do use multiple stages, but if you only ever use the production stage, it should work fine.