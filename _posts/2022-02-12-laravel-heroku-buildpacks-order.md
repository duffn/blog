---
title: "Laravel and Heroku or Dokku Buildpacks Order with Tailwind CSS"
date: 2022-02-12T00:00:00+00:00
author: duffn
layout: post
permalink: /laravel-heroku-buildpacks-order/
description: "Make sure you add your buildpacks in the correct order!"
categories: laravel
tags: [php, laravel, heroku, dokku, tailwind, tailwindcss]
---

For most of my side projects I use either [Dokku](https://dokku.com/) or sometimes [Heroku](https://www.heroku.com/). Heroku is nice for a quick POC, but quickly becomes too expensive. Dokku is great to run on a $5 VPS along with your database, cache, queue, and whatever else you need.

At any rate, I'm using the both excellent [Tailwind](https://tailwindcss.com/) and [`laravel-livewire-tables`](https://github.com/rappasoft/laravel-livewire-tables). The configuration of `laravel-livewire-tables`, which uses Tailwind by default, requires that you tell Tailwind v3 not to purge classes for the tables by setting your `tailwind.config.js` `content` section like this.

```javascript
module.exports = {
    content: [
        ...
        './vendor/rappasoft/laravel-livewire-tables/resources/views/tailwind/**/*.blade.php',
    ],
    ...
};
```

Perfect, I've done this before and that makes sense.

As I get my project up and running, this works fine locally both when building for dev and production, but when I deploy with Dokku to my staging server, the classes for `laravel-livewire-tables` are getting purged. Why does this work locally but not when I deploy with Dokku?

After what is probably an embarassingly long amount of time debugging, I finally realized.

My `.buildpacks` file.

```
https://github.com/heroku/heroku-buildpack-nodejs
https://github.com/heroku/heroku-buildpack-php
```

See it?

Based upon the order of my buildpacks, Dokku is running the Node.js buildpack first before the php buildpack. That means that when my deployment process is running `npm run build`, the directory that contains the `laravel-livewire-tables` Blade templates isn't even there yet! I need to run the php buildpack first so those dependencies are installed before running `npm run build`, otherwise there are no templates there to reference. 🤦

So, if you have encountered a case where you're deploying to Heroku or Dokku and Tailwind seemingly is purging vendor styles that it shouldn't, make sure you're installing those dependencies first!
