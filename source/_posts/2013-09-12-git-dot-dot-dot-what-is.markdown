---
layout: post
title: "'git... what is ____?'"
date: 2013-09-12 08:51
comments: true
categories: [Reference]
---

> some git notes i put together for reference


## Rollback changes
- undo last commit, put changes to staging
{% codeblock lang:sh %}
$ git reset --soft HEAD^
{% endcodeblock %}

- new message, change the last commit
{% codeblock lang:sh %}
$ git commit --amend -m 
{% endcodeblock %}

- undo last commit and all changes
{% codeblock lang:sh %}
$ git reset --hard HEAD^ 
{% endcodeblock %}

- undo 2 commits and all changes
{% codeblock lang:sh %}
$ git reset --hard HEAD^^ 
{% endcodeblock %}

## Remotes
- add new remotes
{% codeblock lang:sh %}
$ git remote add <name> <address>
{% endcodeblock %}

- remove remotes
{% codeblock lang:sh %}
$ git remote rm <name>
{% endcodeblock %}

- push to remotes
{% codeblock lang:sh %}
$ git push -u <name> <branch>
{% endcodeblock %}

- remote show
{% codeblock lang:sh %}
$ git remote show origin
{% endcodeblock %}

- clean-up deleted remote branches
{% codeblock lang:sh %}
$ git remote prune origin    
{% endcodeblock %}


## Tagging
- list all tags
{% codeblock lang:sh %}
$ git tag
{% endcodeblock %}

- checkout code commit
{% codeblock lang:sh %}
$ git checkout v0.0.1
{% endcodeblock %}

- add new tag
{% codeblock lang:sh %}
$ git tag -a v0.0.3 -m "version 0.0.3"
{% endcodeblock %}

- push new tags
{% codeblock lang:sh %}
$ git push --tags
{% endcodeblock %}
