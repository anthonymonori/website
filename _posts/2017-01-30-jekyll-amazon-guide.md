---
title: "Setting up a static website on Amazon AWS in 20 minutes"
layout: post
date: 2017-01-30 23:05
headerImage: false
tag:
- guide
blog: true
star: false
author: anthonymonori
description: Have a Jekyll-based static website running on AWS using S3 storage, Cloudfront CDN, Route 53 DNS and SSL in about 20 minutes.
---

_As a disclaimer, this guide was put together after setting up two different static websites on AWS during the weekend. The first one took a whole day and a half to get it right, then for the second time I have had it up and ready in about 20 minutes. Felt like I could help other folks out with a guide, that could potentially save hours of (precious) time._

## Requirements

All you need to get started is:
- an Amazon AWS account
- a Jekyll-based static website
- a domain name

This guide does not cover either of the above steps, so go ahead and create the account now, have the static website ready locally and buy a domain name if you haven't done it yet!

## Architecture

The setup is quite simple: you store your static website (as files) on Amazon S3 storage and serve it through Amazon Cloudfront. For this tutorial, we'll try to setup a website called `exemple.io` to be running on `https://` only and we are going to redirect traffic from www-subdomain to the APEX-domain (without the www).

One important note to take, is that in order to leverage the free SSL certificate Amazon provides, we need to use the US Standard (us-east-1 or US East N. Virginia) region across all services.

## Pricing

Another disclaimer, is that I do not want to be help accountable for any charges you might encounter at Amazon, so please consult their Pricing page. As of January 2017, the following monthly costs would occur for the website we are about to set up:
- **S3**: $0.023 per GB (for the first 50 TB)
- **Route 53**: $0.5 per hosted zone
- **Cloudfront**: $0.085 per GB (for the first 10 TB)
- **Domain**: yearly fee and varies based on registry
- **SSL**: free (if using Amazon AWS, will cover it later)

## Simple Storage Service (S3)

_Quick link: [https://console.aws.amazon.com/s3/home?region=us-east-1](https://console.aws.amazon.com/s3/home?region=us-east-1)_

To start with, we are going to set up **3 buckets**, corresponding to your domain. This is not a requirement for Cloudfront, but a good convention and a necessity if one day you decide to switch from Cloudfront CDN.

![The 3 buckets created according to our test dns](/assets/images/posts/jekyll-amazon-guide/s3-buckets.png)

`example.io` will be the main bucket we will push to, the `www.example.io` bucket will be used to redirect traffic from it to the main one, and the `logs.example.io` will be used to store logs.

Now let's set up the main bucket:
1. Go to properties, and enable Logging by targeting the `logs.example.io` bucket
2. Go to properties, enable the Static website hosting by picking the 'Use this bucket to host static website' option and write `index.html` for the Index document (Jekyll generates this file by default, but double check your setup).

![How to enable the static website hosting on the main bucket](/assets/images/posts/jekyll-amazon-guide/s3-bucket-static-website.png)

Now make sure the `www-bucket` redirects traffic to the main bucket:
1. Go to properties, enable the Static website hosting by picking the 'Redirect requests' option and write the main bucket (`example.io` in our case) as targeting
2. Optionally enable Logging on this bucket as well, as see above

![How to redirect traffic from www-bucket to main-bucket](/assets/images/posts/jekyll-amazon-guide/s3-bucket-redirect-to-main.png)

Now we need to set up the proper permission on the main bucket and the `www-bucket`:
1. Go the the main bucket, got to Permission and using the drop down menu, select the 'Bucket policy' option. Use the code below to allow public access to the objects stored. Make sure to replace `example.io` with your bucket name:

```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForGetBucketObjects",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example.io/*"
        }
    ]
}
```
2. Select the 'CORS configuration' option from the drop down menu, and make sure the following configuration is present:

```
<CORSConfiguration>
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Authorization</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```
3. Do the same for the `www-bucket`, just make sure to use the correct bucket name on the Bucket policy

And that it for the S3 setup. For now, just take a note of the endpoint and your region (which should be us-east-1)

## Access Management

_Quick link: [https://console.aws.amazon.com/iam/home?region=us-east-1](https://console.aws.amazon.com/iam/home?region=us-east-1)_

Now we are going to set up programmatic access via an API and a secret key, to ease the process of publishing to the main bucket

1. Go to IAM, select Policies from the left hand-side menu and press Create policy
2. Select the 'Create Your Own Policy' option
3. Give it a name, like `Example.io-Website-Access`
4. Use the following policy document, just make sure to use the correct bucket name when you do so:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "cloudfront:*"
            ],
            "Effect": "Allow",
            "Resource": [
                "*"
            ]
        },
        {
            "Action": [
                "s3:*"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::example.io",
                "arn:aws:s3:::example.io/*"
            ]
        }
    ]
}
```
5. Validate and Create policy
6. Now that we have a policy, we need to create a Group; go to the Groups and press New Group
7. Give it a name like `Websites-example.io` and attach the above create policy to it.
8. Now we need a user under this group; go to Users and press Add User
9. Give it a name like `example.io` and check 'Programmatic access'
10. On the next screen attach it to the newly created group and voila!

Make sure you note down the access key id and the secret access key, as we will need it later.

## SSL certificate

_Quick link: [https://console.aws.amazon.com/acm/home?region=us-east-1](https://console.aws.amazon.com/acm/home?region=us-east-1)_

Amazon provides free SSL certificate, until the distribution is within the N. Virginia datacenter. I have not researched the exact reasons of this, but I learned this the hard way, as you won't receive any errors doing otherwise, but the SSL certificate simply won't work. So again, please make sure everything you create is within the `us-east-1` region.

1. Request a certificate
2. Do not use a wildcard domain here - instead specify two domains:
  - `example.io`
  - `www.example.io`
3. Review and request - you'll receive two emails (one for each domain) and make sure to accept them. If you are unsure what email was request sent to, you can check it on the main ACM page (usually the `postmaster@example.io` and the registered email for the domain is a good guess).

That's it, you won't have to note down anything here, but we needed the certificate to be ready for our next step.

## Cloudfront

_Quick link: [https://console.aws.amazon.com/cloudfront/home?region=us-east-1](https://console.aws.amazon.com/cloudfront/home?region=us-east-1)_

Cloudfront CDN will serve your files from your S3 in a faster, more secure and a more lightweight form by:
- distributing geographically your content (not part of this guide)
- caching
- enabling SSL and forcing https
- encoding content using gzip

In order to start using it, we will need to create a so called distribution:

1. Create a new distribution and select Web
2. Don't use the drop down menu when selecting the Origin Domain name (very important, as then you won't be able to serve documents out of subfolders using the default index document e.g. `https://example.io/blog/` would not work but `https://example.io/blog/index.html` would), but instead use the S3 endpoint noted down earlier:

![How to setup a custom origin to your Cloudfront distribution](/assets/images/posts/jekyll-amazon-guide/cloudfront-setup-custom-origin.png)

3. Make sure to select 'Redirect HTTP to HTTPS'
4. Make sure to select 'Yes' for 'Compress Objects Automatically', as this will significantly lower your content-sizes by encoding it using gzip.
5. Put `example.io, www.example.io` as Alternate Domain Names (CNAMEs)
6. Select 'Custom SSL Certificate (example.com):' under the SSL Certificate option - if previously configures correctly, here you will be able to select your assigned SSL certificate for your domain: `example.io (guid)`
7. For 'Default Root Object' select `index.html`
8. Turn logging on and set it up accordingly:

![How to setup logging correctly on your Cloudfront distribution](/assets/images/posts/jekyll-amazon-guide/cloudfront-enable-logging.png)

9. Press 'Create Distribution' and voila!

Before we move on to setting up your hosted zone, note down the Distribution ID.

## Route 53

_Quick link: [https://console.aws.amazon.com/route53/home?region=us-east-1](https://console.aws.amazon.com/route53/home?region=us-east-1)_

Route 53 will be used to set up your DNS and A-records to `www` and the APEX domain. It will be the connecting clip between the end-users and your S3, by routing traffic to your Cloudfront distribution.

### Setting up the hosted zones

1. Go to 'Hosted zones' from the left hand-side menu and press 'Create Hosted Zone'
2. Enter your domain name: `example.io`
3. Make sure to use type: `Public Hosted Zone`

### Setting up your name servers (optional)

If your domain is hosted somewhere else (Namecheap, GoDaddy), then this is a very important part to follow, as we will bridge the domain from their registry to Amazon via their name-servers. Now that you created your Hosted Zone, you'll see that there are already two created sets:

![The two default created sets inside the hosted zone](/assets/images/posts/jekyll-amazon-guide/route-53-default-sets.png)

These four name servers needs to be set up on your registered domain. Please consult your Domain Registry on how to do this.

### Set up the A-records

You'll need two A-record (IPv4 address) to be set up: one for the APEX domain, and one for the www-subdomain. If you were fast enough, you might need to wait a couple of minutes until the Cloudfront distribution is processed, in case it does not show up during the steps below

1. Press Create Record set
2. Leave the name empty (will create a record for the APEX domain)
3. Type: `A - IPv4 address`
4. Alias: `Yes`
5. From the Alias Target drop down link, select the Cloudfront distribution (it picks it up based on the CNAME provided in the distribution). It is important that you select the Cloudfront distribution over the direct S3 instance, as otherwise the `https://` forwarding won't work!
6. Routing Policy: `Simple`
7. Evaluate Target Health: `No` -- unless you want some extra expenditure :-)
8. Save Record Setting
9. Repeat the steps once again, but this time, use `www` as the Name (everything else should be the same!)

Now we have routing set up! Last missing piece is actually pushing content to your S3!

## Jekyll

At this point, we are almost done, and we only need to have a working Jekyll static website. One important part here is to make sure in you `_config.yml` you use the `https://example.io` link as your base URL. Make sure to always `bundle exec jekyll build` before publishing!

### s3_websites

_Quick link: [https://github.com/laurilehmijoki/s3_website](https://github.com/laurilehmijoki/s3_website)_

This is a ruby gem that will ease your pain of publishing your website to this Amazon AWS setup. You can either follow the `README` on the GitHub repository, but I will collect the steps down below with a recommended configuration to go with:

1. `s3_website cfg create`
2. Open `s3_website.yml` with an editor and fill it out accordingly. Most of the information required has been noted down previously. Here's my recommended configuration to go with:

```
s3_id: #S3_ID
s3_secret: #S3_SECRET
s3_bucket: #MAIN_BUCKET e.g. example.io
cloudfront_distribution_id: #CLOUDFRONT_DISTRIBUTION_ID

index_document: index.html
max_age:
  "assets/*": 36000
  "*": 300

gzip:
  - .html
  - .css
  - .md
gzip_zopfli: true

s3_endpoint: us-east-1

s3_reduced_redundancy: true

# Outcomment the following in case you want to invalidate your cached content
# cloudfront_invalidate_root: true
# cloudfront_wildcard_invalidation: true
```

3. `s3_website cfg apply`
4. Make sure again you `bundle exec jekyll build` and that your `_config.yml` is set up correctly!
5. `s3_website push --verbose`

There you go, now you should be able to access your website via your endpoint and `https://example.io` too!

## Summary

This is it! You know have a lightweight, secure, fast and scalable static page. Obviously hosting anything more complex than a static website would require you to use something different than S3, but this setup should be perfect for any hobby project, landing pages for your next project or for a personal website.

Now we have achieved everything using the Amazon stack and are ready to scale up with the matter of couple tweaks, if needed.

---

Hope this guide served you well and helped you to set up your static website on Amazon AWS from zero to complete in under 20 minutes!

If you have any corrections, questions, suggestions and just want to say hi - reach out to me on Twitter [@anthonymonori](https://www.twitter.com/anthonymonori).
