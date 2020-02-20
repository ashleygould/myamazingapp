.. _docker_in_fargate:

Technical Details - Solution 4
==============================


Docker in Fargate with S3
-------------------------

In this scenario the ECS Cluster is defined using the Fargate Launch
Type.  This greatly simplifies AWS Resource coding and cluster
maintenance.  However, it precludes the use of EFS.  Instead, the
application now uses AWS API calls (AWS SDK for Ruby) to post uploaded
files to an S3 bucket.

<link to detailed specification>

.. https://aws.amazon.com/sdk-for-ruby/

Features
--------

- Application and infrastructure code hosted on Github.
- Infratructure provissioned using Cloudformation.
- Automated application deployments via AWS CodePileline, deployments
  triggered by git commit.
- Docker images hosted on AWS ERS.
- Docker instances run in an ECS cluster spanning two AvailabilityZones.
- DB backend hosted on AWS RDS.
- File uploadeds hosted by AWS S3.
- Application logging via CloudWatch with logs residing on S3.



