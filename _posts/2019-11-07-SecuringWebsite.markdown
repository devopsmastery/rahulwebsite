---
layout: post
title:  "Securing website and improve response time through CDN"
subtitle: "Using AWS ACM & CloudFront for content distribution"
date:   2019-11-06 11:17:44 +0530
background: '/img/blogwebsite/https.jpg'
categories: blog
---
In my previous [blog](./posts/2019-11-06-BuildStaticWebsite.markdown){:target="blank"}, we had seen how to make a website live using *S3 static website and Route53*. Although our website is very much live, its not a secured website and affects page ranking that Google does. Google will soon be phasing out HTTP with Chrome and give a warning sign before access, which will be quite frustrating to our users. Also, we would like our website response time a lot quicker to serve our global customers.

**AWS Certificate Manager (ACM)** service provides us with *SSL certificates* which gives the ability to serve content over HTTPS thus encrypting the data.

We also need a **Content Delivery Network (CDN)** that will serve the website content to our global users. CDNs consists of edge locations that cache the data and serve to our users directly rather than go to our website server for each request.  

<p>AWS CDN - CloudFront - is a fully managed AWS service that serves content to users on your website with low latency, high data transfer speeds, and no minimum usage commitments. Cloudfront also uses HTTPS to get objects from your bucket, which means the communication is encrypted and safe.</p>

So lets get started with our next set of activities to make our website secure and fast.

***

##### What we will be using
- **AWS Certificate Manager:** For SSL Certificates.
- **CloudFront:** For distribution of content to edge locations
- **Route53:** Amazons DNS service to point to our Cloudfront distribution
- **S3_website utility:** For uploads and CloudFront invalidations

***

##### Step 1 - Using ACM (AWS Certificate Manager) for SSL certificates

-	Login to AWS and jump to ACM service
-	**NOTE** - Make sure your region is *N.Virginia (US East)*.
-	Click *Request a Certificate* and a public certificate
-	Add Domains - *awscloudtrainings.com* and *\*.awscloudtrainings.com* which means any subdomain.<br><br>
![ACM](/img/blogwebsite/acm1.jpg "Domain Certificates")<br><br>
-	We can use both DNS and Email validations and I'm going to keep it simple for this website to email validations.
-	Click *Email Validations* and complete the process by logging to the registered email id when you purchased the domain. You will receive a mail from AWS which needs to be approved.<br><br>
![ACM](/img/blogwebsite/acm2.jpg "ACM Approve")<br><br>
-	Once you approve on mail, on ACM you should see your certificate as *Issued*
-	Note that the certificate shows *In Use – No*. So our next step is to use it with CloudFront.<br><br>
![ACM](/img/blogwebsite/acm3.jpg "ACM Status") <br><br>

##### STEP 2: Creating Distribution in CloudFront and use ACM certificate.

When you want to use CloudFront to distribute your content, you create a **distribution** and choose the origin from where the content will be cached by CloudFront. The origin in our case is the S3 static bucket that we created earlier.

-	On AWS Console, go to *CloudFront* and click *Create Distribution*
-	As we have a static website, click on **Web Distribution-Get Started** <br><br>

![CDN](/img/blogwebsite/cdn1.jpg "CloudFront Distribution") <br><br>

--	**Origin Domain Name:** Put the S3 Endpoint of your bucket which you should see under **S3 Bucket->Permissions->StaticWebsite**. For E.g Endpoint : *http://awscloudtrainings.com.s3-website.ap-south-1.amazonaws.com*
-	**Viewer Protocol Policy:** Select *Redirect HTTP to HTTPS
-	Under **Distribution Setting -> Alternate Domain Names** - Put in both your fqdn and www domain <br><br>
![CDN](/img/blogwebsite/cdn2.jpg "CloudFront Distribution") <br><br>

-	**SSL Certificate** -  Select *Custom SSL Certificate* and, from the drop down and select the certificate you previously requested.

-	**Default Root Object:** Type ‘index.html’ as this is the file it will search to cache and provide to our users. <br><br>
![CDN](/img/blogwebsite/cdn3.jpg "CloudFront Distribution") <br><br>
-	Click **Create Distribution**. This should take about 15-20min for completion.
-	Copy the *Domain Name* on *Distribution Dashboard* E.g d2g3ds7l4phvye.cloudfront.net in my case. We need this to make changes to our A record in DNS to point to this distribution.
-	While we wait for this distribution to be deployed, lets head over to Route53 – DNS Service of AWS.

##### Step 3: DNS Changes to point to our new CloudFront Distribution.

The final step is to configure the DNS record for the redirecting host name which must point to the CloudFront distribution domain name. So when a user visits the domain, they are redirected to the target URL (which is our S3 Static Bucket).<br>

-	We will be using AWS Route53 as our DNS, so go to Route53 from AWS Console
-	You should be having a public hosted zone already configured for our website. If not, please refer to my [blog](./posts/2019-11-06-BuildStaticWebsite.markdown){:target="blank"} for configuration steps.
-	In my previous article, we had created 2 A record sets pointing to our S3 buckets. As CloudFront is going to serve our content, we no longer need this A record.
-	What we need is 2 A Records (naked and www) to point to our new distribution E.g *d2g3ds7l4phvye.cloudfront.net*
-	You need to set Alias to Yes and dropdown should show the distribution or you can just paste the distribution (just like I did).

![CDN](/img/blogwebsite/cdn4.jpg "CloudFront Distribution") <br><br>

-	Ensure you have 2 new A records as shown below. <br><br>
![CDN](/img/blogwebsite/cdn5.jpg "CloudFront Distribution") <br><br>


##### STEP 4: Deploy your site using S3_website

We need to note that files in the CloudFront CDN are not meant to change and so we had defined cache behaviour during creation of distribution. However, if we were to change the content of our website, the new content needs to be deployed to the CDN's edge servers too for it to be served to our users whenever such new changes occur.<br>

So we have to invalidate the content on the CDN for which we use CloudFront's invalidations feature and a easy way to do this is using the `s3_website` utility.

We discussed s3_website in my previous [blog](./posts/2019-11-06-BuildStaticWebsite.markdown){:target="blank"} on **S3_website utility** so i wont go over it again.

But I will show you the changes required to our *s3_website.yml* file for our Cloudfront invalidations to take place whenever we push or upload the new content.

Do the following changes to **s3_website.yml**

`cloudfront_distribution_id: YOUR_DISTRIBUTION_ID`
`cloudfront_wildcard_invalidation: true` <br>
`cloudfront_invalidate_root: true`

***

##### Troubleshooting tips

- If you don't see your ACM Certificate in Custom SSL drop down, change the region to N-Virginia when you create certificates in ACM
- During creation of distribution in CloudFront, if adding CNAMEs gives an error, try following the steps given [Here](https://aws.amazon.com/premiumsupport/knowledge-center/resolve-cnamealreadyexists-error/)
- Incase of redirection from fqdn bucket to www is not working due to origin path issues, try using redirection rule. For e.g I had an issue with www domain as it appended index folder to root path. So I used redirection of index/index.html to root in s3_website.yml under *redirect* section as `index/index.html: /`

<p> Thats it.. Open a browser and type your domain name with or without www to see your landing page.

Congratulations!! Your website is now secure as you should see a lock in your website url in the browser  </p>


***
