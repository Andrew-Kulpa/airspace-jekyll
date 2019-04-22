---
layout: post
title: Deploying Jekyll GitHub Pages with Travis CI
subtitle: Using Jekyll and Travis CI to autodeploy GitHub Pages
author: Andrew Kulpa
---
## Intro

GitHub Pages are a free and great way to setup and maintain a website for a personal, organizational, or project website. In addition to plain HTML content Github Pages also supports a popular ruby-based static site generator called Jekyll. There are many advantages to using Jekyll including:
  * Using Markdown instead of HTML
  * Using Jekyll themes instead of copying CSS
  * Incorporating templating for website parts like the head, headers, and footers. 
  * Simplified local development using commands such as `jekyll build` and `jekyll serve`

GitHub Pages integrates seamlessly by simplifying the build process. Unfortunately, this simplification is not without its limitations. For instance, GitHub Pages builds with the `--safe` flag, which according to Jekyll documentation disables custom plugins and ignores symbolic links. Thankfully, by using Travis CI, we can build the pages ourselves and publish to GitHub Pages.

## Develop Jekyll Site

I am assuming we have already setup a Jekyll site locally, pushed to GitHub repository, and have determined a location for our GitHub Page. Instead, this tutorial will focus upon the deployment of a Jekyll site. Deploying Jekyll sites is easy once we have the basic pipeline setup. To get to that point, there are two steps:
  * Setup the source repository with Travis CI
  * Setup deployment to destination repository & branch

### Setup Source Repository with Travis CI

The first thing that we need to do is sign-up and connect Travis CI to your GitHub account. To do this, go to travis-ci.org and sign-up with GitHub. Accept the authorization and activate it following a redirect to GitHub. Then, go to your repositories at https://travis-ci.org/account/repositories. Lastly find the source repository for your Jekyll site and flick the repository switch on.

### Setup Deployment to Destination Repo & Branch

The second step to setting up an automatic deployment pipeline for your Jekyll site is much more involved. We can break this down into a few easily chewed parts. 

#### 1. Getting Us All on the Same Page

First, we need to make sure our repository is configured correctly. If there is anything stored in the `vendor` directory, make sure to exclude it from the build process by adding the following line to your Jekyll `_config.yml` file:

```yaml
exclude: [vendor]
```

Next, we need to determine where exactly the Jekyll site should be deployed to. If this is a GitHub Page for a project, the destination by default would be the `gh-pages` branch but could also be located at the `mastery` branch or a folder named `docs`. For user and organization GitHub Pages, 

#### Create deployment `.travis.yml`

```yaml
language: ruby
rvm:
  - 2.5.1

before_script:

# Assume bundler is being used, therefore
# the `install` step will run `bundle install` by default.
script: 
 - set -e # halt script on error
 - bundle exec jekyll build
 - bundle exec htmlproofer --http-status-ignore "999" ./_site

# branch whitelist, only for GitHub Pages
branches:
  only:
  - master

env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer

addons:
  apt:
    packages:
    - libcurl4-openssl-dev

sudo: false # route your build to the container-based infrastructure for a faster build

cache: bundler # caching bundler gem packages will speed up build

# Optional: disable email notifications about the outcome of your builds
notifications:
  email: false
```
### Create repository for user / organization page
### Setup overridden deployment location (user/org) OR just use base gh-pages setup (local)

