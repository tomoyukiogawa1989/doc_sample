# https://github.com/rubyzip/rubyzip
require './example_recursive.rb'
require 'fileutils'
require 'rake'
require 'rubygems'
require 'zip'
require "nkf"
require "tempfile"
 
desc 'pdf生成'
task :pdf do
  Dir.glob(['**/index.adoc', '**/README.adoc']).each do |file|
    dist_path = "dist/#{File.dirname(file)}"
    basename = File.basename(file, '.*')
    FileUtils.mkdir_p(dist_path) unless File.exist?(dist_path)
    sh %W[
      asciidoctor-pdf
      -r asciidoctor-diagram
      -r asciidoctor-pdf-cjk
      #{file}
      -o #{dist_path}/#{basename}.pdf
    ].join(" ")
  end
end
 
desc 'html生成'
task :html do
  Dir.glob(['**/index.adoc', '**/README.adoc', '**/*.md']).each do |file|
    dist_path = "dist/#{File.dirname(file)}".encode('utf-8')
    basename = File.basename(file, '.*')
    case File.extname(file)
    when '.adoc'
      FileUtils.mkdir_p(dist_path) unless File.exist?(dist_path)
      sh "asciidoctor -r asciidoctor-diagram -D #{dist_path} -b html #{file}"
      if File.exist?("#{File.dirname(file)}/images")
        FileUtils.mkpath "#{dist_path}/images"
        FileUtils.cp_r "#{File.dirname(file)}/images/.", "#{dist_path}/images", verbose: true
      end
    when '.md'
      FileUtils.mkdir_p(dist_path) unless File.exist?(dist_path)
      sh %W[
        pandoc
        -f markdown
        -t html5
        --template=md2html.html
        #{file}
        -o #{dist_path}/#{basename}.html
      ].join(" ")
      if File.exist?("#{File.dirname(file)}/images")
        FileUtils.mkpath "#{dist_path}/images"
        FileUtils.cp_r "#{File.dirname(file)}/images/.", "#{dist_path}/images", verbose: true
      end
    end
  end
end
 
desc 'S3へアップロード'
task upload: [:pdf, :html] do
  zf = ZipFileGenerator.new('dist', 'dist.zip')
  zf.write()
  sh 'aws --debug --profile s3 cp dist.zip s3://buckup/document.zip'
end
 
desc '半角カタカナを全角カタカナに変換して保存する'
task :convert_hankana_to_zenkana do
  Dir.glob(['**/*.adoc']).each do |file|
    tempfile = Tempfile.open(File.basename(file))
    File.foreach(file) do |line|
      tempfile.puts NKF.nkf("-Xw", line)
    end
    tempfile.close
    FileUtils.mv tempfile.path, file
  end
end
 
