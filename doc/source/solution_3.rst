.. _solution_3:

Solution 3 - Docker in ECS - EC2 Launch Type
============================================

.. toctree::
   :titlesonly:
   :hidden:

   solution_3_details

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

.. :ref:`solution_3_details`

Operational Considerations
**************************

- docker provides a very convenient development workflow
- fully code-based/immutable infrastructure
- the dockerized application is now very portable
- app team members must become proficient with Docker
- app team members must become proficient with additional AWS
  Services - ECS, ECR, RDS, EFS
- the EC2 instance nodes in the ECS cluster require some ongoing maintenance
- deployment and support of infrastructure now falls to application team
- app team requires read/write access to the AWS environment


.. https://aws.amazon.com/blogs/compute/using-amazon-efs-to-persist-data-from-amazon-ecs-containers/

.. https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-ecs-ec2.html

.. https://aws.amazon.com/premiumsupport/knowledge-center/ecs-create-docker-volume-efs/

.. https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_efs.html


