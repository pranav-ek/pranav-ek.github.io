---
layout: post
title: Backup Files in EC2 Instance to Amazon S3
image: "/content/images/2016/04/cloud-backup.png"
date: '2013-11-26 09:10:08'
---

Every day businesses are creating and aggregating data they need to protect from loss. The importance of data can help us determine if they should be backed up.

Important data may deleted accidentally, can become corrupt, a virus or even a natural disaster can put the data at risk, and that is why it is important to have a backup and disaster recovery plan. Without one, there is nothing to fall back on.

One of the project that I work on generates [TSV](//en.wikipedia.org/wiki/Tab-separated_values) files on a day-to-day basis. These files are used to dispatch bulk emails of recommended products as a part of the cross-channel marketing strategy and very important in determining the [ROI](//en.wikipedia.org/wiki/Return_on_investment). So these files are backed up to the Amazon S3 buckets located at different cloud center regions of [Amazon Web Services(AWS)](//aws.amazon.com).

#### 1. Overview

A [Spring Batch](//projects.spring.io/spring-batch/) job generates the files and saves it on the root volume -of a Ubuntu [EC2](//aws.amazon.com/ec2) instance - backed by the [EBS](//aws.amazon.com/ebs). These files are then backed up to S3 using a [Cron](//en.wikipedia.org/wiki/Cron) job.

The tool that is used for interacting with the S3 is [s3cmd](http://s3tools.org/s3cmd). This command line utility will synchronize data in the EBS with the S3.
#### 2. Install s3cmd
Use apt-get package manager to install s3cmd on the Ubuntu server.

{% highlight ruby %}
sudo apt-get install s3cmd
{% endhighlight %}

#### 3. Configure s3cmd for managing data on S3
Get the access key, secret key from your AWS account prior to this step.

{% highlight ruby %}
s3cmd --configure
{% endhighlight %}

#### 4. Create a S3 [bucket](//docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html) 
S3 bucket can be also created from the web console as well as AWS SDK.

{% highlight ruby %}
s3cmd mb s3://bucket_name
{% endhighlight %}

#### 5. Synchronize files
{% highlight ruby %}
s3cmd sync -r ~/directory_to_backup/ s3://bucket_name/
{% endhighlight %}

#### 6. Write a shell script to automate the  synchronization process
*backup_sync_ec2_s3.sh*
{% highlight ruby %}
#!/bin/bash
s3cmd sync -r ~/backup_directory/ s3://bucket_name/
{% endhighlight %}

#### 7. Edit crontab file
{% highlight ruby %}
crontab -e
{% endhighlight %}

#### 8. Schedule  the script
Add <a href="//help.ubuntu.com/community/CronHowto#Crontab_Sections" target="_blank">cronjob</a> to trigger automated backup everyday at 23:59 hrs
{% highlight ruby %}
59 23 * * * sh ~/backup_sync_ec2_s3.sh
{% endhighlight %}

#### 9. Conclusion 
Mission-critical data should be backed up. Off-site file backup of the data from EC2 to S3 can be done using a command line tool named s3cmd. Automating that process using Cron job makes sure the data are synchronized on daily basis.
 
Check the [s3cmd usage](http://s3tools.org/usage) link to learn more about the available options and commands 
