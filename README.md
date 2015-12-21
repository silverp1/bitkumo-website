# bitkumo-website
Main [bitkumo website](https://bitkumo.com)

# Hacking
This site is generated using [jekyll](https://jekyllrb.com/). Hacking on this site is really easy.

1. Clone the repo
2. `cd` into bitkumo-website
3. Install all dependencies that are defined in the `Gemfile`
    `bundle install`
4. Serve up jekyll locally. This will allow you to make changes and automatically see them at localhost:4000
    `jekyll serve`
5. We are using [html-proofer](https://github.com/gjtorikian/html-proofer) to do some basic testing, you can run tests with:
    `bundle exec rake test`

# License
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png" /></a><br />All blog content and documentation (located in _drafts/, blog/, and docs/) is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

All other code is MIT Licensed.
