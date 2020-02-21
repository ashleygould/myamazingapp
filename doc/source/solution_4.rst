.. _solution_4:

Solution 4 - Docker in Fargate with S3
======================================


.. toctree::
   :titlesonly:
   :hidden:

   solution_4_details

In this scenario the ECS Cluster is defined using the Fargate Launch
Type.  This greatly simplifies AWS Resource coding and cluster
maintenance.  However, it precludes the use of EFS.  Instead, the
application now uses AWS API calls (`AWS SDK for Ruby`_) to post uploaded
files to an S3 bucket.

User stats data and weekly reports are now also posted to an S3 bucket.
Weekly reports are served using a CloudFront distribution.

:ref:`solution_4_details`


Operational Considerations
**************************

- fully code-based/immutable infrastructure
- app team fully supports application and infrastructure
- app team members must become proficient with AWS SDK for Ruby
- app team members must become proficient with additional AWS
  Services - S3, CloudFront

.. _`AWS SDK for Ruby`: https://aws.amazon.com/sdk-for-ruby/


