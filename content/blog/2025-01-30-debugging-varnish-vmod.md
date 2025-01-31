---
title: "Debugging a Varnish VMOD with gdb"
date: 2025-01-30T00:00:00+00:00
permalink: /debugging-varnish-vmod-with-gdb/
description: "Debug your custom Varnish VMOD with gdb"
tags: [varnish, vmod, gdb, debugging]
---

# Introduction

Proper debugging is essential for any application and not just your standalone applications. You can, and should, also debug your shared libraries like your custom Varnish VMOD.

We'll walk through a few steps to debug your Varnish VMOD with gdb in Docker.

# Setup

Rather than walk through how to create a custom VMOD from scratch, which you likely already know how to do if you're reading a post on debugging one, we'll use my [`libvmod-querymodifier`](https://github.com/duffn/libvmod-querymodifier) to jumpstart us as it already has an example useful for debugging.

- Clone the repo.

```bash
git clone git@github.com:duffn/libvmod-querymodifier.git
```

- Navigate to the `debug` directory and build the Docker Compose setup.

```bash
docker compose up --build
```

Take a look at the Dockerfile to review the setup for debugging. There are a couple of things worth noting here.

Here we set some CFLAGS that are helpful for debugging, enabling debug symbols and disabling compiler optimizations. `--enable-asan` is not necessary for our `gdb` debugging, but it's a good idea to check for any memory leaks while we're at it.

```bash
./bootstrap --enable-asan CFLAGS="-g -O0"
```

This next tip is especially necessary as otherwise `gdb` won't be able to find the symbols for your VMOD. You have to ensure that it knows to look in `/var/lib/varnish/varnishd/vmod_cache` for your VMOD. Varnish will cache the VMODs that it will use in this directory.

❗️Don't tell `gdb` to look in the original VMOD directory were all the `.so` files are located! You must instruct it to use the `vmod_cache` director as the search path for shared objects.

You can either do this after launching `gdb` or by including a `.gdbinit` file as shown in the repository example.

- Exec into the Varnish container.

```bash
docker compose exec varnish
```

- Start `gdb` and attach it to the Varnish child process. You can either get the PID with `ps` or Varnish will print the child PID to the console like `varnish-1  | Debug: Child (31) Started.`

```bash
(gdb) attach 31
```

- Set a breakpoint, for example on the `vmod_modifyparams` function.

```bash
(gdb) b vmod_modifyparams
Breakpoint 1 at 0xffff7e0b14cc: file vmod_querymodifier.c, line 219.
```

- Send a request to `http://localhost:8080` that exercises the VMOD.

There's a default VCL file included that will run through the VMOD as an example.

- Continue the debugger and then use `gdb` as you normally would.

```bash
(gdb) c
Continuing.
[Switching to Thread 0xffff855cf140 (LWP 372)]

Thread 101 "cache-worker" hit Breakpoint 1, vmod_modifyparams (ctx=0xffff855cd9b8, uri=0xffff79a3d8ac "/?blah=1&ts=1", params_in=0xffff7e0e7610 "ts,v,cacheFix,date",
    exclude_params=1) at vmod_querymodifier.c:219
219         CHECK_OBJ_NOTNULL(ctx, VRT_CTX_MAGIC);
```

# Conclusion

Once you know where Varnish caches the VMOD symbols, the rest is easy. But make sure you tell `gdb` where to search for your VMOD symbols! After that, you can debug to your heart's content.
