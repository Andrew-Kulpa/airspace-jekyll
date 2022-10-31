# Andrew Kulpa - Personal Site
My personal site using the Airspace Jekyll theme.

## Prerequisites
1. [Ruby](https://www.ruby-lang.org/en/)
2. [Bundler](https://bundler.io)
3. Docker (optional)
## Steps for Setup:

### Clone the repository, and `cd` into it:
```sh
$ git clone https://github.com/ndrewtl/airspace-jekyll.git
$ cd airspace-jekyll
```

### Run the server from Docker:
```sh
$ bundle exec jekyll server
```
Alt:
```sh
$ docker run --rm --volume="$PWD:/srv/jekyll" \
            -p 4000:4000 -it \
            jekyll/jekyll \
            jekyll serve --watch --drafts
```

## Deployment
From any given branch, push to the `gh-pages` branch to trigger an automated deployment.