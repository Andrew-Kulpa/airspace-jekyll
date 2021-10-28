# Andrew Kulpa - Personal Site
My personal site using the Airspace Jekyll theme.

## Prerequisites
1. [Ruby](https://www.ruby-lang.org/en/)
2. [Bundler](https://bundler.io)
## Steps for Setup:

### Clone the repository, and `cd` into it:
```sh
$ git clone https://github.com/ndrewtl/airspace-jekyll.git
$ cd airspace-jekyll
```

### Install dependencies locally:
```sh
$ bundle install --path vendor/bundle
```

This should install a local copy of jekyll.

### Now run the server:
```sh
$ bundle exec jekyll server
```

## Deployment
From any given branch, push to the `gh-pages` branch to trigger an automated deployment.