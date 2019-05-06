---
layout: post
title: Deploying Jekyll GitHub Pages with Travis CI
subtitle: Using Jekyll and Travis CI to autodeploy GitHub Pages
author: Andrew Kulpa
excerpt: GitHub Pages are a free and great way to setup and maintain a website for a personal, organizational, or project website. Thankfully, by using Travis CI, we can build the pages ourselves and publish to GitHub Pages.
---

## Overview

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

Next, we need to determine where exactly the Jekyll site should be deployed to. With the `pages` provider for the deployment process of the Travic CI configuration, we can define this quite easily. If this is a GitHub Page for a project, the destination by default would be the `gh-pages` branch. For user and organization GitHub Pages, the resulting site would need to be located at the `master`. Its worth noting that normal project pages also support a third option: a `docs` folder. 

#### Create deployment `.travis.yml`

Now that we've determined where the site is to be deployed to, we can finally make the Travis CI `.travis.yml` configuration file. First, create a blank `.travis.yml` file. Then, paste the following configuration:

```yaml
language: ruby
rvm:
  - 2.5.1
cache: bundler # caching bundler gem packages will speed up build
before_script:
script: 
 - set -e
 - bundle exec jekyll build
 - bundle exec htmlproofer --http-status-ignore "999" ./_site # test site, but ignore any 999 errors (e.g. LinkedIn checks)
branches: # branch whitelist, only for GitHub Pages
  only:
  - master
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # faster html-proofer install
addons:
  apt:
    packages:
    - libcurl4-openssl-dev
sudo: false 
notifications: # Optionally disable build email notifications 
  email: false
deploy:
  provider: pages
  local-dir: ./_site
  skip_cleanup: true
  name: Deployment Bot
  github_token: $GITHUB_TOKEN  # Set in the settings page of your repository, as a secure variable
  repo: $REPOSITORY # Set in the settings page of your repository, as a secure variable
  keep_history: true
  target-branch: master
  on:
    branch: master
```

This configuration does quite a bit, so I'll break it down. 

```yaml
language: ruby
rvm:
  - 2.5.1
cache: bundler # caching bundler gem packages will speed up build
```

This tells Travis CI to use Ruby and Bundler during the build process.

```yaml
before_script:
script: 
 - set -e
 - bundle exec jekyll build
 - bundle exec htmlproofer --http-status-ignore "999" ./_site # test site, but ignore any 999 errors (e.g. LinkedIn checks)
```

We want to halt the build upon error, in case later script parts fail to process as expected. After this, we can build the site and use `htmlproofer` to test the rendered HTMLfiles to sure the quality of the site.

```yaml
branches: # branch whitelist, only for GitHub Pages
  only:
  - master
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # faster html-proofer install
```

We only want to automatically generate site content from the `master` branch and speed up the `html-proofer` installation.

```yaml
addons:
  apt:
    packages:
    - libcurl4-openssl-dev
notifications: # Optionally disable build email notifications 
  email: false
```

We then make sure that the development files for the OpenSSL flavor of libcurl are available. Optionally, we also disable the email notifications for the build process.

```yaml
deploy:
  provider: pages
  local-dir: ./_site
  skip_cleanup: true
  name: Deployment Bot
  github_token: $GITHUB_TOKEN  # Set in the settings page of your repository, as a secure variable
  repo: $REPOSITORY # Set in the settings page of your repository, as a secure variable
  keep_history: true
  target-branch: master
  on:
    branch: master
```

In this last part of the `.travis.yml` we use the [Github Pages Deployment Provider](https://docs.travis-ci.com/user/deployment/pages/). The site by default is built to the `./_site` local directory. A [personal access token](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line) is needed at this point too. Once you have created your personal access token and determined the target repository, this information can easily be defined using secure variables in Travis CI. The last few lines determine the source and target branches. As the example repository is a master to master deployment from two different repositories, the configuration is as demonstrated, but your configuration may differ.

#### Update Gemfile

Finally, after we have completed the Travis CI configuration we can add the gem dependency for `html-proofer` to the `Gemfile`.

```ruby
gem "html-proofer"
```

## Wrapping Up

In this tutorial we walked through setting up the source repository for your Jekyll site and deploying it to a target repository. While GitHub Pages supports basic deployments, the `--safe` building of Jekyll sites can be rather limiting. Through switching to a Travis CI based deployment process, you may further improve your site without limitation through the newfound freedom of using Jekyll Plugins.