.. _solution_4_details:

Technical Details - Solution 4
==============================




AWS Resources
-------------

The following are snippets of yaml are pseudo cloudformation describing AWS
resources and their interrelationships.

For a very complete example of ECS Fargate CloudFormation template see:
https://github.com/1Strategy/fargate-cloudformation-example


Primary AWS resources
*********************

A high-level overview of solution architecture

::

  VPC1:
    az1:
      public_subnet1:
        ALB1
      private_subnet1:
        ECS Fargate cluster
        RDS_DB_instance1
    az2:
      public_subnet2:
      private_subnet2:
        RDS_DB_standby_replica

  myamazingapp_logging_bucket
  myamazingapp_data_bucket
  myamazingapp_reports_cf_origin_bucket
  myamazingapp_ecr_repo



VPC Details
***********

The VPC is essentially the same as in previous solutions with
a few changes:

- addition of a endpoint for S3
- modified security groups

::

  VPC1:
    internet gateway
    vpc_endpoint_for_s3
    route table
    az1:
      public_subnet1
        route table
        nat gateway
      private_subnet1
        route table
    az2:
      public_subnet2:
        route table
        nat gateway
      private_subnet2:
        route table
    security groups:
    - public_web_access
      - notes: allow web access to the ALB
        source: world
        ports: 80, 443
    - alb_target_group_access
      - notes: allow ALB access to Rails application port on EC2 Instances
        source: public_web_access
        ports: 3000
    - private_subnets_access
      - notes: allow all resources in private subnets to access one another
        source: private_subnets_access
        ports: all




Appliction Load Balancer Details
********************************

The only change from solution 1 is with the target group.  
We set the TargetType to ``ip``.  The ECS Service resource will reference
this and provide an ip for the target group when the service comes up.

::

  ALB1:
    subnet: public_subnet1
    securitygroups: public_web_access
    target_groups:
    - target_group1:
      targetType: ip
    listeners:
    - listener1:
        protocol: https
        ports: 443
        certificates: 
        - myamazingapp.example.com
        rules:
        - type: forward
          forwardconfig:
            targetgroup: target_group1
    - listener2:
        protocol: http
        ports: 80
        rules:
        - type: redirect
          redirectconfig:
            proto: https
            port: 443



RDS DB Instance Details
***********************

Only change here is we drop the bastion host security group

::

  RDS_DB_instance1:
    VPC: VPC1
    MultiAZ: true
    StorageEncrypted: true
    securitygroups:
    - private_subnets_access
    DBsubnet groups:
    - db_subnetgroup1:
      - private_subnet1
      - private_subnet2


ECR Repository Details
**********************


ECS Fargate Cluster and Service Details
***************************************



CodePipeline Instance
*********************

This pipeline sources from a Github repository.  A CodeBuild stage
builds a Docker from the repo and pushes it to an the QA ECR repository.
This triggers a refresh in the QA ECS stack.  The Approval stage
sends SNS notification requesting review and apporoval of the QA environment.
The pipeline waits until a response is recieved, then builds and pushes
to the Prod ECR repository, triggering production deployment.

::

  Parameters:
    GitHubRepo
    GitHubSecret
    QA_REPOSITORY_URI
    PROD_REPOSITORY_URI

  Resources:
    PipelineExecutionRole
    CodeBuildServiceRole
    ArtifactsBucket
    MyAmazingApprovers
      Type: AWS::SNS::Topic
    GithubWebhook:
      Type: 'AWS::CodePipeline::Webhook'
      Properties:
        Authentication: GITHUB_HMAC
        SecretToken: GitHubSecret
  
    MyAmazingPipeline:
      Type: AWS::CodePipeline::Pipeline
        Role: CodePipelineServiceRole
        ArtifactStore: ArtifactBucket
        Stages:
          - Name: Source
            Provider: GitHub
            WebHook: GithubWebhook
          - Name: BuildImageQA
            Provider: CodeBuild
            ProjectName: BuildDockerImageQA
          - Name: GetApproval
            Provider: Approval
            SNSTopic: MyAmazingApprovers
          - Name: BuildImageProd
            Provider: CodeBuild
            ProjectName: BuildDockerImageProd
  
    BuildDockerImageQA:
      Type: AWS::CodeBuild::Project
        ServiceRole: CodeBuildServiceRole
        Artifacts: ArtifactBucket
          BuildSpec:
            phases:
              pre_build:
                commands:
                  - $(aws ecr get-login)
                  - TAG="$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | head -c 8)"
              build:
                commands:
                  - docker build --tag "${QA_REPOSITORY_URI}:${TAG}" .
              post_build:
                commands:
                  - docker push "${QA_REPOSITORY_URI}:${TAG}"

    BuildDockerImageQA:
      Type: AWS::CodeBuild::Project
        ...




