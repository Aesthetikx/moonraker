require 'date'
require 'haml'
require 'sassc'

S3_BUCKET = 'bucket_name'.freeze
CLOUDFRONT_DISTRIBUTION = 'distribution_name'.freeze
REGION = 'us-east-1'.freeze

pages = %w[index about]

task :markup do
  print 'Generating Markup...'

  FileUtils.mkdir_p('./public')

  pages.each do |page|
    File.open("./public/#{page}.html", 'w') do |file|
      haml = Haml::Engine.new(File.read('./layout.haml')).render do
        Haml::Engine.new(File.read("./#{page}.haml")).render
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
    sass = File.read('./style.scss')

    file.puts SassC::Engine.new(sass, style: :compressed).render
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

task upload: :invalidate

task default: :build
