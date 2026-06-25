This repository has the content for [microbiomehelper.ca](www.microbiomehelper.ca).

The site is built using [Jekyll](https://jekyllrb.com/docs/) with the [Chulapa](https://dieghernan.github.io/chulapa/docs) skin (there is information on the available theme options in both of these).

To make changes (including new pages), there seem to be several ways, but I recommend several steps:
1. Clone the repository onto your laptop.
2. Follow the directions to install Jekyll on your OS (e.g. [here](https://jekyllrb.com/docs/installation/macos/) for Mac, which meant installing `homebrew`, `chruby`, `ruby-install` and `jekyll`)
3. Change to the directory that your repository is in and run: `bundle install` and `bundle update github-pages`
4. Make any changes that you want to make!
5. Check them using: `bundle exec jekyll serve`
6. When you are ready, build the site using: `bundle exec jekyll build`
7. And push the changes to Github