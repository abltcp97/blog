---
title: "S3 Me Now: How I put my Resume in the Cloud"
date: 2025-03-23 18:00 -0800
description: "How I built a secure, automated, static site deployment pipeline using Terraform, GitHub Actions, and AWS."
categories: [Cloud Security, Cloud Resume Challenge]
tags: [terraform, aws, cloudfront, github-actions, s3, security, cloud-security, iac, guardduty, iam]
---

![AWS Architecture Diagram](/images/cloud-resume-challenge/crc-banner.png)

## 👋 Welcome back!

I apologize for the wait between blog posts. I have developed an interest in the Cloud and AWS during one of my research phases of Cybersecurity which lead me to find the Cloud Resume Challenge! I found it appealing and sparked a new interest in coding infrastructure I never knew I had! Without further ado let's jump in and I hope you enjoy my blog post!

The [Cloud Resume Challenge](https://cloudresumechallenge.dev) was created by Forrest Brazeal, it’s a hands-on project that pushes you to build a resume website using a full serverless stack on AWS. No drag-and-drop builders here—just code, cloud services, and architecture.

I took the challenge and decided to treat it like a production build. I wanted it to be secure, automated, and something I could show off to potential employers—not just tell them about.

What I used is **Terraform**, **GitHub Actions**, and a bunch of **AWS services** to build a public, serverless cloud portfolio from the ground up. And I wrote this blog because I like to talk about the decisions, the confusion, the fixes, and what I learned along the way.


## 🧠 Why Bother?

I’m into **cloud security**, **DevSecOps**, and building things that run in the real world. This challenge gave me a space to:

- Prove I can ship something public and production-grade  
- Get deeper with **Infrastructure as Code** and CI/CD  
- Build a portfolio that’s more than just a GitHub repo (Was Guilty of this)  
- Practice secure, serverless-first design in AWS  

For anyone looking to stand out in cloud or DevOps, this is the kind of project that does the talking for you.

---

## 📐 How It’s Built (The Architecture)

![AWS Architecture Diagram](/images/cloud-resume-challenge/aws-diagram.svg)

The setup is 100% serverless, with security and automation baked in from the start.

- **Frontend**: Static HTML/CSS/JS on **S3**, served via **CloudFront**, secured with **ACM** (HTTPS)
- **Backend**: Public **API Gateway** calls a **Python Lambda** that tracks visitors in **DynamoDB**
- **IaC**: Everything is built and maintained in **Terraform**
- **CI/CD**: GitHub Actions deploy the site and apply infra changes  
- **Security**: IAM with least privilege, **GuardDuty**, and tight public access controls

It’s simple, clean, and reproducible.

## 🧱 How I Built It

### 1. 🔨 Setting Up the Infrastructure with Terraform

I started with two S3 buckets—one for my resume site, one for the Terraform state. DynamoDB handled state locking and visitor count tracking. I also spun up:

- A Python-based Lambda + API Gateway  
- CloudFront with ACM for HTTPS  
- Cloudflare for DNS + domain (`portfolio.aalamillo.com`)

```hcl
resource "aws_s3_bucket" "portfolio_site" {
  bucket = "portfolio-site-aalamillo"
  website { index_document = "index.html" }
}
```
> CloudFront is picky. A simple  mismatch in origin IDs gave me a NoSuchOrigin error. Which took me a while to figure out.  
{: .prompt-tip} 

### 2. ⚙️ CI/CD with GitHub Actions

I split my project into two repos:
- `portfolio-deploy`: Infra managed by Terraform
- `portfolio-site`: The frontend code

![Github Actions](/images/cloud-resume-challenge/github-actions.png)

I can't tell you how many times I made commits and then realized I forgot change another part of the code. I need to check twice commit once.

GitHub Actions handles deployment on every push. It syncs to S3 and invalidates the CloudFront cache.

```yaml
- run: aws s3 sync . s3://portfolio-site-aalamillo --delete
- run: aws cloudfront create-invalidation --distribution-id XYZ --paths "/*"
```
> Handling secrets between terraform and Github Actions? Make sure the outputs are passed securely and scoped properly!  
{: .prompt-tip} 

### 3. 🐍 The Lambda: Visitor Counter
This function is simple but unbreakable. It ultizies ```boto3``` to connect to DynamoDB and increments the visitor count.

```python
def lambda_handler(event, context):
    table = boto3.resource("dynamodb").Table(os.environ["TABLE_NAME"])
    new_count = get_and_increment_visits(table)
    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*"
        },
        "body": str(new_count)
    }
```
> Got a CORS error? Remember to handle the headers in the lambda itself-don't assume the API Gateway will handle it all. (I learned the hard way.)  
{: .prompt-tip} 

### 🧪 Debugging, Testing, and Hard Lessons
Unfortuntely, testing couldn't just be a checkbox I tick off. I used pytest to write unit tests for the Lambda logic, with mocked DynamoDB responses. Each commit the tests are ran on the CI Pipeline.

Tricky Bugs I ran into:
- I had an early ```return``` statement that made the rest of the code afterwards unreachable in lambda.
- Had some S3 access erros that needed public access blocks lifted and properly managed in terraform.
- Encountered a race condition with terraform and apply-order that required me to use a ```depends_on``` blocks.

> Debugging infrastructure code is tedious but awarding because you learn ***a lot.***  
{: .prompt-tip} 

### ✨ What I Got Out of It
- Site is live: www.aalamillo.com
- Vistor counter works like a charm!
- Infra is modular, secure, and 100% in code.
- Blog hosted separately at blog.aalamillo.com

And I've now got a real-world portfolio I can show- not just talk about.

![Final Portfolio Screenshot](/images/cloud-resume-challenge/final-portfolio.png)

### 🧠 Final Thoughts
This project wasn't just about getting my resume online, it was about proving I can build, secure, and automate a full-stack cloud application. It pushed me to adopt best practices, learn better debugging skills, and gain a deeper understanding of AWS. Now, armed with actual experience I might go do the AWS Certificates!

If you're starting out in cloud or DevSecOps-or just want a project that shows what you can actually do- I'd highly recommend taking this challenge on.

Let's chat! Find me on [Linkedin](https://www.linkedin.com/in/abel-alamillo/) or check out more of my work on [Github](https://github.com/abltcp97).  

