---
layout: post
title: "Continuously Deploying Bitkumo.com with Jekyll and CircleCI"
category: blog
tags:
  - bitkumo
  - devops
  - jekyll
  - circleci
permalink: /blog/:title/
author: levlaz
---

Our main website, [Bitkumo.com](https://bitkumo.com), is built using Jekyll. If you have not heard of [Jekyll](https://jekyllrb.com/), it is an awesome static site generator that was written by one of the [co-founders of GitHub](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html), and is the same software that powers [GitHub pages](https://pages.github.com/). Jekyll makes it really easy to create simple websites, serve static content, and publish a blog without the need for a CMS. We wanted to share how we continuously deploy our site using Jekyll and [CircleCI](https://circeci.com) to our server so that it may help others who are looking to do the same. 

## Why Not Just Use GitHub Pages?

GitHub Pages is perfect for many different purposes, but since it runs in somewhat of a sandbox, you are not able to use any custom extensions or plugins. One thing that we use right now is a [tags plugin](https://github.com/bitkumo/bitkumo-website/blob/master/_plugins/_tag_gen.rb) which we would not be able to use on GitHub Pages. As we expand the site we may use other plugins as well. In short, if you need more flexibility you may be better off deploying to your own server instead of letting GitHub Pages serve your Jekyll site. 

## Workflow Overview

1. Create and edit new site content and blog posts in a local branch. 
2. Push the branch to GitHub and let CircleCI do some tests with [html-proofer](https://github.com/gjtorikian/html-proofer).
3. If tests pass, merge to the master branch. 
4. CircleCI builds our Jekyll site and deploys it to our VPS Server. 

[Getting started with Jekyll](http://jekyllrb.com/docs/quickstart/) is simple, assuming you have followed the steps outlined in the documentation you are ready to begin deploying your site. If you have not already, [sign up for CircleCI](https://circleci.com/docs/getting-started) and follow the GitHub project where you are storing your Jekyll site. 

## Preparing your Server and CircleCI for deployment.  

In order to let CircleCI deploy your Jekyll site to your server you will need to create a new user and a deployment key and add it to CircleCI. Log into your server and create new user: 

```
sudo adduser circleci 
```

Log in as this user and create a new ssh key. Be sure to not create a key passphrase since you will not be able to enter the password during the deployment phase. 

```
su - circleci 
ssh-keygen 
cat .ssh/id_rsa 
```

Copy the private key and paste it into the CircleCI form under *Project Settings* -> *SSH Permissions* (Or the URL in the form of https://circleci.com/gh/$ORG/$REPO/edit#ssh)

<img src="circlci_img" alt="CircleCI Key Entry Form"></img>

Back on your server, add the public key to `/home/circleci/.ssh/authorized_keys` to allow the CircleCI builder to connect to your server. 

```
cp /home/circleci/.ssh/id_rsa.pub /home/circleci/.ssh/authorized_keys 
```

## Configuring Jekyll to Build on CircleCI 

To make it easier to build the project on CircleCI (any any other server) we create a Gemfile and Rakefile to manage our Ruby Gems and define what needs to be done during a build. Our Gem file is shown below and this file specifies all of the Gems that we want to be installed during a build. 

```
source 'https://rubygems.org'
ruby '2.2.1'

group :jekyll_plugins do
  gem "rouge"
  gem "jekyll"
  gem "jekyll-gist"
  gem "jekyll-coffeescript"
  gem "html-proofer"
end
```

Our Rakefile is shown below and specifies what to do when we run tests. 

```
require 'html/proofer'

task :test do
    sh "bundle exec jekyll build"
    HTML::Proofer.new("./_site", {
        :href_ignore => [
            "#"
        ],
        :url_ignore => [
            /linkedin\.com/
        ],
        :disable_external => true
        }).run
end
```

We are using [html-proofer](https://github.com/gjtorikian/html-proofer) to run some basic test on our jekyll site. These tests check if your image references are legitimate, if they have alt tags, if your internal links are working, and so on. One caveat is that links to linkedin.com seem to always be broken so we ignore then during our build. 

## Configuring circle.yml 

CircleCI uses a yml file to specify various configurations for what to do during a build, as well as specifying the deployment phase. Our circle.yml file is shown below: 

```
test:
    override:
        - bundle exec rake test

deployment:
  push_to_server:
    branch: master
    commands:
      - rsync -avz _site/ circleci@bitkumo.com:/var/www/bitkumo
```

Since we have a Gem file, CircleCI will automatically run `bundle install` at the beginning of the build to install all of our dependencies. In the circle.yml file we tell CircleCI to run `bundle exec rake test` to run the html-proofer tests. If this is for a master branch, and the tests succeed, then we deploy to our server by rsyncing the build output located in the `_site` directory to our server. 

We have been pretty happy with this workflow so far, and it helps us deploy with confidence whenever we make changes to our main site. All of the code for our main site is [open source in GitHub](https://github.com/bitkumo/bitkumo-website) so you can try it for yourself. If you have any questions about deploying Jekyll sites to a VPS server using CircleCI please let us know in the comments below! 

Looking for 0% nonsense cloud hosting? [Get started in the Bitkumo Cloud Today!](https://app.bitkumo.com/auth/register)

