---
title: "Find Kubernetes Pods by Annotation Using kubectl"
date: 2020-09-19T00:00:00+00:00
author: duffn
layout: post
permalink: /find-kubernetes-pods-by-annotation/
description: "Quickly find pods with specific annotations using kubectl."
categories: kubernetes
tags: [kubernetes, kubectl, jq]
---

Sometimes you may need to retrieve pods with `kubectl` using an attribute that isn't available as a label or filter, such as an annotation. You can due this by getting all pods and filtering using `jq`.

```bash
kubectl get pods --chunk-size=0 -o json | \
  jq -r '.items[] | select(.metadata.annotations.myAnnotation == "myValue") | .metadata.name'
```

Now you have a list of all of the pod names that have `myValue` for `myAnnotation`.

> Note that if you have thousands and thousands of pods, this isn't going to be the fastest thing in the world, but it is still going to work.
