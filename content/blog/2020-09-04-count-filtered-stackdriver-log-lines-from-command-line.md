---
title: "Count Filtered Stackdriver Log Lines from the Command Line"
date: 2020-09-04T00:00:00+00:00
permalink: /count-filtered-stackdriver-log-lines-from-command-line/
description: "Easily count log lines in Stackdriver from the command line."
tags: [gcp, stackdriver, gcloud, logging, jq]
---

You can easily get log lines from Stackdriver with the `gcloud` CLI. Tack on `jq` to count the lines for a quick way to get a sense of log volume.

In this example, we're counting lines ingested from a Kubernetes cluster, but you can query whatever types of lines you like, but watch out for the `timestamp`! If you have a large volume of logs, this operation could take a long time.

```bash
gcloud logging read 'resource.type=container AND
  resource.labels.cluster_name=my-cluster AND
  resource.labels.namespace_id=default AND
  logName=projects/my-project/logs/my-log-group AND
  textPayload:"your operation failed" AND
  timestamp>="2020-07-01T00:00:00Z"' --format=json | jq '. | length'
```
