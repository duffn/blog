---
title: "Analyzing Cloudflare Logs with AWS Athena"
date: 2019-08-02T00:00:00+00:00
author: duffn
layout: post
permalink: /analyzing-cloudflare-logs-with-aws-athena/
description: "As with many features with Cloudflare, you can enable their Logpush service with the click of a button. If you are using AWS S3 for your storage, you can then utilize Athena to analyze your logs."
categories: cloudflare
tags: [aws, cloudflare, athena]
---

As with many features with Cloudflare, you can enable their [Logpush service](https://developers.cloudflare.com/logs/logpush/) with the click of a button. Logpush sends
your HTTP request logs to your cloud storage provider every 5 minutes.

If you are using AWS S3 for your storage, you can then utilize [Athena](https://aws.amazon.com/athena/) to
analyze your logs.

> Athena is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL.

So with a little setup and some simple SQL, you can analyze your Cloudflare logs.

## Table DDL

In order to being querying, however, you need to [create an external table](https://docs.aws.amazon.com/athena/latest/ug/create-table.html) in Athena that
matches the format of your Cloudflare logs, which are JSON with a newline delineating each record.

Luckily, this is pretty easy to setup in Athena. Here is
the DDL for all of the fields currently included in Cloudflare Logpush.

_Note:_ You can customize the [fields](https://developers.cloudflare.com/logs/logpull-api/requesting-logs/#fields) that Logpush includes, so if you have, your
list of fields may not match the below exactly.

```sql
CREATE EXTERNAL TABLE cloudflare_logs (
    CacheCacheStatus string,
    CacheResponseBytes int,
    CacheResponseStatus int,
    CacheTieredFill boolean,
    ClientASN int,
    ClientCountry string,
    ClientDeviceType string,
    ClientIP string,
    ClientIPClass string,
    ClientRequestBytes int,
    ClientRequestHost string,
    ClientRequestMethod string,
    ClientRequestPath string,
    ClientRequestProtocol string,
    ClientRequestReferer string,
    ClientRequestURI string,
    ClientRequestUserAgent string,
    ClientSSLCipher string,
    ClientSSLProtocol string,
    ClientSrcPort int,
    EdgeColoID int,
    EdgeEndTimestamp string,
    EdgePathingOp string,
    EdgePathingSrc string,
    EdgePathingStatus string,
    EdgeRateLimitAction string,
    EdgeRateLimitID int,
    EdgeRequestHost string,
    EdgeResponseBytes int,
    EdgeResponseCompressionRatio double,
    EdgeResponseContentType string,
    EdgeResponseStatus int,
    EdgeServerIP string,
    EdgeStartTimestamp string,
    FirewallMatchesActions ARRAY < string >,
    FirewallMatchesSources ARRAY < string >,
    FirewallMatchesRuleIDs ARRAY < string >,
    OriginIP string,
    OriginResponseBytes int,
    OriginResponseHTTPExpires string,
    OriginResponseHTTPLastModified string,
    OriginResponseStatus int,
    OriginResponseTime bigint,
    OriginSSLProtocol string,
    ParentRayID string,
    RayID string,
    SecurityLevel string,
    WAFAction string,
    WAFFlags string,
    WAFMatchedVar string,
    WAFProfile string,
    WAFRuleID string,
    WAFRuleMessage string,
    WorkerCPUTime int,
    WorkerStatus string,
    WorkerSubrequest boolean,
    WorkerSubrequestCount int,
    ZoneID bigint)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' LOCATION 's3://my-cloudflare-logs/'
```

Of course, change `s3://my-cloudflare-logs/` to the name of your bucket that you used
when setting up Logpush.

## Querying

An now that we have a table created in Athena, we can analyze our logs in a myriad of ways.

How about checking to see how many requests you've received by the request protocol?

```sql
SELECT count(*) as requests,
         c.clientrequestprotocol
FROM "cloudflare_logs"."cloudflare_logs" c
GROUP BY  c.clientrequestprotocol
ORDER BY  count(*) DESC limit 10;
```

```
 	requests    clientrequestprotocol
1	15063737    HTTP/2
2	6842951     HTTP/1.1
3	4342	    HTTP/1.0
```

Or maybe for reasons unknown, you want to see the average client request size in bytes for today,
grouped by the Cloudflare edge colo ID.

```sql
SELECT avg(clientrequestbytes),
         edgecoloid
FROM "cloudflare_logs"."cloudflare_logs"
-- Assuming you are using the default date format with Logpush
WHERE date_trunc('day', from_iso8601_timestamp(edgestarttimestamp)) = current_date
GROUP BY  edgecoloid
ORDER BY  avg(clientrequestbytes) ASC;
```

As you can imagine, the ways that you can slice and dice your Cloudflare HTTP logs
is nearly limitless. Enjoy diving deep on your Cloudflare logs!
