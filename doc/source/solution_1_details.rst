.. _solution_1_details:

Technical Details - Solution 1
==============================


The following are snippets of yaml pseudo code describing AWS resources
and their interrelationships.

In this solution the assumption is site AWS administrators may not be
provisioning resources with CloudFormation.



AWS Resources
-------------

Layout of primary AWS resources within the VPC.

::

  VPC1:
    EFS_filesystem1
    ALB1
    az1:
      public_subnet1:
        EC2_bastion_host
      private_subnet1:
        EC2_instance1
        EFS_mountpoint1
        RDS_DB_instance1
    az2:
      public_subnet2:
      private_subnet2:
        EC2_instance2
        EFS_mountpoint2
        RDS_DB_standby_replica

  ALB_logging_bucket:
    bucket_policy


VPC Details
***********

::

  VPC1:
    internet gateway
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
      - source: world
        ports: 80, 443
    - public_ssh_access:
      - source: on-prem developer hosts
        ports: 22
    - alb_target_group_access
      - source: public_web_access
        ports: 3000
    - bastion_host_access
      - source: public_ssh_access
        ports: all
    - private_subnets_access
      - source: private_subnets_access
        ports: all


Appliction Load Balancer Details
********************************

::

  ALB1:
    subnet: public_subnet1
    securitygroups: public_web_access
    target_groups:
    - target_group1:
      - EC2_instance1
      - EC2_instance2
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

::

  RDS_DB_instance1:
    VPC: VPC1
    MultiAZ: true
    StorageEncrypted: true
    securitygroups:
    - bastion_host_access
      private_subnets_access
    DBsubnet groups:
    - db_subnetgroup1:
      - private_subnet1
      - private_subnet2


EFS FileSystem Details
**********************

::

  EFS_filesystem1:
    VPC: VPC1
    securitygroups:
      private_subnets_access
    access points:
    - EFS_accesspoint1: /root/uploads
    - EFS_accesspoint2: /root/user_stats
    mount points:
    - EFS_mountpount1:
        subnet: private_subnet1
    - EFS_mountpount2:
        subnet: private_subnet2


EC2 Bastion Host Details
************************

::

  EC2_bastion_host:
    KeyPair: ec2_admin
    SubnetId: public_subnet1
    SecurityGroups:
    - public_ssh_access:
    ImageId: AmazonLinux2 AMI
    InstanceType: t3.nano
    UserData:
      #!/usr/bin/bash
      yum update -y


EC2 Instance Details
********************

AWS specs for two EC2 instances are identical with the exception of the
``SubnetId``.

::

  EC2_instance1:
    KeyPair: ec2_admin
    SubnetId: private_subnet1
    SecurityGroups:
    - bastion_host_access
      alb_target_group_access
      private_subnets_access
    ImageId: AmazonLinux2 AMI
    InstanceType: t3.medium
    UserData: bootscript.sh

EC2 Userdata Script::

  #!/usr/bin/bash
  # bootscript.sh

  yum update -y


  # NFS Mounts
  EFS_FS_ID="fs-12345678"
  EFS_accesspoint1="fsap-XXXXXXXXXXXXXXXXX"
  EFS_accesspoint2="fsap-YYYYYYYYYYYYYYYYY"
  cat << EOF >> /etc/fstab
  file-system-id $EFS_FS_ID efs _netdev,tls,accesspoint=${EFS_accesspoint1} 0 0
  file-system-id $EFS_FS_ID efs _netdev,tls,accesspoint=${EFS_accesspoint2} 0 0
  EOF
  mount -a


  # Install Rails
  #
  # helpful links
  # https://guides.rubyonrails.org/command_line.html
  # http://blog.serverworks.co.jp/tech/2020/01/19/rails6/
  #
  amazon-linux-extras install -y ruby2.6
  yum install -y gcc gcc-c++ make zlib-devel git 
  yum install -y ruby-devel sqlite-devel
  
  curl -sL https://rpm.nodesource.com/setup_12.x | sudo bash -
  curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
  yum install -y nodejs yarn
  
  gem install sqlite3
  gem install rails


  # Install Puppet Agent and generate client SSL certificate
  gems install puppet gpgme
  cat << EOF >> /etc/puppet/puppet.conf
  server = puppet.example.com
  EOF

  chkconfig puppet on
  service puppet start
  puppet agent --no-daemonize --onetime




