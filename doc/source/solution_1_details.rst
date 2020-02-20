.. _solution_1_details:

Technical Details - Solution 1
==============================



AWS Resources
-------------

The following are snippets of yaml pseudo code describing AWS resources
and their interrelationships.

In this solution the assumption is site AWS administrators may not be
provisioning resources with CloudFormation.

Solution Resource Overview::

  VPC1:
    EFS_filesystem1
    ALB1
    az1:
      public_subnet1:
        EC2_instance1
      private_subnet1:
        EFS_mountpoint1
        RDS_DB_instance1
    az2:
      public_subnet2:
        EC2_instance2
      private_subnet2:
        EFS_mountpoint2
        RDS_DB_standby_replica

  ALB_logging_bucket:
    bucket_policy


VPC Details::

  VPC1:
    internet gateway
    route table
    security groups:
      - public web access
        - world 
      - ec2 instance access:
        - ssh
      - efs mount target access
        - ec2 nfs client
      - rds db access
        - ec2 db client
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


ApplicationLoadBalancer Details (ALB)::

  ALB1:
    subnet: public_subnet1
    - target_group1:
        - EC2_instance1
        - EC2_instance2
    - listener1:
        protocol: https
        protocol: 443
        rules:
          - type: forward
            forwardconfig:
              targetgroup: target_group1
    - listener2:
        protocol: http
        protocol: 80
        rules:
          - type: redirect
            redirectconfig:
              proto: https
              port: 443

RDS database Details::

  RDS_DB_instance1:
    VPC: VPC1
    MultiAZ: true
    StorageEncrypted: true
    DBsubnet groups:
    - db_subnetgroup1:
      - private_subnet1
      - private_subnet2


EFS Share Details::

  EFS_filesystem1:
    VPC: VPC1
    access points:
    - /root/uploads
    - /root/user_stats
    mount points:
    - EFS_mountpount1:
        subnet: private_subnet1
    - EFS_mountpount2:
        subnet: private_subnet2


EC2 Instance Details::

  EC2_instance1:
    KeyPair:
    #LaunchTemplate: ??
    SubnetId: public_subnet1
    SecurityGroups: ec2_instance_access
    ImageId: AmazonLinux2
    InstanceType: ??
    IamInstanceProfile:
    BlockDeviceMappings:
    UserData:
