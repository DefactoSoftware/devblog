require 'yaml'

desc "create a new post"
task :post do
  author_name = ENV["author"]
  title = ENV["title"]

  begin
    YAML.load_file("_config.yml")["authors"][author_name]
  rescue
    raise "no such author found, please add your info to the _config.yml file"
  end

  slug = "#{Date.today}-#{title.downcase.gsub(/[^\w]+/, '-')}"

  file = File.join(
    File.dirname(__FILE__),
    '_posts',
    slug + '.markdown'
  )

  File.open(file, "w") do |f|
    f.write(
%Q(---
layout: post
title: "#{title}"
date: #{Time.now.strftime("%Y-%m-%d %H:%M:%S")}
author: #{author_name}
categories: []
---

Write your post in markdown here!
)
    )
  end
end
