require 'rubygems'
require 'nokogiri'
require 'fileutils'
require 'thread'

NUM_THREADS = 16

subheader_link = "http://lofic.github.io/tips/[lofic.github.io/tips]"
project_link = "https://github.com/lofic/tips/[> give me a star here <]"

html_folder = "./html" # Where you generate your html files
txt_folder = "./txt" # Where you write your notes

desc "Default task : build html and index"
task :default do
    Rake::Task[:genhtml].invoke
    sleep 0.2
    Rake::Task[:remrev].invoke
    sleep 0.2
    Rake::Task[:genindex].invoke
end

desc "Generate HTML files from .txt asciidoc files"
task :genhtml, [:theme, :backend] do |t, args|
    args.with_defaults(:theme => "pryz", :backend => "lofic_backend")
    puts "Building notes with the theme : #{args.theme} and the backend : lofic"

    if not File.directory? html_folder
        puts "mkdir html"
        Dir.mkdir html_folder
    end
    FileUtils.rm(Dir.glob("#{html_folder}/*.txt"))
    FileUtils.cp(Dir.glob("#{txt_folder}/*.txt"),html_folder)
    txtfilelist=Dir.glob("#{html_folder}/*.txt")

    if txtfilelist.empty?
       puts "No .txt files... exiting"
       exit
    end

    # A queue is thread safe
    queue = txtfilelist.inject(Queue.new, :push)

    #Until queue.empty?
    #    file = queue.shift.chomp
    #    %x[asciidoc  --backend=#{args.backend} --theme=#{args.theme} #{file}]
    #End

    threads = Array.new(NUM_THREADS) do
        Thread.new do
            until queue.empty?
                file = queue.shift.chomp
                %x[asciidoc  --backend=#{args.backend} --theme=#{args.theme} #{file}]
            end
        end
    end

    begin
        threads.each(&:join)
    ensure
        FileUtils.rm(Dir.glob("#{html_folder}/*.txt"))
    end

end

desc "No rev"
task :remrev do
    # Delete rev and build date
    `sed -i '/^Last updated/d' #{html_folder}/*.html`
    `sed -i -e '/<div id="footer-text">/{n;d}' #{html_folder}/*.html`
end

desc "Generate one file"
task :genfile, [:file, :theme, :backend] do |t, args|
    args.with_defaults(:theme => "pryz", :backend => "lofic")
    if File.exists?(args.file)
        puts "Building html for : #{args.file}"
        %x[asciidoc -o #{html_folder}/#{args.file.delete(".txt")}.html --theme=#{args.theme} --backend=#{args.backend} #{args.file}]
    else
        puts "File #{args.file} doesn't exist"
    end
end

desc "Generate Index file"
task :genindex, [:theme, :backend] do |t, args|
    args.with_defaults(:theme => "pryz", :backend => "index_list")
    puts "Building index with the theme : #{args.theme} and the backend : #{args.backend}"
    index_file = "index.txt"
    FileUtils.cd(html_folder)
    index = File.open(index_file, 'w+')
    head_content = <<EOT
Notes
=====

link:#{subheader_link}

If you like these tips,
*link:#{project_link}*
to help other people know about it !

List
----

EOT
    index.write(head_content)
    txtfilelist=Dir.glob("*.html")
    until txtfilelist.empty?
        note_file = txtfilelist.pop
        page = Nokogiri::HTML(open(note_file))
        title = page.css('title').text
        tags = ""
        page.xpath('//div[@class="paragraph"]').each do | div |
            tags = div.content if div.content.match(/Additional tag/)
        end
        if tags.empty?
            index.write("* link:#{note_file}[#{title}]\n")
        else
            index.write("* link:#{note_file}[#{title}] -h-#{tags}-h-\n")
        end
    end
    index.close
    %x[asciidoc --theme=#{args.theme} --backend=#{args.backend} #{index_file}]
    File.delete(index_file)
end
