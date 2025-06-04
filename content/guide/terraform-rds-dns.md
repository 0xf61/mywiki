---
title: Terraform RDS module with DNS setup
draft: false
date: 2025-06-04
tags:
  - terraform
  - aws
---

I prefer not to manage databases myself. Hosting it on your own computer might be cheaper, but I don't want to constantly worry about backups, database upgrades, and maintenance.

AWS offers managed databases called RDS. I mostly use Postgres, which works well for many situations. (And it's generally a good idea that if you don't really need a NoSQL database, then don't use one).

If I start an RDS Postgres, I can access it if I have set up the correct network connections. (And you shouldn't make your database public unless it's for practice and doesn't contain sensitive information.)

RDS gives a hostname in this format: `mydb.123456789012.us-east-1.rds.amazonaws.com`. This is fine for most purposes, but sometimes you might want it to be easy to remember, or you want to switch database instances without causing downtime. A simple solution is to set up a DNS record, and it would be great if you could set it all up when creating the database!

Here is the Terraform code (and yes, this also works for a copy of the database):

```hcl
resource "aws_db_instance" "db" {
  engine            = var.engine
  allocated_storage = var.allocated_storage
  engine_version    = var.engine_version
  instance_class    = var.instance_class

  identifier = var.identifier
  db_name    = var.db_name
  username   = var.username
  password   = var.password

  replicate_source_db = var.source_db_id # for replica

  parameter_group_name = var.parameter_group_name

  performance_insights_enabled    = true
  performance_insights_kms_key_id = var.performance_insights_kms_key_id

  deletion_protection = var.deletion_protection

  ### networking
  vpc_security_group_ids = var.vpc_security_group_ids
  db_subnet_group_name   = var.db_subnet_group_name

  ### backup (in UTC)
  maintenance_window      = var.maintenance_window
  backup_window           = var.backup_window
  backup_retention_period = var.backup_window != null ? 3 : 0 # Backups are required in order to create a replica
  skip_final_snapshot     = var.skip_final_snapshot

  apply_immediately = var.apply_immediately
}

resource "aws_route53_record" "db" {
  zone_id = var.zone_id
  name    = var.dns_name
  type    = "CNAME"
  ttl     = 5
  records = [aws_db_instance.db.address]
}

```

[Reference](https://karnwong.me/posts/2023/03/terraform-rds-module-with-dns-setup/)
