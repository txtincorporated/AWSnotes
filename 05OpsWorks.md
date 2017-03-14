# Code Fellows Alumni Group
## Notes 2/17/17

### [Lynda.com AWS OPSWORKS](https://www.lynda.com/Amazon-Web-Services-tutorials/OpsWorks-key-concepts/502654/566946-4.html?srchtrk=index%3a5%0alinktypeid%3a2%0aq%3aaws%0apage%3a1%0as%3arelevance%0asa%3atrue%0aproducttypeid%3a2)

#### QUESTIONS & COMMENTS
1. **NOTE** strong contrast between OpsWorks and Elastic Beansatalk
    - EB heavily prescriptive; predefined infrastructure choices;   as PaaS, takes care of more of the stack decisions for you
    - OpsWorks allows granular control in describing your architecture
1. **HOW** is OpsWorks set up to work?
    - Define stack in given region; just a container for the app to be built
    - Defined in layers, e.g., an app server layer to handle the app, a data layer consisting of RDS instances, and a 'load balancing' layer (typically supplied off-the-shelf by attaching the stack to an existing ELB) 
1. **OPS WORKS LIFECYCLE** 
  1. SETUP 
    1. **NOTE** that Chef recipe runs when new instance in layer boots 
    1. **ALSO NOTE**  that recipe also can be run when triggered by user 
  1. CONFIGURE 
    1. **RUNS** when new layer boots up 
    1. **ALSO RUNS** when new Elastic IP attached  
    1. **RUNS ADDITIONALLY** when ELB is attached  
    1. **RUNS, BTW** on every instance in layer to ensure freshness   
  1. DEPLOY
    1. **RUNS** on manual re-deploy from developer 
    1. **ALSO RUNS** *before* CONFIGURE, mind you, on first boot 
  1. UNDEPLOY 
    1. **RUNS WHENEVER** user sets off a manual undeploy 
    1. **ALSO RUNS** if app is deleted from the stack 
  1. SHUTDOWN
    1. **RUNS** on any instance user tells to shut down 
    1. **ALLOWS** time for connection to drain before shutting down instance 
    1. **NOTE** does not run on reboot
1. **SOURCING APPS TO BE RUN** 
  1. **NOTE** that apps stored in S3 or elsewhere on the Web must be zipped
1. **DEFINING AN APP** 
  1. **MUST** specify source location
  1. **MAY** incl. SSL cert.
  1. **MAY** set env. vars.
  1. **SHOULD** supply platform specific settings
1. **CREATE OPS WORKS STACK** 
  1. Go to "Services>Management Tools>OpsWorks Console".
  1. Click "Add Stack" (or "Create First Stack" or some such).
  1. Choose "Chef 11 Stack" for pre-built layers without a default sample app.
  1. Enter "Stack Name", e.g., "RailsDemo".
  1. Keep default VPN and subnet, use Amazon Linux as OS.
  1. Select an SSH key from list (should incl. keys from previous tutorials).
  1. If sourcing own Chef cookbook, set "Use Custom Chef Cookbook" to "Yes" and configure repo type (incl. git and S3), URL and branch, and provide repo SSH key if using (not used in this tutorial).
  1. Click "Advanced".
  1. Select IAM role and instance profile.
      - **NOTE** that being logged in with root access these means these can default to OpsWorks-provided roles; otherwise, roles must be created or selected with appropriate OpsWorks full-access permissions .
  1. Leave "API endpoint" set to default.
  1. Select "Hostname Theme" as desired to set pattern for naming instances on creation.
  1. Leave "OpsWorks Agent Version" set to default.
  1. Leave "Custom JSON" blank; normally would over-ride any built in defaults you might wish to configure yourself.
  1. Leave "Security" toggle set to "Yes" for now to allow OW to use its built in security groups for all created resources.
  1. Click "Add Stack".
  1. Stack console comes up on completion with modular prompt to "Add a layer".
1. **ADD LAYER TO STACK** 
  1. Click "Add a layer" link in console for newly created stack.
  1. Select desired layer type from pulldown ("Rails App Server" for this tutorial).
  1. Select framework/language version and server stack as req'd for your specific type (for this tutorial, Ruby 2.2 and NGINX + Unicorn).
      - **NOTE** that for this tutorial is RubyGems 2.2.2 is required with "Install and Manage Bundler" set to "Yes" and Bundler version 1.5.3 selected.  
  1. Leave "Elastic Load Balancer" set to blank for now.
  1. Click "Add Layer".
  1. OpsWorks stack console comes up with view set to "Layers" and newly defined layer represented in header.
  1. Click into new layer and **NOTE** features:
      - **General Settings:**  summary of basic layer definition just set
      - **Recipes:**  links to GitHub repos for all the Chef recipes governing each phase of the app lifecycle
      - **Network:**  Mgt. console for adding and monitoring ELB to govern EC2 instances needed by the layer
      - **EBS Volumes:**  nothing added here yet
      - **Security:**  AWS security groups containing layer instances; clicking will open EC2 Dashboard to Security groups in new tab with these displayed
        - **NOTE** that in addition to setting HTTP on port 80 and SSH on port 22, both from the world, this shows a number of other traffic settings allowing different components of OpsWorks to intercommunicate
  1. Click "Layers" to return to main layer console and "Add instance" on the far righthand side to add instances to this as-yet unpopulated layer definition.
  1. Select "t2.micro" under "Size" for smallest, free-tier instance size.
  1. Select "Time based", "Load based" or default "24-7" under "Scaling type" to specify when instance will spin up.
  1. Set SSH key from pull-down listing previously generated keys.
  1. Leave other options set to defaults.
  1. Click "Add Instance".
  1. Stack appears in "Stopped" state in console; wait to click "Start" until after next step.
1. **DEFINE APPLICATION, PART I**
  1. Back in main stack console, choose "Add an app" under "Apps".
  1. Enter name for app (e.g., "My Art Gallery").
  1. Leave "Enable auto bundle" defaulted to "Yes" so that Bundler will install from Gemfile on deployment.
  1. Select source (for this tutorial, GitHub repo where art-gallery app is stored) and enter source URL and other access info (none needed here since code is in a public GitHub repo; HOWEVER, **NOTE** that git branch must be specified or will otherwise default to `master`).
  1. **IF USING DEMO APP FOR TUTORIAL,** go into `config/environments/production.rb` and change `config.assets.compile = false` to `config.assets.compile = true`.
      - **NOTE THAT THIS INSTRUCTION IS MISSING FROM THE TUTORIAL**
  1. If using env vars, set under "Environment Variables"
      - For this tutorial, set `SECRET_KEY_BASE` by doing `bundle exec rake secret` to generate a value in terminal (if bash starts whinnying about not being able to find a source for `pg` from the Gemfile try commenting out `gem 'pg'` in the last line of the Gemfile and giving it another go; just be sure to set it back when done), then copying it into the VALUE field.
      - Check "Protected value" box to hide value from all users in future, including yourself, and make it accessible only to the code running the app (no need for this now as value isn't all that important here)
  1. Click "Deploy", and wait for instance to compile and boot up. 
    1. In this tutorial, **NOTE ERROR MESSAGE:**  S.b. something to the effect of "FAILED"; IF SO:
    1. Check log at link in "Stack>Instances" failure message; Error Messages s.b. at end.
    1. Click to "Instances" in l.hand sidebar.
    1. Copy instance Public IP into Clipboard.
    1. In terminal shell do `ssh ec2-user@<paste in Public IP from clipboard> -i ~/.ssh/<yourdemokeyfilehere>.pem` followed by `sudo su - deploy` to change user to OpsWorks' user w username "deploy".
    1. `cd /srv/www`
    1. `ls` to check for presence of `my-helpfully-named-app`, and then `cd` into it.
    1. `ls` to confirm presence of `./releases` and `./shared`.
        - **NOTE** that had the deployment succeeded there would also be `./current`
    1. `cd ./releases` and `ls` to confirm presence of dated release dirs.
    1. `ls -lart` to see which release most recent, and then `cd` into it.
    1. `ls` to confirm presence of two blacked-out filenames, `database.yml` and `memcached.yml`, and `ls -lart` to reveal they are symbolic links with missing pathways.
    1. Try `cat database.yml` to receive Error Message `cat:  database.yml, No such file or directory`
        - **NOTE** this is because in configuring the app "None" was given as the selection for DB, so OpsWorks is now by default treating the app as having no DB despite app's actually having a `database.yml` that specifies, in this tutorial, `SQLite3` should be used.
        - **MUST** externalize DB and tell OW to use s.t. else esides "None"
    1. In OpsWorks "Instances" console, click "Stop" to shut instance down prep to reboot.
1. **CREATE DATABASE**
  1. Navigate to "Layers" view in OpsWorks console and click "Add layer".
  1. **NOTE** three tabs:  "OpsWorks", "ECS" and "RDS". 
  1. Click "Add an RDS DB instance" link in the Error message; "Step 1:  Select Engine" view opens up in new window/tab.
  1. For this tutorial, click "PostgreSQL" tab and then "Select"; view transitions to "Step 2: Production?"
  1. For this tutorial again, select "Dev/Test" option and click "Next Step"; view transitions to "Step 3:  Specify DB Details".
  1. Select as follows:
      - "db.t2.micro" for "DB Instance Class" to stay Free Tier Eligible.
      - "No" for "Multi-AZ Deployment"
      - Default storage settings 
      - `your-helpful-db-name` for the DB name
      - "postgres" or s.t. similarly helpful for username
      - suitably mnemonic password (always write them down!)
  1. Click "Next Step"; view transitions to "Step 4:  Advanced Settings"
      - **NOTE** need to previously set up security group called, e.g., "allow-postgres-from-the-world" with sole policy to allow all incoming PostgreSQL on port 5432 (use EC2 console to create)
  1. Select pre-generated "allow-postgres-from-world" security group and name the DB helpfully for associating it with the app, e.g., `art_gallery_db`.
  1. Leave all else to default and click "Launch DB Instance"; launch confirmation screen comes up w message that instance being created.
  1. Click "View Your DB Instances" to monitor instance creation and other operations; wait ferfrickin ever till it's available.
1. **DEFINE APPLICATION, PART II**
  1. Return to OpsWorks console and click into previously started stack and go to "Layers" in l.hand sidebar.
  1. Click "Add a layer" and select newly-added instance.
  1. Give newly-minted user/pass (e.g., as before, user "postgres" w your mnemonically helpful passwd).
  1. Click "Register with Stack"; view reloads with demo app stack console now showing a new RDS data layer.
  1. Click "Connect app" link in RDS quick look table; list of apps comes up showing one just connected, and then click "Edit".
  1. Apps console comes back up with option to set "Data source type"; our chance to take it off "None"!
  1. Select "RDS", then the associated db name connected with this stack, and fill in name of db created earlier.
  1. Click "Save" for OpsWorks to automatically spawn a templated db config file for the app.
  1. Click "Deploy App"; "Deploy App" view loads.
  1. Toggle "Migrate database" button to "Yes".
  1. Click "Deploy" (if button ghosted, check that db instance has restarted in OpsWorks "Instances" console).
  1. Wait to confirm deploy success, and then back to "Instances" console.
  1. Click public IP link in instance's table row to load app into browser.
1. **ADD LOAD BALANCER**
  1. Define new dedicated load balancer for this stack:
    1. Navigate to EC2 Dashboard.
    1. Click "Load Balancers" under "LOAD BALANCING" in l.hand sidebar.
    1. Click "Create Load Balancer" button, and then select "Classic Load Balancer" and click "Continue" to enter creation wizard.
    1. Set semantically helpful name like, e.g., "opsworks-elb-demo".
    1. **NOTE** that for non-secure demo purposes it is OK to leave "Load Balancer Protocol" settings defaulted to listen on port 80 and proxy to instances on same port internally, but normally one would set up HTTPS and set a cert.
    1. Click "Next:  Assign Security Groups".
    1. For this tutorial, just select previously defined "allow-http-ssh" group, which allows web traffic on port 80 and ssh on port 22; click "Next:  Configure Security Settings".
    1. Expected warning appears re. non-secure listening settings; ignore and click "Next:  Configure Health Check".
    1. No `index.html` for this app, so set ping endpoint to just `/`, which will provide a response, and then click "Next:  Add EC2 Instances" and then again "Next:  Add Tags", since OpsWorks will automatically take care of adding EC2 instances as needed.
    1. Add tag with key "Name" and value of some helpful name like "opsworks-elb-demo".
    1. Click "Review and Create"; confirm details, scroll to bottom and click "Create".
    1. At success confirmation screen, click "Close" and then navigate back to OpsWorks console to associate balancer with app web tier layer while waiting for ELB to spin up.
  1. Add instances to stack and associate load balancer.
    1. In "Layers" console, click into demo app server, click "Instances" in l.hand sidebar and then "+ Instance" beneath display table for instances.
    1. Step through configuration as before and click "Add Instance"; click "start" in instance's row in table and allow to spin up.
    1. Back in app server console, click "Network" tab and select newly-created ELB from dropdown.
    1. Click "Save"; ELB layer now shows up in "Layers" console.
    1. Add server name printout to DOM template in app as in previous tutorial and navigate to app by clicking public IP the ELB's "Layers" console display.
    1. Reload page a few times to see individual host names change.
1. **OPSWORKS TEARDOWN** 


#### TERMS & CONCEPTS
  * **stack:**  content and configuration of each N-tier stack layer and how they relate to one another
  * **layer:**  set of rules for configuring resources; tends to map well with layers in interior architecture
    - obviously, does nothing by itself; EC2 instance must be present to make it useful
    - provisioned with the software and configuration described in layer def.
  * **instance:**  Added to layers as necessary to making them useful... 
    - conform to layer definition and dependencies
    - can be added to layers *ad hoc*, on a schedule or programmatically (e.g., according to load) 

  * **N-Tier architecture:**  web application architecture in which Load Balancer (distribution) layer operates on top of server-clone layer (web app 'server layer'; EC2 instances), in turn on top of database ('data layer')
    - **NOTE** that app servers and web servers sometimes separated into their own layers with their own respective hosts
  * **configuration management:**  principle that it's preferable to script as many server provisioning and maintenance tasks as possible; accordingly handles configuration as code
    - **SOME CONFIGURATION MANAGEMENT TOOLS:**
      - Puppet Labs
      - Chef
         - OpsWorks' choice for its configuration syntax
         - "Recipes", as Chef scripts known, written in Ruby
         - ...how OW allows power-users to define own stacks using own code but...
         - also provides pre-built recipes to define layers of some common stacks 
         - ...defines layers in relation not of one-to-one but of many-to-one, each 
         - ...executing at a specific time in OW lifecycle (see above)  
      - Ansible
      - Each provides a dsl for compile, filestore, user-signup and permissions ops 
      - Created and maintained as user-owned open-source 
      - 


