# Code Fellows Alumni Group
## Notes 3/15/17

### [Lynda.com AWS CODEDEPLOY](https://www.lynda.com/Amazon-Web-Services-tutorials/AWS-CodeDeploy-key-concepts/502654/566954-4.html?srchtrk=index%3a5%0alinktypeid%3a2%0aq%3aaws%0apage%3a1%0as%3arelevance%0asa%3atrue%0aproducttypeid%3a2)

#### QUESTIONS & COMMENTS
1. **WHAT** is CodeDeploy?
   - still less automation and still more control than, successively, either Elastic Beanstalk or OpsWorks
   - write your own fully custom deployment scripts
   - five different, lifecycle-specific deployment hooks for scripts to run at key events like, for example app stop
   - deploy from either S3 or Git
   - no infrastructure mgt. must build and provision this yourself; heavy!
1. **WHERE** can I deploy my code from using CD?
   - Amazon S3 location (using object ID)
   - Git remote (e.g., GitHub or BitBucket; use commit ID)
   - **NOTE** that you *must* therefore have a remote repository in order to deploy from Git; can't just deploy from your local.
1. **WHERE,** as it were, is CodeDeploy Agent, as it were, "installed"?
1. **CAVEATS:**
   - Did I mention "No infrastructure management?"
   - Slow dev iteration cycle: no way to test `appspec.yml` (see below) locally
   - No immutable deploys:  all changes propagate through the entire deploy gp.
   - Must assign load balancer via user scripts; can't be done in the GUI
1. **APPSPEC**
   - `.yml` file residing at app root
   - refers to the other files and scripts needed to run the app
   - scripts will execute at one of five hooks each specific to given lifecycle event
1. **LIFECYCLE EVENTS**
  1. ApplicationStop
  1. BeforeInstall
  1. AfterInstall
  1. ApplicationStart
  1. ValidateStatus
  1. **NOTE** that you may specify configuration scripts for CD Service to run 
    - do anything you could do yourself running them directly on the host
1. **APPSPEC SECTIONS**
  1. Hooks:  see above
  1. Files:  move files around
  1. Permissions:  set file permissions
  1. **NOTE** that Files and Permissions functionality could also be folded into Hooks scripts but are more intuitive for, e.g., users and fellow devs if broken out into their own sections
  1. EXAMPLE:
  ```YAML
    version: 0.0

    os: linux
    files:
      - source: codedeploy/config/nginx.conf
        destination: /etc/nginx
    hooks:
      BeforeInstall:
        - location: codedeploy/scripts/install_dependencies.sh
      AfterInstall:
        - location: codedeploy/scripts/start_web_server.sh
  ```
  **NOTE** that `version` and `os` keys are required
  **ALSO NOTE** that the `nginx.conf` file, which in this tutorial configures the NGINX instance once moved into `/etc/nginx`, must in order to do so be present to begin with in the app's `S3` folder or Git remote
1. **NOTE** that once pulled down to host, revision gets stored under `/opt/codedeploy-agent/deployment-root/` in folders specific first to the deployment group and then the specific deployment
  - in case app requires referencing app path directory in this dynamic environment, several ENV_VARs available:
    1. `$LIFECYCLE_EVENT` (i.e., "ApplicationStop")
    1. `$DEPLOYMENT_ID` (i.e., A-PQR012)
    1. `$APPLICATION_NAME`
    1. `$DEPLOYMENT_GROUP_NAME`
    1. `$DEPLOYMENT_GROUP_ID`
    1. **NOTE** that `$DEPLOYMENT_ID` and `$DEPLOYMENT_GROUP_ID` can be used to construct app revision route at deploy time.
1. **USE CODEDEPLOY**
  1. **SETUP IAM ROLES**
    1. Navigate to IAM Console from, e.g., AWS Main Console, in order to deputize CD to communicate with EC2 instances and S3.
    1. Go to "Roles" in l.hand menu, click in, and then click "Create New Role".
    1. Enter semantically helpful role name, e.g., "aws-codedeploy-service-role", in field shown, and then click "Next Step".
    1. In "AWS Service Roles" section, scroll down to "AWS CodeDeploy" and then click "Select".  
    1. Select "AWSCodeDeployRole" from resulting table of policy choices, and then click "Next Step"; review, and then click "Create Role".
    1. If desired, inspect role by clicking its row in resulting list.
       - **NOTE** that orange cube icon indicates role policy is defined and managed by AWS, so that any AWS will update the policy automatically to supply such new privileges as may be needed by CD.
       - **SEE** actual JSON for the role by clicking "Show Policy".
    1. **MAKE NOTE** of this name for later use in defining app in CodeDeploy.
    1. To create S3 role to attach to instances as they are deployed, go back to "Roles" in l.hand menu and click "Create New Role" as before.
    1. Enter, e.g., "aws-codedeploy-instance-profile" as semantically helpful name for role to allow EC2 instances to deploy from S3.
    1. Select "Amazon EC2" from list of available roles, and enter "CodeDeploy" into search window to filter in relevant policies.
    1. Select "AmazonEC2RoleforAWSCodeDeploy" and click "Next Step".
    1. Review results and then click "Create Role"; as before, click role name to review attached policies.
    1. Again, **MAKE NOTE** of role name for use in creating CD instances later.
  1. **PREPARE INSTANCES FOR CODEDEPLOY USE**
    1. To create auto-scaling group for this tutorial, navigate to EC2 console and choose "Launch Configurations" from l.hand menu.
    1. Click "Create launch configuration" and select "Amazon Linux" from list of available images; allow default to image size of "t2.micro".
    1. Click "Next: configure details".
    1. Supply semantically helpful name, e.g., "codedeploy-launch-config", and select IAM instance profile previously created.
    1. Click "Advanced Details" to access "User Data" field and enter self-supplied shell script.
    1. Open a search window and type, e.g., "install AWS CodeDeploy agent" to get a link to [documentation](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-run-agent-install.html) containing ready-made code.
    1. Click link for Amazon Linux install, and copy code snippets into "User Data" window as shown, omitting `sudo` command throughout and substituting appropriate region for `<bucket.name>`
       - **NOTE** that the tutorial's instructions are for shell scripting, while the documentation given is for the terminal
       - `-y` flags must be added to commands after which prompts may be returned, i.e., `yum update`, `yum install ruby` and, presumably, `yum install wget`
       - **ALSO NOTE** that tutorial shows script installing aws-cli, while current docs show wget instead, although tutorial does include the `wget` command used to fetch the code from S3
       - **TUTORIAL EXAMPLE:**
       ```bash
        #!/bin/bash
        yum -y update
        yum -y install ruby
        yum -y install aws-cli
        cd /home/ec2-user
        wget https://aws-codedeploy-us-west-2.s3.amazonaws.com/latest/install
        chmod +x ./install
        ./install auto
       ```
    1. Click "Next: Add Storage", accept defaults and click "Next: Configure Security Group".
    1. Click "Select an existing security group" and choose the one allowing HTTP/SSH traffic from the world on ports 80/22 respectively.
      - **NOTE** that this configuration is insecure and appropriate only for demo purposes; in production HTTPS with a secure cert would be required.
    1. Click "Review" and ignore warning with deference to foregoing **NOTE.**
    1. Click "Create and launch configuration", select previously used private key and click "Create launch configuration".
    1. Click "Create an Autoscaling Group using this launch configuration".
    1. Enter a semantically helpful name, e.g., "codedeploy-demo-asg"; accept defaults for 1 starter instance, for VPC, and first selection subnet.
    1. Click "Next: Configure scaling policies" and accept default to keep group at initial size; click "Review".
    1. Review details and click "Create Auto Scaling group"; ASG now created.
    1. Click "Close" to return to ASG console; when new ASG appears in list, navigate back to main EC2 Dashboard to check on instance.
    1. Click into "Running Instances", where a new, unnamed instance will appear, possibly still spinning up; copy public IP from its "Description" tab, open a shell and ssh using `ssh ec2-user@<public.ip> -i ~/.ssh/your-demo.pem` as in, e.g., the Elastic Beanstalk lesson.
    1. `sudo service codedeploy-agent status` should return log message like `The AWS CodeDeploy agent is running as PID <process-id-no>`
  1. **CREATE CODEDEPLOY APP**
    1. To create CD app for this tutorial, navigate to CodeDeploy dashboard and click "Get Started Now" if no pre-existing deployment groups exist.
    1. Select "Custom Deployment" radio and click "Skip Walkthrough".
    1. Enter helpful names for `art_gallery_app` and `art_gallery_deployment_group` and allow default to "In-place deployment" (not covered in tutorial).
    1. **NOTE** list of pre-reqs, all of which have been taken care of in the foregoing.
    1. No free-floating EC2 instances in this deployment group, so just select "Auto-Scaling Group" in "Tag Type" drop-down for selecting search tag type and then name of ASG in resulting follow-on dropdown.
    1. Select "CodeDeployDefault:OneAtATime" for "Deployment Config", and leave "Advanced" options to create Triggers and Alarms blank for now.
    1. Under "Rollbacks", check "Roll back when a deployment fails".
    1. Under "Service Role" select name of previously defined CodeDeploy Service Role (careful not to get the similarly named demo EC2 instance profile), and click "Create Application".
  1. **AWS-CLI INSTALL AND CONFIG** 
    1. `pip install awscli` (may need `sudo`)
    1. Check success with `aws`; should get usage message.
    1. To get AWS secret key and secret access key navigate back to AWS GUI, click into IAM console, and then click "Users" in l.hand menu.
    1. Click into your user and go to "Security credentials" tab; if more than one listed, leave no more than one undeleted, and then click "Create access key"; modal comes up with warning that if you don't download you will not be able to view key again; click "Show" and return to terminal.
    1. `aws configure`
    1. Paste in AWS Access key ID at first prompt.
    1. Paste in Secret access key at second prompt.
    1. At next prompt, give `us-west-2` as region (or whichever one you've been working in).
    1. At next prompt, set, e.g., `json` as default output format.
    1. `aws iam get-user` to verify; output should display your correct AWS username.
  1. **APPSPEC & DEPLOY** 
    1. **DEMO `appspec.yml`:**
    ```javascript
        version: 0.0
        os: linux
        files:
          - source: codedeploy/config/ruby-codedeploy-demo-nginx.conf
            destination: /etc/nginx/conf.d
          - source: codedeploy/config/nginx.conf
            destination: /etc/nginx
        hooks:
          BeforeInstall:
            - location: codedeploy/scripts/stop_unicorn.sh
              runas: root 
            - location: codedeploy/scripts/configure_nginx.sh
            - location: codedeploy/scripts/install_dependencies.sh
          AfterInstall:
            - location: codedeploy/scripts/start_unicorn.sh
              runas: root 
            - location: codedeploy/scripts/start_nginx.sh
              runas: root 
    ```
      - `files` section moves two files as indicated
      - `hooks` section specifies location and runtime instructions for three scripts to run before installation and two to run after
    1. **NOTE** how `codedeploy/scripts/configure_nginx.sh:4` sets up symlink using env vars `$DEPLOYMENT_GROUP_ID` and `$DEPLOYMENT_ID` to ensure consistent app root location across deployments where actual deploy folder will change each time
        - Could have written dynamic script to locate actual folder
        - Symlink simpler and more direct, allows use of single pathname, `/opt/current-deployment`, as app root in `codedeploy/config/ruby-codedeploy-demo-nginx.conf`, thus simplifying tracking
    1. **DEPLOY APP**
      1. **NOTE** AWS-CLI command for CodeDeploy, `aws deploy` (EC2 is `aws ec2`, etc.; only Elastic Beanstalk missing, as EB has its own CLI)
      1. **DEMO DEPLOY SCRIPT:**
      ```bash
        aws deploy create-deployment \ 
         --application-name cd_artgall_app \ 
         --deployment-config-name CodeDeployDefault.OneAtATime \ 
         --deployment-group-name cd_artgall_deployment_group \ 
         --description "CodeDeploy demo" \ 
         --github-location repository=txtincorporated/AWSdemo,commit-Id=$(git rev-parse HEAD)
      ```

#### TERMS & CONCEPTS
  * **application:**  in CodeDeploy terms, start by definining application, which refers to a...
  * **deployment group:**  set of EC2 instances to which CodeDeploy should deploy de code; may consist of any combination of autoscaling groups, individual EC2 instances bearing a given tag value, and in-house instances; must merely have CD Agent installed and (as described above) correct CD instance profile attached
  * **deployment configuration:**  
    - also specified by app; simple directive for rolling out new code to your instances...
      - one at a time
      - half your fleet at once
      - all at once
    - unique to CodeDeploy
    - definable as any combo of the following:
      - auto-scaling groups
      - EC2 instances (specify your own tag values to ID each)
      - local, on-site instances (a.k.a., your in-house servers; must be registered manually beforehand in order to add to deployment group)
  * **deployment:**  represents a push of one app revision to the specified deployment group
  * **CodeDeploy Agent** software which must be installedto configure instances to work with CD 
    - available from region-specific, publicly-available S3 bucket (see documentation for your region's location)
    - runs as service, meaning that it launches automatically upon instance startup, whereupon it polls the CD Service to check for new deployment "jobs"; to wit, the instance pulls its own code rather than having it pushed by CD
    - **NOTE** that you'll probably need to marshall, e.g., CloudFormation, to help with building and provisioning infrastructure for these deployments, as CD, remember, does none of this the way EB and OW do.
  * **deployment job** specifies an application revision located either in S3 or Git remote and contains the following:
      - application code, of course
      - all configuration files needed by app
      - the sacred `appspec.yml`
  * **`appspec.yml`:**  
      - written in YAML, obviously
      - light configuration mgt. functionality
      - sections covering script execution, and file mvt. and permissions
      - CodeDeploy lifecycle event tags
  * **EC2 launch configuration:**  as described in previous lesson, a full set of EC2 instance parameters describing a launch-ready EC2 instance, used to auto-deploy multiple copies of it on demand

