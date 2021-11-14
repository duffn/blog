---
title: "Using Minikube with Local Docker Images"
date: 2018-09-20T00:00:00+00:00
author: duffn
layout: post
permalink: /minikube-local-docker-images/
description: "Use locally built Docker images instead of images hosted in a registry when developing with minikube."
tags: [kubernetes, docker, minikube]
---

When developing locally with [Minikube](https://kubernetes.io/docs/setup/minikube/), you may want to use locally built Docker images instead of images hosted in a registry. Why push images up to Google's GCR or AWS ECR if you're only testing locally? Thankfully, this is simple with only a few steps.

- With Minikube running, `eval` the `docker-env` to configure your shell to get started.

```bash
eval $(minikube docker-env)
```

- Build your image.
- In the `containers` specification of your deployment, use the locally built image and specify the [`imagePullPolicy`](https://kubernetes.io/docs/concepts/containers/images/#pre-pulling-images). Setting this policy to `IfNotPresent` tells Kubernetes to use a local image preferentially.

```yaml
spec:
  containers:
    # This is for local use on minikube
    - image: nginx:20180919
      name: nginx
      imagePullPolicy: IfNotPresent
```

- Apply the deployment and that's it! You have a locally built image running in Minikube on your machine.
