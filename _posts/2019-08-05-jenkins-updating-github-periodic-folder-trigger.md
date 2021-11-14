---
title: "Updating the GitHub Periodic Folder Trigger in Jenkins"
date: 2019-08-05T00:00:00+00:00
author: duffn
layout: post
permalink: /jenkins-updating-github-periodic-folder-trigger/
description: "Update the Periodic Folder Trigger in the Jenkins GitHub Branch Source plugin."
categories: jenkins
tags: [jenkins, github, cloudbees]
---

The [GitHub Branch Source Plugin](https://plugins.jenkins.io/github-branch-source)
is a plugin for Jenkins, maintained by [CloudBees](https://www.cloudbees.com/), allows you to organize your multibranch projects
and organization folders from GitHub.

The plugin allows you to easily integrate your GitHub repositories in your organization by
simply adding some credentials and even filtering based upon your repository names.

One thing that this plugin does by default is scan all of your repositories and builds
any branches and PRs that it finds - and by default it does this once per day. This can be
undesirable for a number of reasons, a few of which I encountered and so I attempted to turn
this feature off.

The catch, however, is that even if you turn off `Scan Organization Triggers` at the organization
level at https://myjenkins/job/myorg/configure, this does _not_ trickle down to the individual
repositories (why is this option even there, then?).

In addition, you can't actually turn off this feature or set the interval to perform the scans
at the repository level. You can only `View Configuration` for each repository, which
doesn't allow any editing.

So how do we turn this thing off? Well, I couldn't manage that, but was able to switch the
interval to the maximum 28 days, so at the very least I don't have to deal with this too often.

You can run the below in https://myjenkins/script in order to change the interval.

**WARNING**: This will change settings in your Jenkins instance.

```groovy
// Update the PeriodicFolderTrigger of each job inside of a Cloudbees folder.
// Useful for updating individual repos as you cannot do this through the UI.
// https://stackoverflow.com/questions/57077851/jenkins-github-plugin-scan-organization-triggers
import com.cloudbees.hudson.plugins.folder.computed.PeriodicFolderTrigger
import jenkins.model.Jenkins
import jenkins.branch.OrganizationFolder

println "Multibranch Items\n-------"
Jenkins.instance.getAllItems(org.jenkinsci.plugins.workflow.multibranch.WorkflowMultiBranchProject.class).each { it.triggers
       .findAll { k,v -> v instanceof com.cloudbees.hudson.plugins.folder.computed.PeriodicFolderTrigger }
       .each { k,v -> setInterval(it) }
}

def setInterval(folder) {
  println "[INFO] : Updating ${folder.name}... "
  folder.getTriggers().find {triggerEntry ->
    def key = triggerEntry.key
    if (key instanceof PeriodicFolderTrigger.DescriptorImpl){
      println "[INFO] : Current interval : " + triggerEntry.value.getInterval()
      // Valid intervals are here:
      // https://github.com/jenkinsci/cloudbees-folder-plugin/blob/master/src/main/java/com/cloudbees/hudson/plugins/folder/computed/PeriodicFolderTrigger.java#L261-L278
      def newInterval = new PeriodicFolderTrigger("28d")
      folder.addTrigger(newInterval)
      folder.save()
      println "[INFO] : New interval : " + newInterval.getInterval()
    }
  }
}
```
