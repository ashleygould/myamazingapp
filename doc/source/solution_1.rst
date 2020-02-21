.. _solution_1:

Solution 1 - Managed EC2 with Puppet
====================================

.. toctree::
   :titlesonly:
   :hidden:

   solution_1_details



In this scenario, the application is hosted on two EC2 instances each deployed
into a separate Availability Zone for high availability.  DB back-end is hosted
on two RDS instances (again, in separate AZs) one of which is a standby
replica.  A single EFS file system instance is mounted by both EC2 instances to
provide shared storage of file uploads, user stats data and weekly reports.  An
Application Load Balancer instance distributes incoming traffic among the EC2
instances and handles SSL off-loading.

Application specific system configurations are managed via Puppet.  Passwords
for external services are supplied to puppet manifests via GPG encrypted hiera
data (hiera-eyaml-gpg).

This full stack should be replicated to create a QA or staging environment for
validating application code and for testing AWS infrastructure modifications.
A non-redundant version of this stack with a single EC2 instance should be
used for application and puppet manifest development.  All three stacks can
share the same VPC, but a separate set of security groups should be used for
each environment.

:ref:`solution_1_details`



Operational Considerations
**************************

- central ops team builds and manages all aspects of AWS infrastructure.
- infrastructure is probably not code-based.
- EC2 instances require ongoing system administration: patching, user
  access, etc. (non-immutability).
- depending on thoroughness and detail of puppet manifests, migrating the
  hosted application to another environment could be somewhat difficult or
  fairly easy.  Difficulty is greatly reduced if AWS infrastructure is defined
  in CloudFormation templates.
- app team requires direct shell access to EC2 instances to perform
  application provisioning and support tasks.
- app team does not require access to AWS APIs.


