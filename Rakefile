# Originally adopted from Raimonds Simanovskis Rakefile, 
#   http://github.com/rsim/blog.rayapps.com/blob/master/Rakefile
#   which was in turn adopted from Tate Johnson's Rakefile
#   http://github.com/tatey/tatey.com/blob/master/Rakefile


require 'webrick'
require 'directory_watcher'
require "term/ansicolor"
require "jekyll"
require "rdiscount"
require "yaml"
include Term::ANSIColor
include WEBrick

host_var = YAML::load(File.open('deploy.yml'))


task :default => :develop
 
desc 'Build site with Jekyll.'
task :build => :tags do
	printHeader "Compiling website..."
	options = Jekyll.configuration({})
	@site = Jekyll::Site.new(options)
	@site.process
end
 
def globs(source)
	Dir.chdir(source) do
		dirs = Dir['*'].select { |x| File.directory?(x) }
		dirs -= ['_site']
		dirs = dirs.map { |x| "#{x}/**/*" }
		dirs += ['*']
	end
end

desc 'Enter development mode.'
task :develop => :build do
	printHeader "Auto-regenerating enabled."
	directoryWatcher = DirectoryWatcher.new("./")
	directoryWatcher.interval = 1
	directoryWatcher.glob = globs(Dir.pwd)
	directoryWatcher.add_observer do |*args| @site.process end
	directoryWatcher.start
	mimeTypes = WEBrick::HTTPUtils::DefaultMimeTypes
	mimeTypes.store 'js', 'application/javascript'
	server = HTTPServer.new(
		:BindAddress	=> "localhost",
		:Port			=> 4000,
		:DocumentRoot	=> "_site",
		:MimeTypes		=> mimeTypes,
		:Logger			=> Log.new($stderr, Log::ERROR),
		:AccessLog		=> [["/dev/null", AccessLog::COMBINED_LOG_FORMAT ]]
	)
	thread = Thread.new { server.start }
	trap("INT") { server.shutdown }
	printHeader "Development server started at http://localhost:4000/"
	printHeader "Opening website in default web browser..."
	%x[open http://localhost:4000/]
	printHeader "Development mode entered."
	thread.join()
end

desc 'Remove all built files.'
task :clean do
	printHeader "Cleaning build directory..."
	%x[rm -rf _site]
end

desc 'Build, deploy, then clean.'
task :deploy => :build do
  domain = "tesoriere.com"
	printHeader "Deploying website to #{domain}"
	sh "rsync -rtzh _site/ #{host_var['DEPLOY_USER']}@#{host_var['DEPLOY_HOST']}:~/#{domain}/"
	Rake::Task['clean'].execute
end

task :new do
	title = ask("Title: ")
	article = {"title" => title, "layout" => "post"}.to_yaml
	article << "---"
	fileName = title.gsub(/[\s \( \) \? \[ \] \, \: \< \>]/, '-').downcase
	path = "_posts/#{Time.now.strftime("%Y-%m-%d")}#{'-' + fileName}.markdown"
	unless File.exist?(path)
		File.open(path, "w") do |file| 
			file.write article
			sh "mate " + path
		end
    	puts "A new article was created at #{path}."
	else
    	puts "There was an error creating the article, #{path} already exists."
	end
end

def ask message
	print message
	STDIN.gets.chomp
end

def printHeader headerText
	print bold + blue + "==> " + reset
	print bold + headerText + reset + "\n"
end

desc 'Generate tags pages'
task :tags  => :tag_cloud do
  puts "Generating tags..."
  require 'rubygems'
  require 'jekyll'
  include Jekyll::Filters
  
  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')

  # Remove tags directory before regenerating
  FileUtils.rm_rf("tags")

  site.tags.sort.each do |tag, posts|
    html = <<-HTML
---
layout: default
title: "tagged: #{tag}"
syntax-highlighting: yes
---
  <h1 class="title">#{tag}</h1>
  {% for post in site.posts %}
		{% for tag in post.tags %}
			{%if tag == "#{tag}" %}
				{%include post.html%}
			{%endif%}
		{%endfor%}
  {% endfor %}
HTML

    FileUtils.mkdir_p("tags/#{tag}")
    File.open("tags/#{tag}/index.html", 'w+') do |file|
      file.puts html
    end
  end
  puts 'Done.'
end

desc 'Generate tags pages'
task :tag_cloud do
  puts 'Generating tag cloud...'
  require 'rubygems'
  require 'jekyll'
  include Jekyll::Filters

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')

  html = ''
  max_count = site.tags.map{|t,p| p.count}.max
  site.tags.sort.each do |tag, posts|
    s = posts.count
    font_size = ((20 - 10.0*(max_count-s)/max_count)*2).to_i/2.0
    html << "<a href=\"/tags/#{tag.gsub(/ /,"%20")}\" title=\"Postings tagged #{tag}\" style=\"font-size: #{font_size}px; line-height:#{font_size}px\">#{tag}</a> "
  end
  File.open('_includes/tag_cloud.html', 'w+') do |file|
    file.puts html
  end
  puts 'Done.'
end

