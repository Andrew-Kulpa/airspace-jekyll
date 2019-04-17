---
layout: post
title: Setting Up Your GitHub Personal Page With Jekyll and Travis CI
subtitle: Using Jekyll and Travis CI to autodeploy GitHub Pages
author: Andrew Kulpa
---
## Intro
### Github Pages
### Jekyll
### Travis CI

## Develop Jekyll Site

Jekyll is a Ruby program that generates static sites. The primary commands are `jekyll build` which outputs the static site to `_site` and `jekyll serve` which rebuilds upon changes and runs a local web server at `http://localhost:4000`.

For
## Setup Source Repository with Travis CI
1. Sign-up and connect Travis CI to your GitHub account
2. Go to your repositories at https://travis-ci.org/account/repositories
3. Find the source repository for your Jekyll site
4. Flick the repository switch on
## Setup Deployment to Destination Repo & Branch
### Exclude vendor in Jekyll `_config.yml`

Add the following line:

```yaml
exclude: [vendor]
```

### Create deployment `.travis.yml`

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
## Create repository for user / organization page
## Setup overridden deployment location (user/org) OR just use base gh-pages setup (local)

