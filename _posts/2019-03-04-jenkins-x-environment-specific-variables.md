---
title: "Jenkins X Environment Specific Variables"
date: 2019-03-04T00:00:00+00:00
author: duffn
layout: post
permalink: /jenkins-x-environment-specific-variables/
description: "Leverage the dynamism of Helm charts in Jenkins X to allow different environment variables per environment and even per pull request."
categories: jenkins
tags: [jenkins, jenkinsx, kubernetes, helm, ci/cd]
---

[Jenkins X](https://jenkins.io/projects/jenkins-x/) is the self described
CI/CD solution for modern cloud applications on Kubernetes.

Jenkins X uses [Helm](https://helm.sh/) to manage Kubernetes deployments. This short post is about leveraging the dynamism
of Helm charts to allow different environment variables per environment and even per pull request.

## Prerequisite

You have a cluster running that is managed by Jenkins X. There are plenty of [getting started tutorials](https://jenkins-x.io/getting-started/) on the Jenkins X website and since this post
is more about how to use Helm with Jenkins X, we won't be covering installation.

## Jenkins X

Jenkins X subscribes to the GitOps methodology, which means the Git is used as a single
source of truth for continuous delivery. With Jenkins X, Helm charts are kept in your
code repository, builds are triggered by PRs or branch merges, and Jenkins X handles
deploying to either a [preview](https://jenkins.io/projects/jenkins-x/#pull-request-preview-environments) or an environment based upon your configuration.

It's likely that you would want different environment variables based upon which
environment your application is deployed. I'm going to show you how to do this.

## Builds

For our contrived use of environment variables, we're going to use a slightly modified version of the Jenkins X Python quickstart
application. You can find the finished product on [GitHub](https://github.com/duffn/jx-environment-variables).

### PRs and Staging

For pull requests, we want our app to show the Jenkins branch name as noted in the built-in
`BRANCH_NAME` environment variable. If we merge to master and deploy to staging,
we'll just set the branch as `master`.

For building previews of pull requests, Jenkins X uses a simple Makefile to inject
values into a few of the Helm YAML files. We're going to follow this approach, take the `BRANCH_NAME` variable from Jenkins and use that in our deployment definition.

We're going to add a `branchName` key under the `preview` key in the
charts/preview/values.yaml file. `preview` will look like this:

```yaml
preview:
  image:
    repository:
    tag:
    pullPolicy: IfNotPresent
  branchName:
```

Now, in charts/preview/Makefile, we'll grab the `BRANCH_NAME` environment variable
from Jenkins. After creating the two `branchName` lines, the Makefile will add our new
environment variable to our preview values.

```makefile
OS := $(shell uname)

preview:
ifeq ($(OS),Darwin)
  sed -i "" -e "s/version:.*/version: $(PREVIEW_VERSION)/" Chart.yaml
  sed -i "" -e "s/version:.*/version: $(PREVIEW_VERSION)/" ../*/Chart.yaml
  sed -i "" -e "s/tag:.*/tag: $(PREVIEW_VERSION)/" values.yaml
  # This is a new line.
  sed -i "" -e "s/branchName:.*/branchName: $(BRANCH_NAME)/" values.yaml
else ifeq ($(OS),Linux)
  sed -i -e "s/version:.*/version: $(PREVIEW_VERSION)/" Chart.yaml
  sed -i -e "s/version:.*/version: $(PREVIEW_VERSION)/" ../*/Chart.yaml
  sed -i -e "s|repository:.*|repository: $(DOCKER_REGISTRY)\/duffn\/jx-environment-variables|" values.yaml
  sed -i -e "s/tag:.*/tag: $(PREVIEW_VERSION)/" values.yaml
  # This is a new line.
  sed -i -e "s/branchName:.*/branchName: $(BRANCH_NAME)/" values.yaml
else
  echo "platfrom $(OS) not supported to release from"
  exit -1
endif
  echo "  version: $(PREVIEW_VERSION)" >> requirements.yaml
  jx step helm build
```

Now, we're going to add an `env` section at the bottom of charts/jx-environment-variables/values.yaml.

```yaml
env:
  - name: BRANCH_NAME
    value: "master"
```

Finally, in charts/jx-environment-variables/templates/deployment.yaml, we'll utilize
the variables that we setup. If our values.yaml contains `branchName`, we'll display
that in our app, otherwise we'll get the `env` section that we created above.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: {{ '{{' }} template "fullname" . }}
  labels:
    draft: {{ '{{' }} default "draft-app" .Values.draft }}
    chart: "{{ '{{' }} .Chart.Name }}-{{ '{{' }} .Chart.Version | replace "+" "_" }}"
spec:
  replicas: {{ '{{' }} .Values.replicaCount }}
  template:
    metadata:
      labels:
        draft: {{ '{{' }} default "draft-app" .Values.draft }}
        app: {{ '{{' }} template "fullname" . }}
{{ '{{' }}- if .Values.podAnnotations }}
      annotations:
{{ '{{' }} toYaml .Values.podAnnotations | indent 8 }}
{{ '{{' }}- end }}
    spec:
      containers:
      - name: {{ '{{' }} .Chart.Name }}
        image: "{{ '{{' }} .Values.image.repository }}:{{ '{{' }} .Values.image.tag }}"
        imagePullPolicy: {{ '{{' }} .Values.image.pullPolicy }}
        ports:
        - containerPort: {{ '{{' }} .Values.service.internalPort }}
{{ '{{' }}/*
Here's the section we added.
*/}}
{{ '{{' }}- if .Values.env }}
        env:
{{ '{{' }}- if .Values.branchName }}
          - name: BRANCH_NAME
            value: {{ '{{' }} .Values.branchName | quote }}
{{ '{{' }}- else }}
{{ '{{' }} toYaml .Values.env | indent 10 }}
{{ '{{' }}- end }}
{{ '{{' }}- end }}
        resources:
{{ '{{' }} toYaml .Values.resources | indent 12 }}
```

When we build a PR, we can see that we'll have `PR-XX` output by our app and
when we build staging, we'll see `master`. It works!

### Production

Okay, but now we want to display something different when we promote our staging
image to production. How do we do that since it's going to use exactly the same image
and the values.yaml file as well?

Thankfully, you can override values per environment using the production repository
that Jenkins X creates for us when we initialize our cluster. Mine happens to be
called [environment-razorfortune-production](https://github.com/duffn/environment-razorfortune-production). This is the repository that Jenkins X will
use when we promote an image to our production environment.

In the env/values.yaml file in this repository, let's add our production variable.

```yaml
jx-environment-variables:
  env:
    - name: BRANCH_NAME
      value: "i am in production!"
```

_Note_: You need to add any overriding values underneath a key that is the exact
name of your application. Don't let this trip you up!

And now when we promote our image to production, we'll get `i am in production!`.

## Conclusion

This example was a bit contrived, but now you can see some of the power in utilizing
Jenkins X for continuous delivery in Kubernetes. Now you have environment specific variables (or anything that you can
put in values.yaml for that matter) in Jenkins X. Get to it!
