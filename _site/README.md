This repository has the content for [microbiomehelper.ca](www.microbiomehelper.ca).

The site is built using [Jekyll](https://jekyllrb.com/docs/) with the [Chulapa](https://dieghernan.github.io/chulapa/docs) skin (there is information on the available theme options in both of these).

To make changes (including new pages), there seem to be several ways, but I recommend several steps:
1. Clone the repository onto your laptop.
2. Follow the directions to install Jekyll on your OS (e.g. [here](https://jekyllrb.com/docs/installation/macos/) for Mac, which meant installing `homebrew`, `chruby`, `ruby-install` and `jekyll`)
3. Change to the directory that your repository is in and run: `bundle install` and `bundle update github-pages`
4. Make any changes that you want to make!
5. Check them using: `bundle exec jekyll serve`
6. Confirm them using: 





This repository contains the content for the dalmug.org website.

To add a blog post you need to simply create a new file within the /_posts/ directory and it has to be named YEAR-MM-DD-Short-description.md. For example 2017-10-27-Comparative-Metagenomics-of-Blastocystis.md. The description doesn't need to be the post title - that will be specified at the top of file. This should be the date that you post the blog post and not the date the journal club was held. You should also write the title in the below section and make sure that the file ends with the ".md" extension. Also, note that italics, bold, and other formatting settings will not be carried over if you copy the text from elsewhere (but you can do that formatting in markdown as well!).

Once you make the new .md file you need to place this at the very top of the file (including the --- part):

---
published: true
title: INSERT_TEXT_HERE
post_snippet: INSERT_TEXT_HERE
---
For an example of how this would look like for an example post, see here.

Note that you can replace published:true with published:false if you want to simply save it as a draft and wait to publish it later.

The text following post_snippet is the preview of the post that will be shown on the Blog page under the post title.

Note that if you do not specify a title it will be the "Name-of-Post" part of the post file-name. It is better to specify the title yourself so that you can indicate which words should be capitalized.

Formatting is done using Markdown. This Markdown Cheatsheet might come in handy.

Lastly, instead of editing directly on github, you can use http://prose.io which provides a nicer web-based user interface for editing.