MyAmazingApp
============

This is my reponse to job interview excercise

Excercise description
---------------------

::

  Please describe how you would address the requirements below.  Please
  submit this one day before the interview. Your responses will be
  shared with the interview team.
  
  Requirements:
  
  We have a Rails application that runs in a highly available
  environment. Application deployments are automated.  The application
  support file uploads and downloads which can vary in size (1 mB - 6
  Gib) and are short-lived (remain on the server less than 2 weeks and
  should be removed).   The application is supported by a database
  backend. The application has a rake task that is run weekly to provide
  user stats. It needs to be accessible at the URL
  myamazingapp.cdlib.org.
  
  Describe how you would build infrastructure that would support this
  application to be deployed in a cloud environment.
  
  
  Considerations:
  
  What tools would you use to build this infrastructure?
  
  How you would go about testing your approach, what packages would need
  to be installed to support the application.
  
  How might you support configuration parameters that contain passwords
  to external services?
  
  How would you ensure that you could easily migrate this application to
  another environment or host?


-----


Some Additional Considerations
------------------------------

- Infrastructure as code
- Immutable Infrastructure
- CI/CD
- SSL/TLS encryption
- Expertese and comfort level of developers
- Integaration of AWS API calls within application code 
- Resource Tagging
- Orchestration




I came up with five solutions. All solutions assume AWS as cloud
provider.  Taken in sequence, this set of solutions describes a possible
evolution path for application hosting in AWS:

1. Managed EC2 with Puppet
2. Immutable EC2 with autoscaling
3. Docker in standard ECS
4. Docker in Fargate ECS with S3
5. Lambda with S3, ApiGateway, and CloudFront


1. Managed EC2 with Puppet
--------------------------

In this scenario, the application is hosted on two EC2 instances each
deployed into a separate Availabitly Zone for high availability.  DB
backend is hosted on two RDS instances (again, in separate AZs) one of
which is a read-only replica.  A single EFS instance is mounted by
both EC2 instances to provide shared storage of file uploads.  An
Application Load Balancer instance distributes incoming traffic among
the EC2 instances and handles SSL off-loading.  Application specific
system configuration (including external passwords) is managed via
Puppet.

<link to detailed specification>


- central ops team builds and manages all aspects of AWS infrastructure
- infrastructure is probably not codebased
- EC2 instances require ongoing system administration: patching, user
  access, etc. (non-immutability)
- app team requires direct shell access to EC2 instances to perform
  application support tasks
- app team does not require access to AWS APIs


2. Immutable EC2 with autoscaling
---------------------------------

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

<link to detailed specification>

.. https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ruby-rails-tutorial.html

- all AWS resources defined and deployed from code (CloudFormation).
- deployment of infrastructure can be done by either central ops or the
  application team, depending on our support model.
- no direct shell access to EC2 instances.
- app team requires read/write access to an AWS sandbox account to
  develop and test infrastructure code.
- app team requires read access to CloudWatch logs.



3. Docker in ECS - EC2 Launch Type
----------------------------------

This scenario exchanges the EC2 instances in the AutoScaling Group with
ECS optimized EC2 instances to form an ECS Cluster.  Since the ECS
Cluster is built using the EC2 Launch Type, the application can still
make use of the EFS volume for file uploads.   

ECS Task and Service resources are defined to run the app as a Docker
instance on the ECS Cluster.  Docker images built from application code
are posted to an ECR repository instance.  The ECS Service sources this
repository to re-deploy updated images to the ECS Cluster.

Upon pushing a new commit into the appropriate branch of the
application's git repository, the CodePipeline instance now runs a build
stage which builds a new Docker image and pushes it to the ECR
repository.  It then notifies the ECS Service to re-start the running
Docker instance using the updated image.

<link to detailed specification>

- docker provides a very convenient develpment workflow
- app team members must become profficient with Docker


.. https://aws.amazon.com/blogs/compute/using-amazon-efs-to-persist-data-from-amazon-ecs-containers/

.. https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-ecs-ec2.html

.. https://aws.amazon.com/premiumsupport/knowledge-center/ecs-create-docker-volume-efs/

.. https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_efs.html




4. Docker in Fargate with S3
----------------------------

In this scenario the ECS Cluster is defined using the Fargate Launch
Type.  This greatly simplifies AWS Resource coding and cluster
maintenance.  However, it precludes the use of EFS.  Instead, the
application now uses AWS API calls (AWS SDK for Ruby) to post uploaded
files to an S3 bucket.

<link to detailed specification>

.. https://aws.amazon.com/sdk-for-ruby/
