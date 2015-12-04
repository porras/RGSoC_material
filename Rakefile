namespace :tutorial do
  desc 'Say hello'
  task :hello do
    puts 'Hello'
  end

  desc 'Explain Rake'
  task :explain => :hello do
    puts 'This is how Rake dependencies work'
  end

  desc 'Ask about Rake'
  task :ask => [:explain, :hello] do
    puts 'Do you know now how Rake works?'
  end

  task :all => [:hello, :explain, :ask]

  task :default => :all

  task :salute do
    puts "Hello, #{ENV.fetch('NAME', 'World')}!"
  end

  file 'checksum.md5' => 'rake.md' do
    sh 'md5sum rake.md > checksum.md5'
  end

  rule ".html" => ".md" do |t|
    sh "markdown #{t.source} > #{t.name}"
  end

  task :html => Rake::FileList.new('*.md').ext('.html')
end
