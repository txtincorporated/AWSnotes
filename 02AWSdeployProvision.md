# Code Fellows Alumni Group
## Notes 2/2/17

### [Lynda.com AWS DEPLOY AND PROVISION](https://www.lynda.com/Amazon-Web-Services-tutorials/Amazon-Web-Services-Deploying-Provisioning/502654-2.html?srchtrk=index%3a5%0alinktypeid%3a2%0aq%3aaws%0apage%3a1%0as%3arelevance%0asa%3atrue%0aproducttypeid%3a2)

#### QUESTIONS & COMMENTS
1. **INITIAL SETUP:** 
  1. Sign up for AWS at [aws.amazon.com](https://aws.amazon.com).
    - To pay only nominal costs, sign up for basic service and follow the teardown instructions carefully in the lynda.com tutorial.
  1. Set up or clone a GitHub repo with the code you want to use for your app, and one for the deployment.
    - **NOTE** that both [a client](https://github.com/brandon-rich-aws-deployment-course/art_gallery) and [an API connection](https://github.com/brandon-rich-aws-deployment-course/cloudformation-demo) repo are available to clone if you'd like to configure along step by step with the tutorial.

  1. If configuring the sample repos to follow along with the tutorial: 
    - **Install the bundle** in Ruby's answer to `npm install`, running from the Gemfile, Ruby's answer to package.json, the `bundle` command installs the project dependencies listed in the Gemfile and sets some roles and test script commands.
      - **NOTE, MAC USERS,** that Terminal may not have the `bundle` command installed unless you've already been coding Ruby; if not, run `sudo gem install bundler` and then try running `bundle` again.
      - Be prepared to supply your user password for root access during the bundle install. 
      - Also be prepared to wait for stuff to happen with no feedback as to whether it's actually doing so. 
      - **NOTE, INFREQUENT RUBY USERS** that if you run `bundle --verbose` or `bundle install --verbose` you may actually see a log message reading something like
      ```bash
      Building native extensions with: '' 
      This could take a while...
      ```
      ...which appears intended in earnest.  So don't sweat it; go get coffee; walk the ferret; whatever; it'll eventually finish installing all the dependencies and give you back your prompt.
    - **Install the db:** `bundle exec rake db:migrate`
    - **Run server:** `rails server`; will open on `localhost:3000`
    
1. **EC2:** 
  1. EC2 instances: virtual machines you launch on demand in the AWS cloud
    - created on demand 
    - can be triggered by more advanced AWS services in response to your predefined conditions
    - pay-as-you-go pricing model
  1. Amazon Machine Images
    - contain basic install of a chosen OS like Windows Server or Red Hat (may require license)
    - can roll your own AMI as well to customize for your specific needs
    - choose from predefined instance type families with various capabilities and hourly rates:
        * general purpose
        * storage optimized
        * GPU instances 
        * compute optimized
        * memory optimized 
    - t2 series is the workhorse general purpose machine
        * general purpose
        * burstable
        * like all AMIs, type can be changed on the fly with a simple restart
        * t2.micro is a great starter
  1. Region:  Designated geographical area that is logically isolated from the others
    - us-west-1: California
    - us-west-2: Oregon
    - us-east-1: Virginia
    - us-east-2: Ohio
    - Overseas regions:  Ireland, Frankfurt, Singapore, Seoul, Tokyo, Mumbai & Sao Paulo
    - Very few cross-regional resources
    - Think about latency in selecting and stick as close to both your users and your resources as possible
  1. Availability Zone (AZ):  Sub-regions based on the actual physical data centers in a given region; named by letter, e.g., us-east-1 b, and must be selected to provide redundancy for resources that must not go down even if disaster strikes; separated enough physically to be isolated from each other's physical threats, but close enough to do so without significantly increasing latency
  1. Storage 
    - instance-backed storage:  essentially the hard drive on the machine running your VM 
    - Elastic Block Store:
        * specify size, type and volume mapping and AWS configures virtual storage to spec 
        * independent of EC2; re-usable from one instance to the next:  data persists after reboot unless explicitly deleted     
  1. Other features:
    - SSH keys
    - security groups
    - key-value metadata tags for tracking resources 
1. **AWS ELASTIC LOAD BALANCERS (ELB):** 
  1. Distribute excess traffic from one instance to backups
    - EC2 security group should be configured to allow only traffic from the ELB
    - ELB itself has a separate security group to expose your web traffic 
    - need to open and map listeners for traffic; may listen to any port and forward to any port on back end
    - Can also be internal (VPC traffic only)
    - subject to their security group; apply same security you would to EC2 instance
    - use SSL (https)
    - install SSL certificate on the server itself on a single-instance app like this
    - can use free AWS cert management if you are using AWS; update cert on all your load balancers from one place
1. **AWS APPLICATION LOAD BALANCERS (ALB):** 
  1. Differ from ELBs or so-called "Classic load balancers"
    - ELBs just convey data from listening to mapped port
    - ALBs can decide where to send traffic based on URL, header, and can route to multiple ports on same machine
1. **AWS AUTO-SCALING** 
  - vertical scaling: resize your EC2 instance; requires reboot
  - horizontal scaling: add more instances; requires some tedious work
  - auto-scaling = horizontal scaling as a service; spins up new instances as needed; you pay only for the op time
  - can point your ELB group to an ASG for horizontal auto-scaling
  - take prescribed action such as spinning up new instance on firing of policy events defined by you in a Launch Configuration
    * **NOTE** that launch configs cannot be edited; can only make changes by duplicating with desired alteration and re-assigning to desired ASG
  - PROVISIONING OPTIONS:
    * pre-bake AMI as basis for launch config
    * script config based on user data
    * ASG has name, config of where to build new instances, and starting group-size
    * scaling policies define conditions for adding and removing instances based on CloudWatch Alarms
    * can also define load balancer for group, tag group for tracking, event notifications and alerts, sched. scaling events at predetermined times
1. **AWS SECURITY GROUPS** 
  - differs from traditional firewall in being simpler, more modular and thus more scalable
  - reusable groups of network security rules
  - define incoming and outgoing rules
  - can be defined to almost any AWS resource:  EC2 instances, ELBs, RDS databases...
  - act like local firewalls for their given resource
  - deny all incoming and allow all outgoing traffic by default
  - **NOTE** that without a security group attached to a resource AWS allows no traffic in or out, so removing traditional firewall protection doesn't expose internal resources.
  - can also be configured to accept traffic from each other, not just allowed IP ranges; just provide the ASG's id rather than an IP block and you're in business -- IPs inside the allowed group don't have to be individually specified in the target group's policy
1. **SSH KEY AUTHENTICATION** 
  - conventional authentication:
    * port 22 must be accessible
    * host must have SSHD running
    * must have username and password
    * passwd auth enabled
    * must use OS X's built-in SSH command or PuTTY for Windows
  - AWS authentication:
    * Amazon Linux automatically provisions default admin level "EC2 User"
    * SSH key auth used instead of passwd by default
    * generate pub/pvt key pair with `ssh keygen` command and copy contents of pub file into .pub file on remote user
    * **NOTE** that AWS generates the private/public key pair for you out of the box; gives you a one-time download of private key (your-key.pem; default location `~/.ssh`) 
    * **ALSO NOTE** that .pem file must be set to read-write only by owner (`chmod 400 my-key.pem`)
    * EC2 instances automatically given your pub key, meaning you can log in using your private key without further ado
    * run `ssh ec2user@<remote-ip> -i /path/to/private-key.pem`
    * **DANGER!  NOTE!** THAT EC2 HAS NO BACK DOOR -- lose your private key and you permanently lose access to the instances that accept it
    * probably want to organize your keys by adding them to your ssh keyring: 
      - add key: `ssh-add /path/to/private-key.pem`
      - remove key: `ssh-add -d /path/to/private-key.pem`
      - list keys: `ssh-add -l`
      - **NOTE** that having more than five keys to try on a ring can lead to lockouts on some systems
1. **CONFIGURE AND LAUNCH AN INSTANCE** 
  1. login to https://aws.amazon.com/console/
  1. good page to explore
  1. shows your region at the top righthand corner; resources will then be created within that region
  1. click "Launch Virtual Machine with EC2"
  1. click "advanced EC2 Launch Instance wizard" for full control over configuration
  1. select OS (our example: Amazon Linux)
  1. select instance type (our example: t2.micro, which is Free Tier Eligible -- yay!)
  1. click "Next: Configure Instance Details" button
    - Network:  Allows choice of which virtual data center to build instance in
    - Shutdown behavior: you want "stop" not "terminate"
    - Advanced Details: text box allows shell scripts for runtime package install and config
  1. click "Next: Add Storage"
    - default 8GB SSD installed; good enough for demo purposes
  1. click "Next: Tag Instance"
    - metadata key/value pairs used for example, to...
    - give instance a Name (will always appear at top of list, great for tracking instances and keeping them organized)
    - indicate its purpose
    - etc.
  1. click "Next: Configure Security Group"
    - Set up ports and access for SSH and http 
    - **NOTE** that 0.0.0.0/0 means worldwide access; OK for demo but you'd want to be more specific in production 
    - Defaults to port 22 for SSH (required), port 80 for HTTP
  1. click "Review and Launch"
      - Double check details
  1. click "Launch"
      - Set and download .pem file unless you already have a private/public key pair
      - click "Services>History>EC2" to monitor launch
  1. log into instance
      - set .pem file permissions to read-only owner: `chmod 0400 /path/to/key.pem`
        - use `ls -l` to confirm `-r--------@ 1 youtheuser staff <file size> <today's date and current timestamp> your-key.pem`
      - ssh login info with `-i` option specifying path to private key:  `ssh ec2user@<assigned IP> -i /path/to/key.pem`
      - click through scary looking prompts
      - You're in your server!
1. **AWS CLOUDFORMATION** 
  1. Build and manage your data center via code 
  1. Reusably configure whole stacks of AWS resources at a time in JSON or YAML and spin them up at will
  1. Structurally complex service configuration can be done fully programmatically at any scale
  1. Formalized in code and can thus be version-controlled and process-managed in a single document 
1. **STACK CREATION AND PROVISION WITH CLOUDFORMATION -- GENERATE STACKFILE**
  - **NOTE** that for purposes of these notes .yml will be used as the configuration language and the alternative, .json was not attempted while creating them, but to make them useful for doing the same deployment using a .json stackfile should require only accounting for the difference in syntax and file format between the two. 
    * For example .yml and .json formatting see the CloudFormation docs [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html). 
  - **IMPORTANT:** all indentations must be two spaces and two spaces only for .yml to know what you're talking about.
  ```yml 

    ---
    Description:  Example Cloudformation Template
    Parameters:  
      Subnet:
        Description: Where to put this instance
        Type:  AWS::EC2::Subnet::Id
      SecurityGroup:
        Type:  AWS::EC2::SecurityGroup::Id
      InstanceType:
        Type:  String
        Default:  t2.nano
        AllowedValues:
          - t2.nano
          - t2.micro
          - t2.small
    Resources:
      MyInstance:
        Type:  AWS::EC2::Instance
        Properties:  
          ImageId:  ami-f173cc91
          InstanceType:  
            Ref: InstanceType
          SecurityGroupIds:  
            - Ref:  SecurityGroup
          SubnetId:
            Ref: Subnet
          UserData:
            Fn::Base64: | 
              #!/bin/bash -xe
              yum install nginx -y
              sudo service nginx start
  ```
  - **NOTE** that `Description` is required for the `Subnet` property.
  - **ALSO NOTE** that nearly all `Types` are given as parameters; these will be supplied by user selection in the AWS console at provision time.  `t2.nano` is treated as the default for purposes of this demonstration.  
  - **NOTE FURTHER** that the AMI types shown in the dashed list are array values that will be used to populate a select in the AWS console during the config.
  - **NOTE STILL FURTHER** that the only hard-coded value is the instance `ImageId`, which can be obtained by going to the Dashboard, clicking the **Launch Instance** button, and then copying the desired image-id from the list of available machine types on the next screen (AWS Linux SSD Volume Type used in this demo) and pasting it into your code. 
  - **NOTE EVEN FURTHER YET** (YAML newbies) that on the `SecurityGroupIds` branch's `- Ref` property the leading dash is required for proper configuration in that it sets off an array in the same way that bracket notation does in .json. 
  - **ADDITIONALLY,** be aware that if you are using a new AWS account without previous resources configured on it you may have only two security groups to choose from, the AWS default, which may cause connection timeouts, or the one set up earlier, allowing all HTTP and SSH traffic in and out on port 22 for SSH and port 80 for HTML, which explicitly meets the minimal default port requirements of the nginx server you are installing.
  - **AND FINALLY,** notice that as for the value given for `UserData`, which allows shell scripting content to be added to the configuration, the notation shows a bash function being added as the value for the `Fn` key with an option specified to treat the value as a single base-64 string and everything after the pipe parsed as code at config time.  This particular script installs and launches the nginx server on the machine instance. 
  - **SAVE FILE FOR UPLOAD TO CLOUDFORMATION:** 
      - Any convenient filepath fine 
1. **STACK CREATION AND PROVISION WITH CLOUDFORMATION -- CLOUDFORMATION UI**
  1. In AWS console click 'Services' and choose 'CloudFormation' from the menu under 'Management Tools'.
  1. Click 'Create Stack'.
  1. Under 'Choose a Template', click second radio button to upload stackfile; navigate to and select stackfile.
  1. Choose a semantically helpful name for the stack.
  1. Select SecurityGroup and Subnet from the dropdowns (**NOTE** that AWS default can cause problems as described above).
  1. Add 'Name' tag to have instance display in list after creation, and assign semantically helpful name.
  1. Review options and click 'Create'.  Process will take a minute or two; progress can be tracked in lower pane.
  1. Test by going to EC2 console, checking instance and copying IPv4 Public IP from 'Description' tab, then pasting IP into a browser nav bar and hitting 'return'. 'NGINX' test page should display in browser.

#### TERMS & CONCEPTS
  * **Elastic Compute Cloud (EC2):**  central service supporting the others provided by AWS:
    - **Instances:**  virtual machines you launch in the AWS cloud as you decide to configure them, either on demand or programmatically
    - **Pay as you go pricing:** only pay for time your instance is running (cost varies depending on resource requirements of the system you want to host)
    - **Elasticity:** Run Windows, Ubuntu, Red Hat, AWS-Amazon-Linux oss; pay-as-you-go; create, resize, and destroy instances -- platform optimizes elasticity of options available through AWS EC2
  * **`bundle`:** Ruby's shell utility for installing Ruby project dependencies from specificiations stored in a data file; may need to be installed on some OS X-based systems by running `gem install bundler` with root access; invoked from project root where the Gemfile is stored
  * **AMI:** Amazon Machine Image, a virtual machine deployed in Amazon's cloud with your choice of server OS installed; come in several families of both general purpose and specialized "instance type" configurations and can also be custom-configured to suit your specific needs
  * **burstability:** capacity of an AWS AMI to provide extra processing capacity as needed based on accrued "CPU credits" gained during normal operation (demand pricing)
  * **AWS auto-scaling groups:** groups of AMIs handling a given traffic stream that are configured to spin up new instances upon observing predetermined events such as reaching given load characteristics
  * **stack file:**  a master .yml file used to configure complex assemblages of AWS resources such as, e.g., load balancers, EC2 host server and data storage instances, interoperational rules and security groups, so that they can be spun up programmatically without user input to the console based on the attainment of specific threshold values in watched processes; parameterizable so that not just individual machines but whole data center infrastuctures now defined in code -- can be spun up automatically if given the correct predefined conditions -- can be edited, version-controlled and pipeline managed in a single document.


