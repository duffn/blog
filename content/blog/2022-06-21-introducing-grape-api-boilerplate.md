---
title: "Introducing Grape API Boilerplate"
date: 2022-06-21T00:00:00+00:00
permalink: /introducing-grape-api-boilerplate/
description: "A full-featured, production ready, and easy to understand API boilerplate for the Grape framework."
tags: [ruby, grape]
---

For reasons not quite known to me, I've always wanted to learn more about [Ruby](https://www.ruby-lang.org/en/). While I'm a big Python fan, and that certainly covers my interpreted language needs, Ruby has always been in the back of my mind as something that I wanted to know.

A few weeks ago I started playing around with all of the Slack related repos in the [`slack-ruby` GitHub organization](https://github.com/slack-ruby?type=source). I was interested in making a Slack bot and learning Ruby and this turned out to be a great place to do both.

Fast forward a bit and I got very interested in the [Grape framework](https://www.ruby-grape.org/) on which all of the bots in the organization are based upon. I opened up a few PRs in [`ruby-grape`](https://github.com/ruby-grape?type=source) and was off to the races.

There are a few basic Grape boilerplates, but I wanted to add a more full-featured one to the mix so, created [`grape-api-boilerplate`](https://github.com/duffn/grape-api-boilerplate).

The [boilerplate](https://github.com/duffn/grape-api-boilerplate) already has many features with more planned.

- Local development with [Docker](https://www.docker.com/) and Docker Compose.
- Automatic Puma reloading locally with [`guard-puma`](https://github.com/jc00ke/guard-puma).
- ActiveRecord with [`otr-activerecord`](https://github.com/jhollinger/otr-activerecord).
- Swagger API documentation with [`grape-swagger`](https://github.com/ruby-grape/grape-swagger).
- User authentication with [`bcrypt`](https://github.com/bcrypt-ruby/bcrypt-ruby)
  using [`jwt`](https://github.com/jwt/ruby-jwt).
- Model pagination with [`api-pagination`](https://github.com/davidcelis/api-pagination).
- Comprehensive [RSpec](https://rspec.info/) test suite and code coverage.
- Easy [Heroku](https://www.heroku.com/) deployment.

[Contributions](https://github.com/duffn/grape-api-boilerplate/blob/main/CONTRIBUTING.md) always welcome! Enjoy!
