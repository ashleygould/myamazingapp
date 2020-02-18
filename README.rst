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

- SSL
- Codebased Infrastructure
- Immutable Infrastructure
- CI/CD
- Integaration of Application Code with Infrastructure APIs




I came up with 5 alternative solutions. All solutions assume AWS as cloud provider:

1. Managed EC2 with Puppet
2. Immutable EC2 with autoscaling
3. Docker in ECS with EFS
4. Docker in Fargate with S3
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

This scenario is mostly similar to the above with two primary
differences: EC2 instances are now part of an Autoscaling Group, and
both application and system logs are exported to an S3 bucket via
CloudWatch.  Puppet is no longer used, as application and system
configuration are applied via userdata scriptes at EC2 launch time.
External passwords are retrieved from either SSM Parameter Store or
SecretsManager at launch time.  EC2 instances are ephemeral.  Any
changes to EC2 state, whether for system configuration, patching, or
application code updates, are executed by trigging the Autoscaling
service to perform a controlled re-launch of the EC2 instances.


<link to detailed specification>

- infrastructure is completely codebased 
- deployment of infrastructure can be done by either central ops or the
  application team.





4. Docker in Fargate with S3
----------------------------

I propose to build out this application infrastructure using Docker 
containers hosted in AWS Fargate Elastic Container Service.  Fargate ECS
provides HA and horizontal scaling of docker containers.  The solution
is entirely codebased, and all infrastructure is immutable - no servers to
maintain.

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


.
