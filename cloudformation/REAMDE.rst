Cloudformation templates
========================

Prerequisites
-------------

- Route53 HostedZone
- Signed SSL certificate (what is the aws resource name atgain?) 

VPC
---

This template generates an EC2 VPC with public and private subnets in two
AvailabilityZones.  It includes internet and nat gateways, routes and routetables,
and an S3 endpoint.  It exports ARNs for the vpc, subnets, and S3 endpoint.


Fargate
-------

This template generates a fargate ECS cluster, and ApplicationLoadBalancer, an
ERS repository, and ECS service definition all configured to host the Rails
container instance.  It includes VPC security groups, S3 bucket for application
logging, S3 bucket for file uploads.  It also sets up a Route53 resource record
and SSL certificate configuration for the application Domainname.  It exports
ARNs for the ERS repository, and the application URL.


CodePipeline
------------










