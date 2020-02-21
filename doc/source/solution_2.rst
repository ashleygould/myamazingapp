.. _solution_2:

Solution 2 - Immutable EC2 with autoscaling
===========================================

.. toctree::
   :titlesonly:
   :hidden:

   solution_2_details


This scenario is mostly similar to solution 1 with a few notable
differences.  EC2 instances are now part of an AutoScaling Group. 
Application and system logs are written to the EFS file share.

EC2 instances are ephemeral.  Any changes to EC2 state, whether for
system configuration, patching, or application code updates, are
executed by the Autoscaling service which performs a sequential
re-build/re-launch of the EC2 instances.

Such events are controlled by a CodePileline instance and are triggered
whenever a commit is pushed to the appropriate branch of the application's git
repository.  The CodePileline instance can be configured to refresh the Staging
environment, send SNS notification to QA staff, and then wait.  Once QA staff validates the update, they submit approval, and CodePileline continues to refresh
the production environment.  A separate CodePileline instance controls the
development environment.

Puppet agent is no longer used.  Application and system configuration are
applied via userdata scripts at EC2 launch time.  However, existing puppet
manifests can be applied as part of userdata scripts.  External passwords are
retrieved from either SSM Parameter Store or SecretsManager at launch time.

It may be possible to build and deploy this entire stack using
`AWS Elastic BeanStalk for Rails`_.


.. :ref:`solution_2_details`

.. _`AWS Elastic BeanStalk for Rails`: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ruby-rails-tutorial.html

Operational Considerations
**************************

- all AWS resources defined and deployed from code (CloudFormation).
- migrating the app to another environment is much easier now.
- no direct shell access to EC2 instances.
- deployment of infrastructure can be done by either central ops or the
  application team, depending on our support model.
- app team requires read/write access to an AWS sandbox account to
  develop and test infrastructure code.
- app team must become proficient with CloudFormation and some AWS
  services - EC2, AutoScaling, ALB, IAM, SSM, CodePipeline.


