+++
title = "DevOps principles introduction"
description = "Introduction to development operations principles."
date = 2023-12-10T08:00:00+00:00
draft = false
weight = 10
sort_by = "weight"
template = "docs/page.html"
slug = "introduction"

[extra]
lead = ''
toc = true
top = false
+++

Development operations aka DevOps means a lot of things to a lot of different people. I've met people from all different walks of life, businesses of various size, and different industries and the role of a DevOps engineer is very fluid. At it's core it's focused on the software side of the business from the perspective of automation, efficiency, continuous improvement, and rapid delivery of software. It's a niche layer between software and infrastructure that bridges the gap, very necessary in cloud environments, and funnily enough is it's own software path....you manage software and infrastructure with other types of software.

In most small to medium businesses, DevOps engineers wear many hats such as:

* release engineer
* CI/CD pipeline manager
* Infrastructure and system operations (on-call support)
* Security operations
* Database administrator (sometimes)

I think of DevOps as the glue for cloud operations and software practices and is a demanding role which requires many skillsets across the tech stack. Here are some important concepts which are crucial for the role:

## Release Engineering (RE)

RE to me is assisting with packaging the code to prepare it for deployment on cloud. Most software developers are simply writing the code to add features to the product they support without too much of thought of getting it running in production. In some orgs the devs do maintain that but most do not. Sometimes releases are more complicated than simply deploying a new version of the software, it will typically include things such as:

* database schema modifications
* post deployment tasks
* configuration adjustments
* infrastructure availability (standing up systems)
* feature flagging

Another area in this is optimizing the artifact which goes into production, slimming down container image sizes, shipping lean artifacts. DevOps teams are often involved with all of the above to bridge the gap between the software and the cloud infrastructure to ensure releases of new software can happen often, go smoothly, and have a path to revert in case of emergency.

## Site Reliability Engineering (SRE)

SRE to me is a culmination of all the areas involved here. The term was originally coined by Google and is essentially "solving operations problems with software through automation and careful design". SREs are typically very senior people and the foremost experts in their craft. An SRE is typically someone well versed with all areas of: 

* software development
* complex system design/architecture
* system operations
* business continuity and requirements
  * service level agreements
  * disaster recovery
  * recovery objectives
  * error budgets

While much of the role goes into planning and design, it's also extremely important in changes after the fact. Any time a system inevitably breaks/fails (outages and incidents) the SRE is there to analyze the root cause and help design fixes to the underlying issue which allowed the failure to happen. SREs gauge the system health and performance on a macro level for the products they support.

## Infrastructure as Code (IaC)

The reality of cloud based systems is that everything has become an API. You don't have hardware to manage, you have Platform-as-a-Service (PaaS) products which need to be maintained. While this can be done in a web interface, such as the AWS Console (we call it "ClickOps"), this becomes untenable for larger orgs where changes need to be peer reviewed and carefully planned. Leaving complex infrastructure changes in the hands of an manual process by individual will eventually lead to mistakes. IaC tools essentially bring infrastructure change processes much like software release processes; code changes, peer request, pipelines for release. This can become a niche aspect where it's yet another codebase to maintain but somewhere between applications and infrastructure and it comes with it's own overhead. Some popular tools include:

* Terraform 
* Cloudformation (AWS only)
* Pulumi
* AWS CDK (AWS only)

## Infrastructure and System Operations (InfraOps, TechOps, SysOps, etc)



## Security Operations (SecOps)

