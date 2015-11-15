# Command line arguments

Commands accept arguments. Whenever in a shell we're running anything that's more than one word, we're passing arguments:

* `cd Code`
* `ls -l`
* `ruby script.rb`
* `bundle install`

## In Ruby

Ruby gives us a very simple way to get the arguments that got passed into a Ruby program

```ruby
p ARGV
```

    $ ruby example.rb 
    []
    $ ruby example.rb hello
    ["hello"]
    $ ruby example.rb hello world
    ["hello", "world"]

# Command line arguments parsing

There are some conventions about how programs should interpret the arguments they are passed. This is useful because in this way it's easier to learn to use new programs. These conventions are described in an stardard called POSIX which we **don't** need to read :D This is a summary:

* Arguments starting with a dash (`-`) are options: `-l`
* Options usually have a short and long version, starting with one dash (`-`) or two (`--`). They do the same: `--long`
* Options can (optionally) expect a value. This value just follows the option (short version) or is introduced with an equal sign (`=`): `-f my_file.txt` or `--file=my_file.txt`
* There are a couple of subtleties we can see at the end, or not :)

## In Ruby

Ruby has a library that helps us implement options that follow these conventions, avoiding us a lot of manual handling with `ARGV`. It's called `optparse`. We're going to see how it works in detail.

[Note: there are some gems that make it even easier. We're going to stick to `optparse` because 1) It's good enough in my view and 2) this way we avoid adding dependencies to our program, which is sometimes something useful. But you might want to explore on your own, they're nice!]

The main class in this library is `OptionsParser`. Its constructor accepts a block, in which we'll configure the accepted options:

```ruby
require 'optparse'

OptionParser.new do |opts|
  ...
end
```  

The main method we'll call on this `opts` object is `on`. `on` configures an option, and provides a block to handle it when it's passed. It accepts four parameters (but some of them are optional):

* A short version of the option
* A long version of the option
* A description of the option
* A class to which the option argument will be converted

Some examples:

```ruby
opts.on('-h', '--help', 'Show the help') { ... }
opts.on('-n N', '--number=N', 'Sets n', Integer) { |n| ... }
opts.on('-r[N]', '--retry=[N]', 'Retry failures N times (default: 1)', Integer) { |n| ... }
```

`on` has a couple of variants `on_head` and `on_tail` to ensure that an option is the first or last (not very useful because otherwise they are processed in order).

Something useful to know is that `opts.to_s` returns a nicely formatted summary of the supported options, so implementing `-h`/`--help` is really easy!

```ruby
opts.on('-h', '--help', 'Show the help') { puts opts; exit }
```

To add additional niceness to that summary we can use `options.banner=` (to define a message that will be printed before it) and `options.separator` (to group options together).

Finally, when we call `parse!` on the `OptionParser`, it will run all the relevant handlers. A nice thing about `parse!` is that it removes the parse it already *used* from `ARGV`, so we can manage what is left there, usually as main arguments (not part of an option).

A complete example:

```ruby
require 'optparse'

options = {
  verbose: false,
  number: 5,
  retry: false
}

ARGV << '-h' if ARGV.empty?

OptionParser.new do |opts|
  opts.banner = "Usage: ruby #{__FILE__} [options] <file1> <file2>"

  opts.separator('Retries')

  opts.on('-r[NUMBER]', '--retry=[NUMBER]', 'Retry on failure (NUMBER times, default 1)', Integer) do |n|
    options[:retry] = true
    options[:n_retries] = n || 1
  end

  opts.separator('Other options')

  opts.on('-n NUMBER', '--number=NUMBER', 'Set number to NUMBER', Integer) { |n| options[:number] = n }
  opts.on('-v', '--verbose', 'Print extra information') { options[:verbose] = true }
  opts.on_tail('-h', '--help', 'Show this help') { puts opts; exit }
end.parse!

options[:files] = ARGV

require 'pp'

pp options
```
