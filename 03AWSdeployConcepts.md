# Code Fellows Alumni Group
## Notes 2/7/17

### [Lynda.com AWS DEPLOYMENT CONCEPTS](https://www.lynda.com/Amazon-Web-Services-tutorials/Architect-apps-horizontal-scaling/502654/566932-4.html?srchtrk=index%3a5%0alinktypeid%3a2%0aq%3aaws%0apage%3a1%0as%3arelevance%0asa%3atrue%0aproducttypeid%3a2)

#### QUESTIONS & COMMENTS
1. **NOTE** that it's a good idea to avoid using the file system for any user data when scaling horizontally.
  - If load balancer sends user's next click to a different machine than the last one, user's session data will no longer be the same.
  - Good to use database-backed or cookie session stores
  - Upload assets to DB or Amazon S3
  - Amazon EFS possibly a good solution (see below)
1. **TO USE DEMO APP:** Follow instructions for setup and server launch in [Initial Setup]() and navigate to `localhost:3000`
1. **NOTE:** With db-backed Rails apps, necessary to ensure that `bundle exec rake db:migrate` is run when setting up the app locally as well as when deploying; AWS services often take care of this step
  - **WHAT** would be the NodeJS equivalent of this caveat?  Just making sure that the DB path is set either in `package.json` or a `.env` file?
1. **NOTE:** Rails' built-in SQLite db not an appropriate choice for deployment 
  - popular alternatives incl. Unicorn, Passenger and Puma
  - need to be decoupled from file system and moved to, e.g., a MySQL or PostgreSQL db you run yourself on an EC2 instance or run on Amazon RDS (see below)
  - one popular remote db solution: have NGINX web server proxy requests to Unicorn 
  - will need to install NGINX, copy global config file, copy app-specific config file, and control NGINX start/stop during deploy process
1. **ALSO NOTE:** necessary to config app as well, in this case make sure `bundler` installed, run `bundle`, and start/stop Unicorn; enter Elastic Beanstalk


#### TERMS & CONCEPTS
  * **horizontal scaling:**  adding machines to a deployment configuration rather than increasing the capacity of existing ones
  * **Elastic File System (EFS):**  self-scaling shared storage volume mountable by multiple instances
  * **Amazon Relational Database Service (RDS):**  Amazon's managed db service
