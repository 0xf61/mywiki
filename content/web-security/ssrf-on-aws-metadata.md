---
title: SSRF on AWS - Cloud Metadata Attacks
draft: false
date: 2025-01-01
tags:
  - owasp
  - ssrf
  - aws
  - cloud_security
  - metadata_service
  - imds
  - server_side_request_forgery
---

Server-Side Request Forgery (SSRF) is when someone tricks a server into making requests to places it shouldn't, like internal or external services. In cloud places like AWS, this can let attackers find secret keys that control AWS stuff.

AWS computers use a thing called Instance Metadata Service (IMDS) to give temporary security keys to apps running inside them. If an app can be tricked into making internal requests, attackers can get to this metadata and steal AWS keys. This can let them do things like see private data or take over the whole account.

Imagine a website that lets you fetch stuff from other websites by typing in a URL:

```http
GET /fetch?url=https://example.com
```

If the website isn't careful about checking the URL, an attacker can make it ask AWS for secret info:

```http
GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

1.  The attacker asks the server to get data from AWS's secret info place (169.254.169.254).
2.  The server tells the attacker what kinds of keys are available for the computer.
3.  The attacker then asks for the actual keys:

```http
GET http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2Role
```

1.  The server gives the attacker the keys:

```json
{
  "AccessKeyId": "AKIAEXAMPLE123",
  "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "Token": "FQoGZXIvYXdzEXAMPLE...",
  "Expiration": "2025-03-31T12:00:00Z"
}
```

1.  Now the attacker has real AWS keys and can:

* List and steal S3 buckets:

```bash
aws s3 ls --access-key AKIAEXAMPLE123 --secret-key wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY --token FQoGZXIvYXdzEXAMPLE...
```

* Create or delete EC2 computers, change what IAM roles can do, or steal data.

Here's how to prevent this:

* Make rules to stop the app from talking to 169.254.169.254.
* In AWS, turn off the old way of getting keys (IMDS v1) and use the new way (IMDSv2), which is safer:

```bash
aws ec2 modify-instance-metadata-options --instance-id i-1234567890abcdef0 --http-endpoint enabled --http-tokens required
```

* Only allow the app to fetch URLs from websites you trust.
* Don't allow requests to go to IP addresses, localhost, or internal services.
* Here's an example of how to check if a URL is okay:

```regex
^(https?:\/\/(www\.)?trusted-domain\.com\/.*)$
```

* Give each EC2 computer only the keys it needs to do its job.
* Block keys from doing important things (like listing all S3 buckets or changing IAM roles) if they don't need to.
* Use VPC Security Groups and NACLs (Network ACLs) to control which computers can talk to each other.
* Make sure EC2 computers can't just ask for anything from internal services.
