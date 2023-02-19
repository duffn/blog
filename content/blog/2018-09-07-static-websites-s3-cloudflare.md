---
title: "Static Websites with AWS S3 and Cloudflare"
date: 2018-09-07T00:00:00+00:00
permalink: /static-websites-with-aws-s-3-and-cloudflare/
description: "Learn how to get started with your static site on S3 and your DNS in Cloudflare."
tags: [aws, cloudflare, s3, cloudfront]
---

Amazon Web Service's [S3](https://aws.amazon.com/s3/) is an excellent place to deploy a static website. If [Cloudflare](https://www.cloudflare.com/) is your DNS provider, however, there are a few tricks that you need to be aware of when setting up your static site.

We're going to talk about how to get started with your static site on S3 and your DNS in Cloudflare.

- [AWS](#AWS)
  - [S3 Bucket](#s3-bucket-configuration)
  - [Cloudfront Distribution](#cloudflare-distribution)
- [Cloudflare](#cloudflare)
  - [DNS](#dns)

## AWS

First, of course, we need a static site deployed to S3. The [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html) does a good job at explaining how to setup a site, but we'll briefly run through the steps here as well.

### S3 Bucket Configuration

One of the first things that you'll see in the AWS documentation linked above is that your bucket name must match the name of the website you are hosting. That's true in most cases, but since we're going to be using Cloudflare DNS, that's not true in our case.

Oh, and we don't even need to tell AWS that we're going to be using this bucket to host our site. I'll show you why shortly.

- Head to the [S3 home page](https://s3.console.aws.amazon.com/s3/home) and create a bucket named whatever you like.
- Give your bucket this policy, changing `example.com` to the name of the bucket you just created.

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicReadGetObject",
			"Effect": "Allow",
			"Principal": "*",
			"Action": ["s3:GetObject"],
			"Resource": ["arn:aws:s3:::example.com/*"]
		}
	]
}
```

**NOTE**: _This makes items in your bucket available to the public. Do not put things in this bucket that you do not want everybody in the world seeing._

- You can upload whatever HTML, CSS and JS you want to serve on your site, but for now just upload a simple `index.html` file for testing.

### Cloudfront Distribution

Now, we're going to create a [Cloudfront](https://aws.amazon.com/cloudfront/) distribution in AWS. While not strictly necessary for a static website on AWS, it is necessary to get Cloudflare's SSL to work with our site.

- On the [Cloudfront](https://console.aws.amazon.com/cloudfront/home) service page and click `Create Distribution`.
- Click `Get Started` under the Web section on the next page.
- In the `Origin Domain Name` input, select the bucket that you created above.
- We're going to select Redirect HTTP to HTTPS for the `Viewer Protocol Policy` option.
- There are only two other things that we need to change now. Here are those tricky bits that you may not have known about!
  - In the `Alternate Domain Names (CNAMEs)` input, type the domain name that we're going to use for our site. This your domain that you will or already have setup in Cloudflare.
  - In `Default Root Object`, enter `index.html`, the file that we uploaded the our bucket.
- That's all we need to do to setup Cloudfront. Hang tight though, as this can sometimes take awhile to become active.

## Cloudflare

We're in the final stretch now. We only have to setup Cloudflare to point to our Cloudfront distribution. From here on out, I'm going to assume that you already have an account and have setup your domain in Cloudflare.

### DNS

- On the DNS tab of your [Cloudflare dashboard](https://dash.cloudflare.com/), we're going to add two CNAME records.
  - Add one CNAME record with the Name of your domain, `example.com`, and the Domain Name of your Cloudfront distribution, `d1234gflgylt.cloudfront.net`. Make sure not to include `https://` with your Cloudfront distribution.
  - Add another record with the Name of `www` and again the Domain Name of your Cloudfront distribution. We want to make sure that we send both `www.example.com` and `example.com` to our static site.
- Make sure to leave the little Cloudflare cloud checked and orange.

And that's it! Wait a few minutes (most likely seconds), and you'll see your static site served from S3, through a Cloudfront distribution and finally to Cloudflare under your custom domain.
