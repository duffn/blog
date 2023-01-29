---
title: "Caching Astro fetch Requests"
date: 2023-01-29T00:00:00+00:00
author: duffn
layout: post
permalink: /caching-astro-fetch-requests/
description: "eleventy-fetch isn't just a plugin for 11ty."
categories: astro
tags: [astro, eleventy, 11ty, ssg]
---

Caching `fetch` requests, particularly when developing locally, is a nice addition to your workflow when you're working on your remote data-powered, [Astro](https://astro.build/) static site.

One of the packages that I've used in the past with [Eleventy](https://www.11ty.dev/), another great SSG, is [`eleventy-fetch`](https://github.com/11ty/eleventy-fetch).

`eleventy-fetch` is a simple package that caches `fetch` requests based upon a few configuration options. And while the package calls itself a "a plugin for the Eleventy static site generator", there's no reason that it can't be used with other frameworks!

So, sorry if you were looking for some complex method to cache requests with Astro, but I'm going to tell you that if you're looking to cache your requests to avoid hitting that API too frequently when developing locally, just use `eleventy-fetch`, even with Astro. It'll save you some time and headache.
