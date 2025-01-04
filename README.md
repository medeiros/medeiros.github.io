# Daniel's

[![TravisCI](https://travis-ci.org/medeiros/medeiros.github.io.svg?branch=master)](https://travis-ci.org/medeiros/medeiros.github.io)

Personal website, to share knowledge related to software engineering.

# Run Locally

## PreReqs

- Ruby & Jekyll
- see https://pages.github.com/versions/

## Installing RVM in Arch Linux

Please follow the steps described in https://wiki.archlinux.org/title/RVM

## Installing Ruby 2.7 using RVM

```
rvm install 2.7.4
```

## Preparing jekyll to run with Ruby 2.7.4

```
rm -rf ./vendor/bundle
rvm autolibs rvm_pkg  # so that OpenSSL dependency will be automatlically installed
rvm install 2.7.4 
rvm use 2.7.4
bundle install 
```

## Running Jekyll 
```
/bin/bash --login
./start.sh
```

