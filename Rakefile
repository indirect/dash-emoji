require 'shellwords'
require 'nokogiri'

namespace :doc do
  def docset(path = "")
    File.join(*["Emoji.docset", path.split("/")].flatten.compact)
  end

  def db
    docset('Contents/Resources/docSet.dsidx')
  end

  def docdir(path = "")
    docset(File.join('Contents/Resources/Documents', path))
  end

  def sql(*cmd)
    IO.popen("sqlite3 #{Shellwords.escape(db)}", 'w+'){|sql| cmd.each{|l| sql << l } }
  end

  def doc
    @doc ||= Nokogiri::HTML(File.read(docdir("index.html")))
  end

  task :clean do
    rm_rf docset
  end

  task :download do
    unless File.exist?("site")
      sh "git clone https://github.com/arvida/emoji-cheat-sheet.com.git site"
      src = doc.css("script[src*='jquery.min.js']").first.attributes["src"].value
      sh "wget", "http:#{src}"
      cp "jquery.min.js", docdir
    end
  end

  task :create do
    mkdir_p docset("Contents/Resources")
    cp_r "site/public", docset("Contents/Resources/Documents")
    cp "icon.png", docset
    cp "Info.plist", docset("Contents")
  end

  task :index do

    def index(name, type, path)
      sql("INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', '#{type}', '#{path}');")
    end

    # Create the database and schema
    rm_rf db if File.exist?(db)
    sql "CREATE TABLE searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);",
      "CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);"

    # Index each emoji by name, and add an anchor
    doc.css("ul.emojis li").each do |li|
      name = li.css("span.name").text
      li.set_attribute "id", name
      puts "Indexing #{name}"
      index name, "Entry", "index.html##{name}"
    end

    # Index each sound by name, with a namespaced anchor
    doc.css("ul#campfire-sounds li").each do |li|
      name = li.text.match(/\/play (.*)/){|m| m[1] }
      li.set_attribute "id", "p-#{name}"
      puts "Indexing /play #{name}"
      index name, "Event", "index.html#p-#{name}"
    end

    # Remove the GA tracking script
    doc.css("script:not([src]), script[src*=twitter]").remove

    # Rewrite the script tags to work locally
    doc.css("script[src*='jquery.min.js']").each do |s|
      s.set_attribute "src", "jquery.min.js"
    end

    File.open(docdir("index.html"), "w"){|f| f.write doc.to_html }
  end

  task :generate => [:clean, :download, :create, :index]
end

task :default => "doc:generate"
