---
layout: post
title: "creating rails app from scratch"
date: 2013-10-10 10:47
comments: true
categories: [Rails]
---
this is a trick i always use when i create a new project and i don't have rails gem
installed globally. but before i put the commands, here's my current setup:

{% codeblock lang:sh %}
$ gem list

*** LOCAL GEMS ***
bundler (1.3.5)
rbenv-gem-rehash (1.0.0)
{% endcodeblock %}


also, i install the gems in a vendor directory inside the project; which means you
need to specify the `--path vendor` when running bundle install.

here are the commands:

{% codeblock lang:sh %}
$ mkdir appname
$ cd appname
$ echo "source 'https://rubygems.org'" > Gemfile
$ echo "gem 'rails', '~> 4.0.0'" >> Gemfile

# make sure that your ruby version is set before running the next commands. am
# using version 2.0.0.

$ bundle install --path vendor
$ bundle exec rails new . --skip-bundle -d mysql
$ bundle install --path vendor
$ rails -v
$ bundle package
$ echo 'vendor/ruby' >> .gitignore
{% endcodeblock %}
