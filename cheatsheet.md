### Important commands

Run locally:
```
bundle exec jekyll serve --livereload
```

Build:
```
bundle exec jekyll build
```

### Add plugin
To add `jekyll-myplugin`, the `Gemfile` should look like this:
```
source 'https://rubygems.org'

gem "jekyll"

group :jekyll_plugins do
  gem "jekyll-myplugin"
end
```
In `_config.yaml` add the following line under plugins:
```
plugins:
    - jekyll-myplugin
```

More on plugins [here](https://jekyllrb.com/docs/step-by-step/10-deployment/#plugins).