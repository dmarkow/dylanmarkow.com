---
title: "Load Order with RubyMotion"
date: 2012-05-06 16:44
comments: true
categories:
---
Because of the way [RubyMotion][] compiles your application, you may run into dependency issues between your ruby files. By default, RubyMotion uses a simple `Dir.glob` command to load your files:

``` ruby
Dir.glob(File.join(app.project_dir, 'app/**/*.rb'))
```

Suppose you create a custom module under `app/lib/foo.rb` with some convenience methods:

``` ruby
module Foo
  def self.bar
    "bar"
  end
end
```

And then you include it in your `AppDelegate` class:

``` ruby
class AppDelegate
  include Foo

  def application(application, didFinishLaunchingWithOptions:launchOptions)
    @bar = self.bar
    true
  end
end
```

When you run `rake`, you may run into this:

    (main)>> 2012-05-06 16:58:49.270 app[88253:f803] *** Terminating app due to uncaught exception 'NameError', reason: 'uninitialized constant AppDelegate::Foo (NameError)
    '
    *** First throw call stack:
    (0x156a022 0x1b0cd6 0xd1c34 0x2377 0x2225)
    terminate called throwing an exception

The basic solution, which [RubyMotion's documentation suggests][doc], is to use the `app.files_dependencies` option in your `Rakefile`:

``` ruby
Motion::Project::App.setup do |app|
  # Use `rake config' to see complete project settings.
  app.name = 'app'
  app.files_dependencies 'app/app_delegate.rb' => 'app/lib/foo.rb'
end
```

However, if you are building a large app, and want `Foo` to be available to all your classes, you probably don't want to have dozens of `'app/baz.rb' => 'app/lib/foo.rb'` entries. In my project, I'm just altering the array of files from `Dir.glob` instead, ensuring my modules get loaded first:

``` ruby
app.files = Dir.glob(File.join(app.project_dir, 'app/lib/**/*.rb')) |
            Dir.glob(File.join(app.project_dir, 'app/**/*.rb'))
```

[RubyMotion]:http://www.rubymotion.com
[doc]:http://www.rubymotion.com/developer-center/guides/project-management/#_files_dependencies