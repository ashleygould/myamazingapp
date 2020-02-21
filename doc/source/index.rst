.. MyAmazingApp documentation master file, created by
   sphinx-quickstart on Thu Feb 20 09:44:16 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to MyAmazingApp's documentation!
========================================




This is my response to job interview exercise

Exercises description
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
- User Authentication
- Expertise and comfort level of developers
- Integration of AWS API calls within application code 
- Resource Tagging
- Orchestration



My Solutions
------------

I came up with five solutions. All solutions assume AWS as cloud
provider.  Taken in sequence, this set of solutions describes a possible
evolution path for application hosting in AWS.

I have included detailed technical specs for solutions 1 and 4 only.


.. toctree::
   :titlesonly:

   solution_1
   solution_2
   solution_3
   solution_4
   solution_5

