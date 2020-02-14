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

Docker Solution
---------------

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



