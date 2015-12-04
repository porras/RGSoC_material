# Rake

Rake is a Ruby program, included with the Ruby distribution, inspired by Make. Both Make and Rake were originally conceived to compile other programs, that's why they are called **build tools**. In fact, both Make and Rake can be used for a wider variety of tasks.

Sad but beautiful fact about Rake: [https://github.com/jimweirich/wyriki/commit/d28fac7f18aeacb00d8ad3460a0a5a901617c2d4](https://github.com/jimweirich/wyriki/commit/d28fac7f18aeacb00d8ad3460a0a5a901617c2d4)

## Things Rake is good for

* Tasks that involve generating files using other files as base
* Tasks that have dependencies between each other

## How Rake works

Rake reads the definition of the tasks from a file called `Rakefile`. Many projects contain one, with tasks that are relevant for that project. Ways of defining tasks:

### `task`

`task`: defines a task, expressed in that block:

```ruby
task :hello do
  puts 'Hello'
end
```

A task can have dependencies:

```ruby
task :explain => :hello do
  puts 'This is how Rake dependencies work'
end
```

Even more than one:

```ruby
task :ask => [:explain, :hello] do
  puts 'Do you know now how Rake works?'
end
```

Rake will be clever not to execute a task twice (note how if it weren't, `hello` would be done twice).

A task can consist of only dependencies:

```ruby
task :all => [:hello, :explain, :ask]
```

And in fact, there is a special task called `default` that is usually used this way. It's the task that will be invoked if we just call `rake`.

```ruby
task :default => :all
```

#### Passing parameters to tasks

Rake has a way to pass parameters to tasks, but it's a bit weird, so most people just use environment variables:

```ruby
task :salute do
  puts "Hello, #{ENV.fetch('NAME', 'World')}!"
end
```

### `file`

`file` is similar to `task`: it defines how to do something, optionally with dependencies. The difference is that it's used specifically to generate a file. And the reason that we want Rake to know that it's a file is because that way it can be clever enough to decide not to run it, if the file already exists, and none of the dependencies changed:

```ruby
file 'checksum.md5' => 'rake.md' do
  sh 'md5sum rake.md > checksum.md5'
end
```

In this example we are seeing that files can also be used as dependencies (also for normal tasks)

### `rule`

`rule` is once again similar to `file`, but in this case, it defines the way of creating a lot of files whose name follow a pattern, from other files that also follow a pattern:

```ruby
rule ".html" => ".md" do |t|
  sh "markdown #{t.source} > #{t.name}"
end
```

#### Documenting and organizing our Rake tasks

### `desc`

`desc` provides a description for the following `task` or `file` (not `rule`, because a `rule` can be invoked in many different ways). They are used by `rake -T`.# 

```ruby
desc 'Say hello'
task :hello do
  puts 'Hello'
end
```

### `namespace`

`namespace` groups all tasks defined in the block. They will be invoked with `rake namespace:task`

```ruby
namespace :tutorial do
  desc 'Say hello'
  task :hello do
    puts 'Hello'
  end
end
```

**Important:** `file`s and `rule`s defined inside the block are not affected by the namespacing.

#### This weird trick

##### FileList

Rake contains a lot of utilities for managing files and commands. We already saw one, `sh`. Another almost random example (but a very useful one) is `FileList`. With it, we can define a list of files, using a pattern, with which we can do a lot of things, among others, add it as a dependency or, my favorite, define an alternative list with the extensions changed:

```ruby
task :html => Rake::FileList.new('*.md').ext('.html')
```

##### Processing dependencies in parallel

When dependencies don't depend between each other, we can tell Rake to process them in parallel by using `multitask` instead of `task`. While this can be often useful to run our tasks faster, we should keep it for simple cases because it can lead to problems (for example, if the tasks we're running in parallel all depend in other task, we might end up running that task more than once).

##### Debugging tasks and their dependencies

Calling rake with the `--trace` option will print some additional information that can be useful to debug tasks that are not behaving the way we expect.

```
$ rake --trace html
** Invoke html (first_time)
** Invoke README.html (first_time)
** Invoke README.md (first_time, not_needed)
** Execute README.html
markdown README.md > README.html
** Invoke parsing_arguments_and_options.html (first_time)
** Invoke parsing_arguments_and_options.md (first_time, not_needed)
** Execute parsing_arguments_and_options.html
markdown parsing_arguments_and_options.md > parsing_arguments_and_options.html
** Invoke rake.html (first_time)
** Invoke rake.md (first_time, not_needed)
** Execute rake.html
markdown rake.md > rake.html
** Invoke ruby_objects_and_classes.html (first_time)
** Invoke ruby_objects_and_classes.md (first_time, not_needed)
** Execute ruby_objects_and_classes.html
markdown ruby_objects_and_classes.md > ruby_objects_and_classes.html
** Execute html

$ rake --trace html
** Invoke html (first_time)
** Invoke README.html (first_time, not_needed)
** Invoke README.md (first_time, not_needed)
** Invoke parsing_arguments_and_options.html (first_time, not_needed)
** Invoke parsing_arguments_and_options.md (first_time, not_needed)
** Invoke rake.html (first_time, not_needed)
** Invoke rake.md (first_time, not_needed)
** Invoke ruby_objects_and_classes.html (first_time, not_needed)
** Invoke ruby_objects_and_classes.md (first_time, not_needed)
** Execute html

### Documentation

* [http://rake.rubyforge.org/](http://rake.rubyforge.org/)
* [http://jasonseifer.com/2010/04/06/rake-tutorial](http://jasonseifer.com/2010/04/06/rake-tutorial)
