---
layout: layouts/home.njk
eleventyNavigation:
  key: Search
  order: 3
---

<link href="{{ "/_pagefind/pagefind-ui.css" | bustCache }}" rel="stylesheet">
<style>
  @media (prefers-color-scheme: dark) {
    :root {
      --pagefind-ui-primary: #eeeeee;
      --pagefind-ui-text: #eeeeee;
      --pagefind-ui-background: #152028;
      --pagefind-ui-border: #e0e0e0;
      --pagefind-ui-tag: #152028;
    }
  }
</style>

<div id="search" class="search"></div>
<script src="{{ "/_pagefind/pagefind-ui.js" | bustCache }}" onload="new PagefindUI({ element: '#search', showImages: false });"></script>
