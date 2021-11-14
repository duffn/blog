---
title: "Get Kubernetes Nodes and Pods with Custom Columns Using kubectl"
date: 2021-02-16T00:00:00+00:00
author: duffn
layout: post
permalink: /get-kubernetes-nodes-with-custom-columns/
description: "Add custom columns to your kubectl get nodes or pods output."
categories: kubernetes
tags: [kubernetes, kubectl, aws]
---

`kubectl` has so many ways that you can format and customize output, it can be overwhelming. One of my favorite ways to customize node output is the [`custom-columns`](https://kubernetes.io/docs/reference/kubectl/overview/#custom-columns) option. With this option you can tell `kubectl` exactly what columns you want in your `kubectl get nodes` output.

For example, if you wanted to only get the name of your nodes plus a couple of labels you could use the below to specify only those columns.

```bash
kubectl get no -o=custom-columns=NAME:.metadata.name,"MY CUSTOM LABEL":".metadata.labels.me/my-custom-label","AWS NODE SIZE":".metadata.labels.beta\.kubernetes\.io/instance-type"
NAME

NAME                          MY CUSTOM LABEL   AWS NODE SIZE
ip-10-10-1-123.ec2.internal   value1            t3.large
ip-10-10-1-124.ec2.internal   <none>            t3.large
ip-10-10-1-125.ec2.internal   value2            t3.large
ip-10-10-1-126.ec2.internal   value2            r5a.large
ip-10-10-2-127.ec2.internal   value2            r5a.large
```

A couple of notes here:

- This example assumes EKS for the `metadata.labels.beta.kubernetes.io/instance-type` label, so adjust accordingly for whatever columns you'd like to retrieve.
- Be sure to escape the periods in any of the fields that you'd like to retrieve, like `.metadata.labels.beta\.kubernetes\.io/instance-type`.
- Also try the `custom-columns` option with pods!
