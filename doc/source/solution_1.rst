.. _solution_1:

Solution 1 - Managed EC2 with Puppet
====================================

.. toctree::
   :titlesonly:
   :hidden:

   solution_1_details



In this scenario, the application is hosted on two EC2 instances each
deployed into a separate Availability Zone for high availability.  DB
back-end is hosted on two RDS instances (again, in separate AZs) one of
which is a read-only replica.  A single EFS instance is mounted by both
EC2 instances to provide shared storage of file uploads, user stats data
and weekly reports.  An Application Load Balancer instance distributes
incoming traffic among the EC2 instances and handles SSL off-loading.
Application specific system configuration (including external passwords)
is managed via Puppet.

:ref:`solution_1_details`

- central ops team builds and manages all aspects of AWS infrastructure
- infrastructure is probably not code-based
- EC2 instances require ongoing system administration: patching, user
  access, etc. (non-immutability)
- app team requires direct shell access to EC2 instances to perform
  application support tasks
- app team does not require access to AWS APIs


