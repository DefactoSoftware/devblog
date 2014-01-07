defactosoftware.github.io
=========================

The Defacto Engineering blog.

Dependencies to run locally:
- Ruby
- Jekyll (`gem install jekyll`)

To run locally: `jekyll serve`, add the -w flag to autocompile changes.

It's built with [Jekyll][jekyll], to add a post:
- `git clone git@github.com:DefactoSoftware/DefactoSoftware.Github.io.git`
- `git checkout -b my-awesome-blogpost`
- add your info to the `authors` in the [_config.yml](_config.yml) file if it's
not there yet.
- create a new post with
`rake post author="your name" title="My Awesome Blogpost"`, this will
create a markdown file in the `_posts` directory.
- write your post in [markdown][markdown]
- `git push origin my-awesome-blogpost` and open a pull request

[jekyll]:    http://jekyllrb.com
[markdown]:    http://daringfireball.net/projects/markdown/
