---
title: "Test Promtail Pipeline Stages Locally on Your Machine"
date: 2022-03-19T00:00:00+00:00
author: duffn
layout: post
permalink: /test-promtail-pipeline-stages-locally/
description: "Test your Promtail pipelines locally on your machine before deploying"
categories: laravel
tags: [promtail, logging, grafana, loki]
---

If you use [Loki](https://grafana.com/oss/loki/) as your log aggregation system, then you're likely familiar with [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/), the agent that ships your local logs to a private Grafana instance or [Grafana Cloud](https://grafana.com/).

Promtail allows you to write powerful and complex [pipelines](https://grafana.com/docs/loki/latest/clients/promtail/pipelines/) that can transform your logs prior to export to your Loki instances. For example, perhaps you want to use a regex to extract your log lines into searchable fields, or add a new label to your logs, or drop log lines that you don't need.

And while you may think that you need to deploy updates to your Promtail configuration in order to test them, you can actually test these locally first with a few simple steps!

In this example, we'll drop log lines that we don't need in Loki.

## Install Promtail

If you're on macOS, you can install with brew.

```
brew install promtail
```

There are also [installation options](https://grafana.com/docs/loki/latest/clients/promtail/installation/) for other operating systems.

## Create Example Logs

Create a file that contains example logs lines that you want to test. We'll save this as `logs.txt`.

```
{"remote_ip":"1.2.3.4","request_id":"-","response_code":"200","request_method":"POST","request_path":"/actual-endpoint","request_querystring":"","request_timetaken":"6120","response_length":"22"}
{"remote_ip":"2.3.4.5","request_id":"-","response_code":"200","request_method":"GET","request_path":"/metrics","request_querystring":"","request_timetaken":"2476","response_length":"4227"}
{"remote_ip":"1.2.3.4","request_id":"-","response_code":"204","request_method":"GET","request_path":"/healthcheck","request_querystring":"","request_timetaken":"1198","response_length":"0"}
```

## Create a Promtail Configuration File

This is a regular Promtail configuration file with the caveat that _you can only have one scrape config_. Also, if you need to add labels to log lines, you can with a `static_configs` section.

```yaml
clients:
  - url: http://localhost:3100/loki/api/v1/push
scrape_configs:
  - job_name: testing-my-job-drop
    pipeline_stages:
      - match:
          selector: '{job="my-job"}'
          stages:
            - json:
                expressions:
                  request_path:
            - drop:
                source: "request_path"
                expression: "(/healthcheck|/metrics)"
          drop_counter_reason: my_job_health_or_metrics
    static_configs:
      - labels:
          job: my-job
```

Here we're saying only match on logs that have the `my-job` `job` label. We add that label to the log lines with the `static_configs` section. We then extract `request_path` from the JSON log line and then drop all lines that have `/healthcheck` or `/metrics`. Save this one as `promtail.yml`.

## Run Promtail in Dry Run Mode

Now that we're setup, we can run Promtail locally and test our config!

```bash
$ cat logs.txt | promtail --config.file ./promtail.yml --stdin --dry-run --inspect
Clients configured:
----------------------
url: http://localhost:3100/loki/api/v1/push
batchwait: 1s
batchsize: 1048576
follow_redirects: false
backoff_config:
  min_period: 500ms
  max_period: 5m0s
  max_retries: 10
timeout: 10s
tenant_id: ""
stream_lag_labels: filename

level=info ts=2022-03-19T15:24:24.318359Z caller=server.go:260 http=[::]:80 grpc=[::]:9095 msg="server listening on addresses"
[inspect: json stage]:
{stages.Entry}.Extracted["request_path"]:
	+: /actual-endpoint
2022-03-19T09:24:24.317955-0600{job="my-job"}	{"remote_ip":"1.2.3.4","request_id":"-","response_code":"200","request_method":"POST","request_path":"/actual-endpoint","request_querystring":"","request_timetaken":"6120","response_length":"22"}
level=info ts=2022-03-19T15:24:24.318525Z caller=main.go:119 msg="Starting Promtail" version="(version=, branch=, revision=)"
[inspect: json stage]:
{stages.Entry}.Extracted["request_path"]:
	+: /metrics
[inspect: json stage]:
{stages.Entry}.Extracted["request_path"]:
	+: /healthcheck
```

There it is!

We can see that Promtail extracted the `request_path` from our JSON log lines and then drops the lines that contain `/healthcheck` or `/metrics` leaving us with the single `/actual-endpoint` line.

## Conclusion

So, with a little setup and a carefull crafted Promtail configuration, you can test your pipeline stages locally and feel confident that your production setup will work as expected.
