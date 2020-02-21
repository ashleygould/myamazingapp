.. _solution_5:

Solution 5 - Lambda with S3, ApiGateway, and CloudFront
=======================================================

.. toctree::
   :titlesonly:
   :hidden:

   solution_5_details

This scenario the Rails application runs as a set of Lambda functions fronted
by an ApiGateway instance.  Access logs, uploaded files, user stats data and
weekly reports all reside in S3.  As in solution 4, weekly user stats reports
are served via CloudFront.

.. :ref:`solution_5_details`

Operational Considerations
**************************

- serverless architecture
- fully code-based/immutable infrastructure
- app team fully supports application and infrastructure
- app team members must become proficient with additional AWS
  Services - Lambda, ApiGateway
- application is now very tightly bound to AWS services
- potential cost savings

.. https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html

