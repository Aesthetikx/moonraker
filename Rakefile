require 'date'
require 'haml'
require 'sass-embedded'
require 'webrick'

S3_BUCKET = 'bucket_name'.freeze
CLOUDFRONT_DISTRIBUTION = 'distribution_name'.freeze
REGION = 'us-east-1'.freeze

pages = %w[index about]

task :markup do
  print 'Generating Markup...'

  FileUtils.mkdir_p('./public')

  pages.each do |page|
    File.open("./public/#{page}.html", 'w') do |file|
      haml = Haml::Template.new('./layout.haml', escape_html: false).render do
        Haml::Template.new("./#{page}.haml").render
      end

      file.puts haml
    end
  end

  puts 'done.'
end

task :style do
  print 'Generating Stylesheet...'

  FileUtils.mkdir_p('./public')

  File.open('./public/style.css', 'w') do |file|
    file.puts Sass.compile('./style.sass', style: :compressed).css
  end

  puts 'done.'
end

task build: %i[style markup]

task sync: :build do
  print 'Syncing to s3...'
  `aws s3 --region #{REGION} sync public/ s3://#{S3_BUCKET}/`
  puts 'done.'
end

task invalidate: :sync do
  paths = pages.map { |p| "\"/#{p}.html\"" }.join(' ') + ' "/style.css"'
  print "Invalidating #{paths}..."
  `aws cloudfront create-invalidation --distribution #{CLOUDFRONT_DISTRIBUTION} --paths #{paths}`
  puts 'done.'
end

task serve: :build do
  root = File.expand_path './public'

  server = WEBrick::HTTPServer.new Port: 4567, DocumentRoot: root

  trap('INT') { server.shutdown }

  server.start
end

task upload: :invalidate

task default: :build
