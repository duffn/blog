---
title: "Supervisor Reread Update"
date: 2016-07-30T00:00:00+00:00
permalink: /supervisor-reread-update/
description: "Why won't you update supervisor!?"
tags: [supervisor, supervisord, linux]
---

I needed to add some options to [gunicorn](https://gunicorn.org/) running under [supervisor](http://supervisord.org/) the other day. I ran gunicorn from the command line with the new argurments — no problem. I added the arguments to the command in supervisor, restarted with supervisorctl — problem.

Supervisor wasn’t picking up the new arguments. I restarted again. Nothing. I tried the command arguments inline. I tried a gunicorn [conf](http://docs.gunicorn.org/en/stable/settings.html#config-file) file. I tried a shell script with the commands. All nothing.

After spending an embarrassing amount of time trying to figure out what was wrong, I finally remembered! You need to reread and update the supervisor conf file after making changes. Simply restarting via supervisorctl is not enough.

```bash
supervisorctl reread
supervisorctl update
```

So don’t forget. Reread and update after changing a supervisor configuration, otherwise it won’t get picked up.
