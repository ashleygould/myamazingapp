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

::

  Resources:
    EcrRepository:
      Type: "AWS::ECR::Repository"
      Properties:
        RepositoryName: MyAmazingImageRepo


ECS Fargate Cluster and Service Details
***************************************

This example is shamelessly cut-n-paste from:
https://github.com/1Strategy/fargate-cloudformation-example

::

  Parameters:
    ServiceName
    ContainerPort

  Resources:
    Cluster:
      Type: AWS::ECS::Cluster

    ExecutionRole:
      Type: AWS::IAM::Role
    TaskRole:
      Type: AWS::IAM::Role

    TaskDefinition:
      Type: AWS::ECS::TaskDefinition
      Properties:
        NetworkMode: awsvpc
        RequiresCompatibilities:
          - FARGATE
      ExecutionRoleArn: !Ref ExecutionRole
      TaskRoleArn: !Ref TaskRole
      ContainerDefinitions:
        - Name: !Ref ServiceName
          Image: !Ref Image
          PortMappings:
            - ContainerPort: !Ref ContainerPort
      Secrets:
        - Name: DATABASE_PASSWORD
          ValueFrom: arn:aws:ssm:::parameter/qa_db_passwd

    Service:
      Type: AWS::ECS::Service
      Properties:
        ServiceName: !Ref ServiceName
        Cluster: !Ref Cluster
        TaskDefinition: !Ref TaskDefinition
        DesiredCount: 2
        LaunchType: FARGATE
        NetworkConfiguration:
          AwsvpcConfiguration:
            AssignPublicIp: DISABLED
            Subnets:
              - private_subnet1
              - private_subnet2
            SecurityGroups:
              - private_subnets_access
        LoadBalancers:
          - ContainerName: !Ref ServiceName
            ContainerPort: !Ref ContainerPort
            TargetGroupArn: target_group1.arn



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



Docker Details
--------------

Project Dir::

  [agould@localhost docker]$ ls -1
  app
  bin
  config
  config.ru
  db
  docker-compose.yml
  Dockerfile
  entrypoint.sh
  Gemfile
  Gemfile.lock
  lib
  log
  package.json
  public
  Rakefile
  README.md
  storage
  test
  tmp
  vendor


Dockerfile::

  # https://docs.docker.com/compose/rails/
  #
  FROM ruby:2.6
  
  RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
  RUN mkdir /myapp
  WORKDIR /myapp
  COPY Gemfile /myapp/Gemfile
  COPY Gemfile.lock /myapp/Gemfile.lock
  RUN bundle install
  COPY . /myapp
  
  # Add a script to be executed every time the container starts.
  COPY entrypoint.sh /usr/bin/
  RUN chmod +x /usr/bin/entrypoint.sh
  ENTRYPOINT ["entrypoint.sh"]
  EXPOSE 3000
  
  # Start the main process.
  CMD ["rails", "server", "-b", "0.0.0.0"]


docker-compose.yaml::

  # https://docs.docker.com/compose/rails/
  #
  version: '3'
  services:
    db:
      image: postgres
      volumes:
        - ./tmp/db:/var/lib/postgresql/data
    web:
      build: .
      command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
      volumes:
        - .:/myapp
      ports:
        - "3000:3000"
      depends_on:
        - db
















.. links

.. https://aws.amazon.com/premiumsupport/knowledge-center/ecs-data-security-container-task/
