---
layout: post
title: "how i start my controller spec"
date: 2013-10-20 08:04
comments: true
categories: [Rails]
---
first, setup the data e.g.

{% codeblock lang:ruby %}
let(:user) { FactoryGirl.create(:user) }
let(:admin) { FactoryGirl.create(:admin) }
{% endcodeblock %}

then define who has access to it e.g.

{% codeblock lang:ruby %}
describe "admin access" do
end

describe "user access" do
end

describe "guest access" do
end
{% endcodeblock %}

if needed, i use **shared examples** e.g.

{% codeblock lang:ruby %}
shared_examples("public access to products") do
end

shared_examples("full access to products") do
end
{% endcodeblock %}

and call them like this:

{% codeblock lang:ruby %}
it_behaves_like "public access to products"
{% endcodeblock %}

here's the full source for reference:

{% codeblock lang:ruby %}
require 'spec_helper'

describe ProductsController do

  let(:user) { FactoryGirl.create(:user) }
  let(:admin) { FactoryGirl.create(:admin) }

  shared_examples("public access to products") do
    describe "GET #index" do
      before :each do
        get :index
      end

      it "renders the :index template"
    end
  end

  shared_examples("full access to products") do
    describe "GET 'new'" do
      before :each do
        get :new
      end

      it "renders the :new template"
    end
  end

  describe "admin access" do
    before :each do
      sign_in :user, admin
    end

    it_behaves_like "public access to products"
    it_behaves_like "full access to products"
  end

  describe "user access" do
    before :each do
      sign_in :user, user
    end

    it_behaves_like "public access to products"
    it_behaves_like "full access to products"
  end

  describe "guest access" do
    it_behaves_like "public access to products"

    describe "GET 'new'" do
      it "requires login"
    end
  end
end
{% endcodeblock %}
