---
layout: post
title: "AWS Overview"
description: >
  Overview of AWS main elements.
categories: cloud
tags: [aws]
comments: true
image: /assets/img/blog/aws/aws-overview.png
---
> AWS is a set of web services provided by Amazon.
In this page, the overview of AWS will be covered.
{:.lead}

- Table of Contents
{:toc}

## Regions and AZs

Regions are isolated geographic areas, which brings the best fault tolerance and stability.
When viewing resources in AWS console, you can only see resources related to your current region. There is no way to share resources between regions - those are not automatically replicated, and area isolated from each other.
In order to launch an instance, it is required that an [AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) (Amazon Machine Image) in the same region is selected - so you cannot select an AMI from another region. If the AMI in is a different region, it can be [copied](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html) to the current region, and then the instance of that particular AMI can be launched.

## Instances and AMIs

An Amazon Machine Image (AMI) is a template to create instances. That template cointans elements such as operational system, application server, and applications. From an AMI, you can launch an instance. A instance is a copy of an AMI running as a virtual server in the cloud.

- You can also launch multiple instances of an AMI
- If an instance fails, you can launch a new instance from the AMI

More information related to Instances and AMIs can be found [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instances-and-amis.html), and AMI specific details can be found [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html).

## References

- [AWS Cloud Practitioner Essentials ](https://aws.amazon.com/pt/training/digital/aws-cloud-practitioner-essentials/)
- [AWS Technical Essentials (free)](https://explore.skillbuilder.aws/learn/course/external/view/elearning/1851/aws-technical-essentials)
- [AWS basic concepts](https://aws.amazon.com/pt/getting-started/fundamentals-core-concepts/)
- [Stephane Maarek's Ultimate AWS Certified Solutions Architect Associate SAA-C03 @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4704&courseLabel=24774&q=stephane+maarek&sort=relevance&src=sac)
- [AWS Certified Solutions Architect - Associate 2020 @ Udemy](https://www.udemy.com/course/draft/362328/learn/lecture/13885822?start=15#overview)
- [AWS Certified Solutions Architect Associate Training SAA-C03](https://www.udemy.com/course/aws-certified-solutions-architect-associate-hands-on/learn/lecture/28616728?start=15#overview)
- [Architecting on AWS](https://aws.amazon.com/pt/training/classroom/architecting-on-aws/?ep=sec&sec=assoc_saa)