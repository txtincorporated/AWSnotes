# Code Fellows Alumni Group
## Notes 2/7/17

### [Lynda.com AWS ELASTIC BEANSTALK](https://www.lynda.com/Amazon-Web-Services-tutorials/Understand-Elastic-Beanstalk/502654/566935-4.html?srchtrk=index%3a5%0alinktypeid%3a2%0aq%3aaws%0apage%3a1%0as%3arelevance%0asa%3atrue%0aproducttypeid%3a2)

#### QUESTIONS & COMMENTS
1. **NOTE** structure of an Amazon PaaS setup:
  - domain name points to a load balancer distributing traffic to as many EC2 app servers as needed to handle traffic volume, all pointing in turn at a shared database or RDS instance
1. **NOTE** one typical app server architecture (used in this demo), all provisioned by EBS along with load balancing, DBs, etc:
  - NGINX web server serving static content and proxying all other requests to Rails app server 
  - Rails app server
  - Ruby for runtime with Bundler for package mgt.
1. **WHAT** EB actually does:
  - templates configuration of entire app environment from load balancer to EC2 instances to security and autoscaling groups to which they are assigned, RDS instances, etc.
  - manages load balancing and SSL certs
  - can deploy apps from either S3 or a git remote, e.g., GitHub
  - can carry out all environment creation and management, deploy and update apps in Beanstalk CLI
  - also handles some monitoring and tracks app health
  - IDE plugins available for, e.g., VS Code
  - can deploy either from EB CLI and git local or from AWS console
1. **INITIALIZE EB:**
  1. YOU WILL NEED 
    - git cli 
    - Python 2.7 (`which python` to verify location; `python --version` to check version; [www.python.org/downloads](https://www.python.org/downloads))
    - pip, the Python package manager (`which pip`; get from [pypi.python.org/pypi/pip](https:pypi.python.org/pypi/pip))
    - AWS EBCLI
  1. Installing EBCLI using pip:
    - `pip install --upgrade --user awsebcli` 
    - `--user` flag will tell it to install into your user space rather than globally 
    - Like a good English gin, you may have to give it a moment.
    - Open `~/.bash_profile` (or `bashrc` as the case may be) and add path for `eb` command:
      `export PATH=~/Library/Python/<python.version>/bin:$PATH`
    - Remember to do `source ~/.bash_profile` or open new shell to make command accessible
    - `eb --version` to confirm path is set
  1. Initialize EB app using EBCLI:  
    1. navigate to app root (in this tutorial, `/your/path/to/art_gallery`) 
    1. do `eb init` and set region (us-west-2 is default, so if you're in Oregon, just hit return)
    1. on security prompt, in AWS console go to Your Name>Security Credentials
    1. click 'Get Started With IAM Users' to set up a new IAM user and avoid working in AWS root
    1. click 'Add User'
    1. give user a name and check both the 'Programmatic Access' and 'AWS Management Console Access' boxes
    1. set password as desired
    1. click 'Next:  Permissions' and select 'Attach Existing Policies Directly'
    1. in search window, type 'administrator' and checkbox 'AdministratorAccess' in the list to give user full admin privileges equivalent to root, as these will be needed going forward
        - **NOTE** that this sort of defeats the purpose but is O.K. for this course
        - Normally 'ElasticBeanstalkFullAccess' would be a better choice
    1. click 'Next: Review'
    1. review user details and policy and then click 'Create user'
    1. **NOTE** success screen will be your only chance to copy credentials into the clipboard, so do not navigate away until first copy/pasting access ID into shell at `(aws-access-id)` prompt and showing, then copy/pasting secret key at next prompt (also have choice to download in .csv and send in e-mail message)
    1. Continue setting stackfile values in EB CLI: 
        - respond 'y' to query about Ruby
        - OK to go w/ default Ruby/Puma platform version
        - *'n' to CodeCommit query for now*
        - 'y' to SSH query to enable login to EC2 server instances in unlikely case manual changes should be necessary; use previously created demo key, s.b. listed as first choice
    1. `cat .gitignore` to confirm additions by `eb` to the effect of  

        ```bash
        # Elastic Beanstalk Files
        .elasticbeanstalk/*
        !.elasticbeanstalk/*.cfg.yml
        !.elasticbeanstalk/*.global.yml
        tmp/
        ```
    1. `git` add and commit changes
1. **CREATE EB ENVIRONMENT:**
  1. **NOTE:**
    * Creates all infrastructure needed for app to run on AWS
    * Will deploy app upon creation of environment 
    * Need to prep app to run on selected stack.  
  1. Edit Gemfile to specify chosen server ():
      1. Open Gemfile in editor
      1. Add `gem 'puma'` at end
      1. Save, `git` add and commit (no need to push, as EB doesn't talk to GitHub during deployment)
  1. Create dev environment:
      1. `eb create dev-helpful-env-name -i m1.small`
          * **NOTE** that `-i` option allows specification of which EC2 instance type to use; default t1 type will have insufficient RAM for the demo app, so m1.small is specified here
          * **NOTE** Again with 'This may take awhile'
          * **NOTE** that whether you deploy from GitHub or an S3 instance or what-have-you, EB will upload to S3
      1. Confirm progress creating env by going to 'Services>Elastic Beanstalk'; click on environment box score to see creation details in progress (Terminal will also display running log of during creation)
          - can also confirm S3 upload via 'Services>S3' and clicking Elastic Beanstalk bucket with your bucket's assigned region and AWS acct. no. in the name
          - confirm load balancer creation in EC2 dashboard by clicking 'Load Balancers' in list at top or lefthand sidebar
          - confirm autoscaling group in ECS as well, under 'Auto Scaling Groups' on the left; **NOTE** that 'Name' key under 'Tags' tab gives a very handy way of making sure which ASG you're looking at 
1. **VALIDATE EB ENVIRONMENT:**
  1. Open deployed app in browser:  `eb open dev-helpful-env-name`
  1. Investigate any error messages: `eb logs dev-helpful-env-name`
    - may find 
    
      ```bash
      Rack app error: #<RuntimeError: Missing `secret_token` and `secret_key_base` for
      'production' environment, set these values in `config/secrets.yml`
      ```
    - Set production secret_key_base env. variable to remediate this Rails-related issue:
      1. `bundle exec rake secret` to generate `<newkeyvalue>`
      1. `eb setenv SECRET_KEY_BASE=<newkeyvalue> -e dev-helpful-env-name`
      1. EB console will show env health changing to 'Info' while update takes place; log updates in Terminal 
      1. Once env health changes back to OK, reload app in browser and it should display  
1. **UPDATE EB ENVIRONMENT:**
  1. Make, a-c-p and validate local changes as usual
  1. Push to Beanstalk:
    1. `eb deploy dev-helpful-env-name`
    1. As before, wait for reboot if only single instance is running; health transitions from OK to Info.
1. **BEANSTALK DATABASE:**
  1. Move datastore from local storage to AWS RDS
    1. In EB Console click 'Configuration' in the lefthand sidebar; a number of controllable box score preference stat cards loads in the view 
    1. Scroll down to 'Data Tier' footer at bottom of page and click 'create a new RDS database'.
        - **NOTE** that this input spins up an instance that is closely tied to the structure of your app.
        - **THUS** in the real world it is better to roll one up of your own that's abstracted enough to be easily re-usable.
    1. Now to input a Master username and Master password 
        - **NOTE** that vals will be injected into environment variables
        - **NOTE ALSO** that password will be applied by EB rather than directly by the user, you 
    1. Click 'Apply' in RBS configuration view
        - **WAIT** for EB to update the environment 
    1. Return to text editor while environment updates and 
        1. Edit Gemfile by adding `gem 'pg'` (for postgres, of course) immediately after the `gem 'puma'` that was appended to file earlier
        1. Beanstalk will install PostgreSQL on stack when running Bundler through Gemfile, and next file, `config/database.yml` will use it to configure actual RDS instance 
    1. Still in text editor 
        1. Edit `config/database.yml`: 
          ```yaml
           production:
             <<: *default
             adapter: postgresql
             encoding: unicode
             database: <%= ENV['RDS_DB_NAME'] %>
             username: <%= ENV['RDS_USERNAME'] %>
             password: <%= ENV['RDS_PASSWORD'] %>
             host: <%= ENV['RDS_HOSTNAME'] %>
             port: <%= ENV['RDS_PORT'] %>
          
          ```
        1. `git` a-c-p
    1. `eb deploy dev-helpful-env-name`
      1. Once again health transitions to 'Info'
      1. Validate in browser by doing `eb open dev-helpful-env-name` and confirming that previously saved art is no longer being retrieved
      1. Validate functionality by posting new image file to DB 
1. **BEANSTALK CONFIGURATION:**
    - Expand capacity and by adding more instances
      - Elastic Beanstalk based entirely on auto-scaling...
      - ...thus need only increase the capacity of autoscaling group

  1. Get started: go to AWS Elastic Beanstalk console 
  1. Select 'Configuration' and click gear icon at corner of 'Scaling' control-param table 

  1. Expand 'Auto-scaling'
     - Auto-scaling group's control parameters displayed in selectable/fillable input console view 
     - Could make same changes directly in EC2 instance console, but not advisably  
  1. Before proceeding to make changes to ASG params, go back to 'Configuration>Network Tier', click gear icon and look at 'Load Balancing'
    1. Scroll down to 'Application health check URL'
    1. **NOTE** that no value has been set 
    1. Switch to EC2 console and go into 'Load Balancers' 
    1. Select ELB connected with Beanstalk environment in question  
    1. Select 'Health Check' tab
    1. **NOTE** that Ping Target defaults to TCP on port 80 with no way to define a target path 
        - Any ping sent on TCP will produce a passing health check as long as the response is TCP on port 80, even if that response is an error.
        - No such problem with HTTP on port 80
        - ELB will attempt to ping a specified HTTP endpoint
        - Only a response code <=200 from the path specified will cause health check to pass 
        - Could set ping target to HTTP on port 80 here, but not advisably for same reasons as previously 
    1. Switch to Terminal (could also return to Elastic Beanstalk console and do this part, but...)
        1. `eb config dev-helpful-env-name` -- opens env.yml file in GNU nano
        1.  Scroll down .yml to `aws:autoscaling:asg:` and find `MinSize` property.
        1.  Change `MinSize` val to 2 to maintain a minimum of two instances running at any given time.
        1.  Scroll further down to `aws:elasticbeanstalk:application:`
        1.  Set Application Health Check URL to `/`
            - Will cause all subsequent pings to address port 80 using HTTP and request app's root path
            - Thus eliminates the need to set ping protocol to HTTP anywhere else
        1. Write-out 
            - Config change begins with running status messages on command line
            - Elastic Beanstalk Dashboard shows 'Health' status transitioning as before with status messages displayed in bottom pane 
            - **NOTE** that any app updates will result in taking down first instance after clone is spun up so until the orig is updated and spun back up you will see a 'Severe' health status in the EB console which you can in this case ignore given that your new instance is already handling all the traffic fine.  Anon you should have two instances purring away and Health Status should go back to OK.
        1. Still should be able to navigate to your IPv4 Public URL and interact w/ app just fine
              
    1. **CONFIRM BOTH INSTANCES ACTUALLY HANDLING TRAFFIC**
        1. To demonstrate operation of two distinct instances
          1. Add `require 'socket'` on first line of `app/arts_controller.rb` and, immediately under `def index`, `@hostname = Socket.gethostname`
              - gets socket and records value as `hostname`
              - now passable to the view template 
          1. Add `<%= @hostname %>` to end of `app/views/arts/index.html.erb`
              - pulls hostname from `app/arts-controller.rb`
              - does equiv of JS `document.write()` in the DOM, only dynamically
          1. git a-c-p, and then `eb deploy dev-helpful-env-name` 
              1. Wait and wait for each instance to go down, update and redeploy. 
              1. Check EC2 console to confirm both instances running. 
          1. Navigate to app and check that host IP is displayed at bottom of page 
              1. Go to ELB URL listed at top of EB Dashboard Overview page.
              1. Note host IP. 
              1. Refresh and re-check host IP to confirm both EC2 instances responding in turn.
              1. How freakin cool is that!
    1. "Congratulations!" on being the proud owner of an external RDS database and a web tier that talk to each other reliably and can scale out to multiple hosts.

1. **BEANSTALK CUSTOMIZATION:**
    1. Three ways to customize:
        1. Custom AMI:  roll up own launch config with desired customizations and set ASG launch config to use it (make sure to base custom AMI on existing Beanstalk instance to guarantee Beanstalk Agent running)
            - CON:  involved process to get up and running 
            - PRO:  fastest spinup time for new instances once custom AMI type set up 
        1. UserData:  place custom shell scripts into this field in stackfile 
            - CON:  takes a while to write all the necesary scripts 
            - CON:  slow spinup time as shell scripts compile and run 
            - PRO:  accessible to ordinary version control 
            - PRO:  based on EB's already-existing process, so can be done in course of ordinary dev-ops 
        1. annnd...
    1. **`.ebextensions` DIRECTORY:** 
        - Lightweight built-in Beanstalk tool 
        - Place in app root 
        - Add to app revision 
        - Stocked with config files evaluated in alphabetical order 
        - EXAMPLE:  

        ```yaml 
         # basic.config

         # default package mgr. for Amazon Linux, installs Samba Client package
         packages: 
           yum:
             samba-client: []

         # define environment variables w/o using eb setenv
         option_settings: 
           - option_name: SECRET_KEY_BASE
             value: 1234

         # any needed commands; will be executed alphabetically
         container_commands:
           01deploy:
             command: rake db:seed
           02deploy:
             command: some_command.sh
        ``` 

        - **NOTE** that `packages` also supports Python, RBM and RubyGems; demo app uses Bundler and a Gemfile instead 
        - If placed in `.ebextensions`, will configure each new instance added to environment
        - **NOTE** other sections that can go into a `.ebextensions` file:
            - `users`:  create user accounts on the box
            - `groups`:  create user groups on the box
            - `sources`:  unpack .zip files
            - `files`:  create files from inline text or fetch from URL
            - `services`:  manage `ntp` and other UNIX services
1. **BEANSTALK TEARDOWN:**
    1. EB 'Dashboard>Actions>Delete Application'
        - Removes all infrastructure associated with app, including ASG
    1. RDS 'Dashboard>DB Instances' 
        1. Check RDS instance for app in question and click 'Instance Actions'
        1. Select 'Delete' and 'Create final Snapshot' at next screen if desired

#### TERMS & CONCEPTS
  * **AWS Elastic Beanstalk (EB):**  Amazon's application platform as a service, automating CloudFormation setup of AWS stacks for app deployment and provisioning
  * **PaaS:**  Platform as a Service, an extension of Infrastructure as a Service (IaaS), in which, e.g., AWS, takes the complexity and physical labor of, e.g., setting up a load-balancer and mirror servers, abstracts it out and implements it virtually for you in the form of an ELB or ALB talking to a bunch of EC2 instances, etc.; PaaS applies the same idea to getting an app server up and running
  * **stack:**  language version and app server used for a given application
  * **environment:** computing environments in which the app may run, e.g., development or production, to which applications and app revisions may be deployed
  * **rolling deployment:** in multi-server environments, running the update machine by machine so that there are always instances of the app running and thus no downtime for redeploy; may require adding instances to avoid the service hit from taking machines offline
  * **immutable deployment:** in multi-server environments, creating an entire new ASG with the updated version of the app and then shifting traffic to it from the old version once it is fully tested and ready to go; no downtime, no service hit
  * **IAM:** AWS Identity and Access Management; allows creation of multiple AWS users with various credentials and privileges in order avoid AWS root access except when absolutely necessary as well as for a host of other reasons related to access control in complex multi-user deployment schemes
  * **User Data:** key in the stackfile accepting as its value shell script strings that will execute on bootup and provision deployment with language-version and server stack selected; **NOTE** that since application software required by environment not embedded in AMI but rather is called by the user data and executes when environment starts up, it slightly slows Elastic Beanstalk deployments' transaction times down relative to manual ones; possible to avoid this routine lag by saving this config onto its own AMI for use in future Elastic Beanstalk deployments


