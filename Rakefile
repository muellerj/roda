require "rake"
require "rake/clean"
require "rdoc/task"

NAME = 'roda'
VERS = lambda do
  require_relative 'lib/roda/version'
  Roda::RodaVersion
end
CLEAN.include ["#{NAME}-*.gem", "rdoc", "coverage", "www/public/*.html", "www/public/rdoc", "spec/assets/app.*.css", "spec/assets/app.*.js", "spec/assets/app.*.css.gz", "spec/assets/app.*.js.gz", "spec/iv.erb"]

# Gem Packaging and Release

desc "Packages #{NAME}"
task :package=>[:clean] do |p|
  sh %{gem build #{NAME}.gemspec}
end

### RDoc

RDOC_OPTS = ["--line-numbers", "--inline-source", '--title', 'Roda: Routing tree web toolkit']

begin
  gem 'hanna-nouveau'
  RDOC_OPTS.concat(['-f', 'hanna'])
rescue Gem::LoadError
end

RDOC_OPTS.concat(['--main', 'README.rdoc'])
RDOC_FILES = %w"README.rdoc CHANGELOG MIT-LICENSE lib/**/*.rb" + Dir["doc/*.rdoc"] + Dir['doc/release_notes/*.txt']

RDoc::Task.new do |rdoc|
  rdoc.rdoc_dir = "rdoc"
  rdoc.options += RDOC_OPTS
  rdoc.rdoc_files.add RDOC_FILES
end

RDoc::Task.new(:website_rdoc) do |rdoc|
  rdoc.rdoc_dir = "www/public/rdoc"
  rdoc.options += RDOC_OPTS
  rdoc.rdoc_files.add RDOC_FILES
end

### Website

desc "Make local version of website"
task :website_base do
  sh %{#{FileUtils::RUBY} -I lib www/make_www.rb}
end

desc "Make local version of website, with rdoc"
task :website => [:website_base, :website_rdoc]

desc "Serve local version of website via rackup"
task :serve => :website do
  sh %{#{FileUtils::RUBY} -C www -S rackup}
end


### Specs

spec = proc do |env|
  env.each{|k,v| ENV[k] = v}
  sh "#{FileUtils::RUBY} #{"-w" if RUBY_VERSION >= '3'} spec/all.rb"
  env.each{|k,v| ENV.delete(k)}
  if File.directory?('.sass-cache')
    require 'fileutils'
    FileUtils.rm_r('.sass-cache')
  end
end

desc "Run specs"
task "spec" do
  spec.call({})
end

task :default=>:spec

desc "Run specs with method visibility checking"
task "spec_vis" do
  spec.call('CHECK_METHOD_VISIBILITY'=>'1')
end
  
desc "Run specs with coverage"
task "spec_cov" do
  spec.call('COVERAGE'=>'1')
end
  
### Other

desc "Print #{NAME} version"
task :version do
  puts VERS.call
end

desc "Start an IRB shell using the extension"
task :irb do
  require 'rbconfig'
  ruby = ENV['RUBY'] || File.join(RbConfig::CONFIG['bindir'], RbConfig::CONFIG['ruby_install_name'])
  irb = ENV['IRB'] || File.join(RbConfig::CONFIG['bindir'], File.basename(ruby).sub('ruby', 'irb'))
  sh %{#{irb} -I lib -r #{NAME}}
end


