---
title: "Introducting GReleaser"
date: 2018-08-19T00:00:00+00:00
permalink: /introducting-greleaser/
description: "Do you use Jira? Do you use GitHub? Do you create versions in Jira and also releases in GitHub? Are you tired of manually tagging commits in GitHub and copying release notes from Jira?"
tags: [jira, node.js, git, github]
---

Do you use Jira? Do you use GitHub? Do you create versions in Jira and also releases in GitHub? Are you tired of manually tagging commits in GitHub and copying release notes from Jira?

Then GReleaser is for you!

GReleaser is a command line tool that allows you to automatically create releases in GitHub using Jira release notes and names.

To create a release in GitHub from a Jira version, it can be as simple as passing the Jira project, version and the name of your GitHub repo.

```bash
greleaser -p 10003 -v 10001 -g my-github-project
```

There are many more options that can be passed in, including the commit to tag, tag name, release name and more.

See the full details at [https://github.com/duffn/greleaser](https://github.com/duffn/greleaser).
