---
title: "rsync While Specifying an SSH key"
date: 2020-10-17T00:00:00+00:00
permalink: /rsync-while-specifying-an-ssh-key/
description: "rsync While Specifying an SSH key."
tags: [rsync, ssh, shell, bash]
---

If you need to `rsync` to a server and specify a different SSH than your default you certainly can, though it's not initially intuitive. If you read the [man page](https://linux.die.net/man/1/rsync), however, and see the `-e` flag, then it starts to make sense. This flag says that it's used to "specify the remote shell to use". With that, we can create an `rsync` command to use a different SSH key.

```bash
rsync -azP -e "ssh -i ~/.ssh/my_other_key" --delete local_dir/ my_remote_user@12.34.56.789:/remote_dir/
```
