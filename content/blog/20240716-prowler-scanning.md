+++
title = "Prowler Scanning Automation"
description = "Open source security scanning on AWS with automation and dashboarding."
date = 2024-07-16T11:00:00+00:00
updated = 2024-07-16T11:00:00+00:00
draft = false
template = "blog/page.html"
slug = "prowler-scanning-automation"

[extra]
lead = ""
math = true
toc = true
images = ["/images/20231228-cad.png"]
+++

I run a lot of <a href="https://github.com/prowler-cloud/prowler" target="_blank">Prowler</a> scans (open source security scanner script) for work when I'm meeting with new customers who want to get a sense of their AWS security. Most companies struggle with tightening up their cloud security posture, for good reason, because it's incredibly complex and requires full-time attention. Larger organizations can afford a team of security personnel to do this, but smaller ones have to wear that hat in addition to sysops, software dev, devops, etc.

When customers want to get a consult of their security we run this tool against their AWS environment and have a call to review. Out of the box the tool provides some simple outputs: CSV, JSON, and HTML. The outputs are pretty raw and need a bit of massaging to look presentable. The HTML output, while self-contained in a single file, is pretty rough around the edges. There is also a dashboard, which looks much nicer than the HTML output, but doesn't meet my needs.

In the past some of us have spent time making more presentable outputs, but all of which were tedious, and required a lot of copy and pasting. I've been wanting to refactor our tooling and process around it for years but never had the time. Fortunately, I found a gap between project and sales work and spent a few days doing just that!

## Existing Prowler Dashboard

The open source version has support for a dashboard. You run the scan and then run `prowler dashboard` and you get a decent looking page.

<img src="/images/20240716-dashboard.png" width="100%" />

**However**, this doesn't suit my needs and I have some problems with it. 

1. It runs as a web service which means I can't share an artifact; the customer will need to access a URL to review the data. 
2. I don't want to run a long-term instance for each customer and I definitely don't want to make this publicly accessible; it contains sensitive data and potential vulnerabilities in AWS accounts. 
3. There's no authentication and I don't want to build that.
4. It's built for an administrator who wants to view security posture over time presumably as they remediate findings. I need an interface for one-off scans and have no intention of capturing all findings for different customers in one place; this can't differentiate between different organizations.

I opted to instead build my own dashboard using <a href="https://react.dev/" target="_blank">React</a> that can be exported as a standalone artifact I can provide so customers can open it locally and share internally.


## Solution - Backend

I implemented a Python based <a href="https://github.com/lfglance/prowler-scanner" target="_blank">CDK stack</a> which sets up the below infrastructure in my AWS account. 

It's fairly straightforward. When the stack is up, I have a script to generate a Cloudformation template which contains cross-account permissions for the IAM role deployed in the stack. That template is distributed to other AWS accounts you want to scan. 

Those stacks will have outputs such as `role_arn` (the role to assume), `external_id` (a random string defined by whoever launches the stack - avoids the confused deputy problem), and `scan_name` (a helpful identifier for resulting files and logs). This is the architecture on AWS.

<img src="/images/20240716-prowler.png" width="100%" />

When I have a request to do a scan, the customer launches the Cloudformation stack, provides me the outputs, and I can invoke the process by `curl`ing my API Gateway:

```bash
curl -s "https://xxxxxxx.execute-api.us-west-2.amazonaws.com/prod/?external_id=hello_@&role_arn=arn:aws:iam::1111111111:role/CrossAccountRole&scan_name=LanceProwlerTesting"
```

I get something like this:

```json
{
  "message": "Launched EC2 instance to run Prowler scan (LanceProwlerTesting)",
  "instance_profile": "arn:aws:iam::11111111:instance-profile/ProwlerScannerStack-InstanceProfile-zzzzzzzz",
  "prowler_version": "4.2.4",
  "sg_id": "sg-00000000",
  "instance_id": "i-00000000",
  "instance_type": "t2.medium",
  "subnet_id": "subnet-0000000000",
  "vpc_id": "vpc-0000000000",
  "ip_address": "10.0.151.20",
  "connection": "aws ssm start-session --target i-0000000",
  "output_bucket": "prowlerscannerstack-prowlerresultsbucket017fd61c-qqqqqqqq",
  "output_object": "LanceProwlerTesting-1721159874.tar.gz"
}
```

I have a Slack workflow that accepts the 3 fields and issues the HTTP request for me so I don't need to leave Slack **and also means I can have my sales folks run these on their own**.

The API Gateway invokes a Lambda which launches an EC2 instance with an IAM Instance Profile to allow it to assume roles in other accounts with some custom user data (Shell script) which proceeds to run Prowler and generate the outputs. I go about my business and when the scan is complete I get pinged in Slack thanks to a Zapier webhook in place.

## Solution - Frontend

I wanted the output to be very visually appealing with charts and graphs, filterable and interactive text, so I wrote a [React app](https://github.com/lfglance/prowler-ui) which takes the output raw data and formats it nicely.

This was a fun little project and kept my engineer side happy for a few days.

<video controls autoplay loop muted width="100%">
    <source src="https://cdn.fs10xer.dev/20240716-interactive.mp4" type="video/mp4" />
</video>

I know my colors aren't right - severities should be red, orange, and yellow. Will fix.

## v1 - Serverless

If you saw the above and thought, *uhhh, ec2? I thought we do things cloud native?? Why not Lambda and Fargate*? You're not wrong, and in fact, my first iteration was exactly that; a Lambda function which launches an ECS task to do the scan. However, there are many pitfalls with this approach:

I hit quite a few snags with this relatively "simple" solution.

### 1. Role chaining limits with ECS Tasks

<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-role-chaining" target="_blank">Role chaining</a> on AWS limits your `AssumeRole` sessions to a maximum of one hour. For a Prowler scan of a large account with many resources to check, this isn't long enough and would result in the process stopping mid-scan to error with expired temporary credentials.

When a Lambda function launches a Fargate task, it passes a "task role", the IAM Role for the task to assume. This constitutes as your first "hop". When that container then runs the Prowler scan by passing the remote account's IAM cross-account role, that's the second "hop", at which point your session is capped at 1 hour. There are workarounds such as having a background script run on intervals to refresh your session, but that's super clunky.

IT's a pretty big show-stopper that caused me to try other methods such as "fanning out" tasks; one scan per region, one scan per compliance framework, etc. This lead to other problems...

I could have switched to ECS on EC2, but that seemed like more trouble than it was worth if it's EC2 in the end.

### 2. Rate limiting from Docker Hub

ECS Fargate does not cache container images. If I were running an EC2 based cluster it would persist on the container host but that is not the case for Fargate.

Every time a task would spin up it would download the pre-built Prowler image from Docker Hub which took about 30 seconds to a minute each time just to start the container.

Once I implemented the fan-out approach I was spinning up dozens of containers (1 for each 28 regions) as I was iteratively developing. I hit limits pretty quickly and was stuck waiting for them to cool off.

### 3. Clunkiness with container automation

I was using the pre-built Prowler image that the developer maintains on Docker Hub. It runs the scan and passes all of the command line arguments. It worked fine, but I needed other things to happen such as taking the outputs, building the React site, and publishing the artifact to S3.

The solution here would be to write my own Dockerfile and use that container image as my base, then incorporate my own script as the entrypoint. I frankly didn't want to do that as it's just more steps to deal with and debug.

## v2 - EC2

For all of the above reasons I decide to just use an EC2 instance which is way more configurable with less hassle. EC2 instances don't have the same role chaining limitation since the IAM Role they attach is not considered an AssumeRole.

In my Lambda function which launches the instance I wrote a user data shell script which runs when the instance launches. It executes the whole workflow from top to bottom. The end result is the artifact on S3 with a presigned URL posted to my Slack channel with a ping.

The IAM Role it attaches also has permission to terminate EC2 instances that have a tag which I assign to the EC2 instances that launch. This allows the instance to terminate itself. The final step in the workflow is to terminate itself to clean up. 

The code for this can be found <a href="https://github.com/lfglance/prowler-scanner/blob/main/functions/worker.py#L52" target="_blank">here</a>.

## Wrap Up

This is now being used in production and customers can request the scan, sales staff can provide the Cloudformation template and kick off the process. We'll get notified when the report is complete and then we setup the review call. All self service, all automated, all built with readily available open source tooling.

The code can be found on my Github:

* Backend - <a href="https://github.com/lfglance/prowler-scanner" target="_blank">https://github.com/lfglance/prowler-scanner</a>
* Frontend - <a href="https://github.com/lfglance/prowler-ui" target="_blank">https://github.com/lfglance/prowler-ui</a>