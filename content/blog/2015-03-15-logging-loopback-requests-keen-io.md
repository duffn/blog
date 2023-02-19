---
title: "Logging LoopBack Requests to Keen.io"
date: 2015-03-15T00:00:00+00:00
permalink: /logging-loopback-requests-keen-io/
description: "Learn how to log LoopBack requests to Keen.io."
tags: [loopback, keen.io, node.js]
---

I've been experimenting with [LoopBack](http://loopback.io/) recently, using it to prototype an API. It really is quite amazing how quickly you can get a full-fledged API up and running with LoopBack. Just follow the [getting started guide](http://loopback.io/) and you’ll see what I mean.

After getting a pretty decent API up and running, I began testing a few different ways of logging API requests. I still haven't quite settled on the way that I want to do this yet, but one issue that I did encounter was trying to get the response time of the call from [expressjs/response-time](https://github.com/expressjs/response-time).

Response-time is a great, little Node.js module that automatically adds an X-Response-Time header to your responses to indicate the time in milliseconds that the request took.

The issue that I was running into was that I could not figure out how to properly get to that header in the LoopBack afterRemote method that I was using to log requests. Thanks to an [SO](http://stackoverflow.com/questions/28917210/can-i-get-to-response-headers-in-loopback-afterremote-hook/29028330#29028330) user, however, I was able to eventually get to what I wanted. The key was that the response wasn't available when I was trying to log the query -I needed to use the `res.on('finish')` event to get the response headers.

```javascript
var Keen = require("keen-js");

module.exports = function (myModel) {
	myModel.afterRemote("*", function (ctx, affectedModelInstance, next) {
		// Listen to the finish event of the response to wait
		// for the response to be available
		ctx.res.on("finish", function () {
			var client = new Keen({
				projectId: "keenprojectid",
				writeKey: "keenwritekey",
			});

			var queryEvent = {
				ip: ctx.req.ip,
				baseUrl: ctx.req.baseUrl,
				url: ctx.req.url,
				route: ctx.req.route,
				query: ctx.req.query,
				method: ctx.methodString,
				// Here's the header that I wanted to grab
				responseTime: Number(ctx.res._headers["x-response-time"]),
				keen: {
					timestamp: new Date().toISOString(),
				},
			};

			client.addEvent("queries", queryEvent, function (err, res) {
				if (err) {
					console.log(err);
				} else {
					console.log(res);
				}
			});
		});

		// Don't forget to call next
		next();
	});
};
```
