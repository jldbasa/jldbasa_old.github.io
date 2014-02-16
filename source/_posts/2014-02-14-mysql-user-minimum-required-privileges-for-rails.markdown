---
layout: post
title: "mysql user minimum required privileges for rails"
date: 2014-02-14 09:51
comments: true
categories: [Rails, Good Practice]
---

if your application uses `root` as your database username; you're doing it wrong.
  a good practice is you should have two users for your application. let say
  i have a database called `blog_production`, i will have two users in my
  database: `blog` and `blog_admin`. the former will be use for regular stuff,
  and the latter for database administration. each db user have diffrent set of privileges. 

* `blog` user privileges
  - Select
  - Insert
  - Update
  - Delete
  - Lock
* `blog_admin` user privileges
  - Select
  - Insert
  - Update
  - Delete
  - Lock
  - Create
  - Drop
  - Index
  - Alter

{% codeblock lang:mysql %}
CREATE DATABASE blog_production;

CREATE USER 'blog'@'localhost' IDENTIFIED BY 'db_password';
GRANT Select,Insert,Update,Delete,Lock Tables ON blog_production.* TO 'blog'@'localhost';

CREATE USER 'blog_admin'@'localhost' IDENTIFIED BY 'db_password';
GRANT Select,Insert,Update,Delete,Create,Drop,Index,Alter,Lock Tables ON blog_production.* TO 'blog_admin'@'localhost';

FLUSH PRIVILEGES;
{% endcodeblock %}

now, how do you use this two users in your rails app? in `databases.yml`, add a check for `DB_ADMIN` env var.

{% codeblock lang:ruby config/databases.yml %}
<%
  if ENV['DB_ADMIN']
    username = "blog_admin"
    password = "11111111"
  else
    username = "blog"
    password = "22222222"
  end
%>
production:
  adapter: mysql2
  encoding: utf8
  reconnect: false
  database: blog_production
  pool: 5
  username: <%= username %>
  password: <%= password %>
{% endcodeblock %}

by default it uses `blog` db username unless you pass `DB_ADMIN=true` env var.
the reasone we have this env is because we need to pass it when we run
migration  tasks e.g:

{% codeblock lang:bash %}
$ DB_ADMIN=true bundle exec rails db:migrate
{% endcodeblock %}

if you use **capistrano** for deployment, you need to set `migrate_env` var.

{% codeblock lang:ruby config/deploy.rb %}
set :migrate_env, "DB_ADMIN=true"
{% endcodeblock %}

if you have better method, hit the comments. 

and that's all there is to it. hth.