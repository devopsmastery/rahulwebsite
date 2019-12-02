---
layout: post
title:  "How to create a website and launch it under 2 hours"
subtitle: "Using AWS S3 & ROUTE53 to host website free of cost"
date:   2019-11-06 11:17:44 +0530
background: '/img/blogwebsite/website.jpg'
categories: blog update
---
<p>Ever wanted to have a website for your blog or simple website for your business or just a personal website to showcase your work and profile?</p>

*Static websites* work best in these cases and are very simple to manage as they dont have a backend database or server side scripting.

We will be using *Jekyll* which is a static website generator and has many free sample/templates available on the net. <br>

Jekyll uses Ruby which you need to install. [Click here for installing ruby](https://www.ruby-lang.org/en/documentation/installation/) <br>

So Lets get Started !!

***

##### Pre-Requisites
-	Basic knowledge of HTML/CSS – we will be using Jekyll for static website pages
-	Basic Knowledge of AWS Services such as S3 and Route53.
- Checkout ***[AWS Technical Essentials](http://www.techtutorr.com/aws-technical-essentials/){:target="_blank"}*** course for more details on AWS fundamentals.
-	Editor – I will be using **[Atom](https://atom.io/)** to edit HTML/Markdown pages.
-	Git (optional) – Used as code repository for our website <br>

***

##### What we will be using
- **Registered domain name** from provider like GoDaddy or use AWS Route53 DNS.
- **S3 Bucket** to host our static website
- **Route53** - Amazons DNS service
- **S3_website utility** for pushing our local website to production

***

##### Step 1 - Purchasing Domain
Purchase a Domain from any domain provider such as GoDaddy or Route53. I purchased mine from GoDaddy.<br>

I will be using ***awscloudtrainings.com*** domain in this article as example.

##### Step 2 - Create a static website with Jekyll

I have used a freely available Jekyll template to create my blog website – rahulgujar.com
-	Download the zip file or clone it from **[github](https://github.com/BlackrockDigital/startbootstrap-clean-blog){:target="_blank"}** of a simple template -
-	Now go ahead and install Jekyll with the steps provided on **[Jekyll](https://jekyllrb.com/docs/installation/){:target="_blank"}** site
-	Update Jekyll with command – `bundle update Jekyll`
-	On Local M/c: Create a new directory where you would like your local site to be built.
-	Let’s generate a new website in our new directory. **cd \<newdirectory\>**
    - Run `jekyll new <mywebsite.com>` (use the same name as domain you have purchased)
    - Copy content of sample website is available on **[github](https://github.com/BlackrockDigital/startbootstrap-clean-blog){:target="_blank"}** into our website folder.
    - Edit *Gemfile* and add `gem "jekyll-theme-clean-blog"`
    - Edit *_config.yml* and change **title, email id, description and author**
    - *Baseurl* – ``“”`` (empty double quotes)
    - Build and Serve the site locally using command –<br>
    `bundle exec jekyll serve`
    - Check if site is working locally – `http://127.0.0.1:4000/`
- At this point the site should be working locally and we are all set for next step on AWS configurations
<br><br>

##### Step 3: Create S3 Bucket that will have our static website code

-	Login to AWS console and jump to S3 service
-	We need to create 2 separate buckets -
    - *awscloudtrainings.com* (naked domain) and
    - *www.awscloudtrainings.com*.
- Our naked domain bucket will be hosting the static website with www bucket configured to redirect to naked domain bucket
-	Click *Create Bucket* and select region closest to you.<br><br>
![bucketcreate](/img/blogwebsite/createbucket1.jpg "Create Bucket")
![bucketcreate2](/img/blogwebsite/createbucket2.jpg "Create Bucket")
<br><br><br>
- Give Public access to bucket by unchecking “Block All public access”
<br><br>
![bucketcreate3](/img/blogwebsite/createbucket3.jpg "Public Bucket")<br><br>
- Under Properties -> Select Static Website Hosting<br><br>
![bucketcreate3](/img/blogwebsite/createbucket4.jpg "Static Bucket")<br><br>
- Add the following Bucket policy under *Permissions* to make all objects public as we want the site to be publicly available.<br>

      {
          "Version": "2012-10-17",
           "Statement": [  
           {  
             "Sid": "PublicReadGetObject",  
             "Effect": "Allow",
             "Principal": "*",  
             "Action": "s3:GetObject",  
             "Resource": "arn:aws:s3:::awscloudtrainings.com/*"  
           }  
          ]  
      }  


We are done with our main bucket which would host our static website. <br><br>
- Now create another bucket for our **www** domain with similar setting and bucket policy as our naked domain bucket except for this small change on Static Website hosting where we *redirect requests to target bucket* of our naked domain bucket we create earlier. <br><br>

![bucketcreate5](/img/blogwebsite/createbucket5.jpg "Redirect Bucket")<br><br>

- Also note that the bucket policy should have the arn corresponding to our www bucket as below <br><br>
![bucketcreate6](/img/blogwebsite/createbucket6.jpg "Redirect Bucket Policy")<br><br>

##### STEP 4: Deploy your site from LOCAL to S3 bucket

The easiest way is to dump everything in the **\_site** folder into our main (naked domain) bucket. You can do this via AWS Console.

A better and also effective way is to create a pipeline which automatically detects changed files in local and pushes them to our static website bucket.
We will use a simple utility called **[s3_website](https://github.com/laurilehmijoki/s3_website)**

-	Firstly, download the zip or git clone `s3_website` github repo locally
-	Install it with command- `gem install s3_website`
-	Run `s3_website cfg create`. This generates a configuration file called s3_website.yml
-	Update *s3_website.yml* with your AWS credentials <br><br>
    - `s3_id = AWS_ACCESS_KEY_ID`
    - `s3_secret = AWS_SECRET_ACCESS_KEY`
    - `s3_bucket = awscloudtrainings.com`
    - `s3_endpoint = <your_aws_region>` e.g ap-south-1 for Mumbai

  **DO NOT UPLOAD s3_website.yml to GIT or S3 Buckets as it has the credentials.** A safer way is to create a .env file into your root where you can define all environment variables. Please refer to s3_website documentation on github for more details.
  - Run `s3_website cfg apply` to apply configuration changes.
  - Run `s3_website push --site=<path-to-your-local website>` to push your website to S3 <br><br> (E.g `s3_website push --site=../awscloudtrainings.com/_site/.`) I have created s3_website in the website folder.<br><br>
  ![s3website](/img/blogwebsite/s3website.jpg "s3_website command")<br><br>

That’s It!!! Your code files are now uploaded to S3 bucket which you can view in console or through AWS CLI S3 commands on your terminal.

You can see your website by clicking the S3 endpoint url. <br><br>
![s3website](/img/blogwebsite/s3website2.jpg "blog website")<br><br>

However, we want our users to come to our website using the domain name and not S3 endpoint url. So we need to do the DNS changes. <br><br>

##### STEP 5: ROUTE53 DNS Changes and NS record changes
This will be our final step in making our website live in production.
- Sign in to your AWS Console and go to “Route 53”
- Jump to Hosted Zones and create a Public hosted zone (as it’s a public website) to add our new.
<br><br>
![s3website](/img/blogwebsite/route531.jpg "hosted zone")<br><br>

- Copy the NS (Name Servers) over to your domain registrars admin for ‘Name Servers’. Once this is done, Route 53 will handle the configuration of the domain.
- Click “Create Record Set” and create an `A record` for our naked domain.
- Set Alias target to Yes and select the S3 bucket from dropdown.
![s3website](/img/blogwebsite/route532.jpg "blog website")<br><br>

- Create another Record Set for our `www` bucket.
![s3website](/img/blogwebsite/route533.jpg "blog website")<br><br>

<p> Thats it.. Open a browser and type your domain name with or without www to see your landing page.

Congratulations!! Your website is live... Yiippeee.   </p>

##### What next?
Although we have a live website, there are few things you may want to consider as further enhancements<br>
- Google Ranking factors such as secured website with HTTPS
- Improve responsiveness of our website for serving global customers <br>

I will shortly write another blog on how to secure website with AWS Certificate Manager (ACM) which is pretty much free and also speed up our website with AWS CloudFront. <br>

That's it for now folks!!


***
