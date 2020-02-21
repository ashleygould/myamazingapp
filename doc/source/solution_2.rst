.. _solution_2:

Solution 2 - Immutable EC2 with autoscaling
===========================================

.. toctree::
   :titlesonly:
   :hidden:

   solution_2_details


This scenario is mostly similar to the above with a few notable
differences.  EC2 instances are now part of an Autoscaling Group. 
Both application and system logs are exported to an S3 bucket via
CloudWatch.

EC2 instances are ephemeral.  Any changes to EC2 state, whether for
system configuration, patching, or application code updates, are
executed by the Autoscaling service which performs a sequential
re-build/re-launch of the EC2 instances.  Such events are controlled by
a CodePileline instance and are triggered whenever a commit is pushed to
the appropriate branch of the application's git repository.

Puppet is no longer used.  Application and system configuration are
applied via userdata scripts at EC2 launch time.  External passwords are
retrieved from either SSM Parameter Store or SecretsManager at launch
time.

.. :ref:`solution_2_details`

.. https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ruby-rails-tutorial.html

Operational Considerations
**************************

- all AWS resources defined and deployed from code (CloudFormation).
- no direct shell access to EC2 instances.
- deployment of infrastructure can be done by either central ops or the
  application team, depending on our support model.
- app team requires read/write access to an AWS sandbox account to
  develop and test infrastructure code.
- app team requires read access to CloudWatch logs.
- app team must become proficient with CloudFormation and some AWS
  services - EC2, AutoScaling, ALB, IAM, SSM, CodePipeline.


