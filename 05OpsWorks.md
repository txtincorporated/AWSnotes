# Code Fellows Alumni Group
## Notes 2/17/17

### [Lynda.com AWS OPSWORKS](https://www.lynda.com/Amazon-Web-Services-tutorials/OpsWorks-key-concepts/502654/566946-4.html?srchtrk=index%3a5%0alinktypeid%3a2%0aq%3aaws%0apage%3a1%0as%3arelevance%0asa%3atrue%0aproducttypeid%3a2)

#### QUESTIONS & COMMENTS
1. **NOTE** strong contrast between OpsWorks and Elastic Beansatalk
    - EB heavily prescriptive; predefined infrastructure choices; as PaaS, takes care of more of the stack decisions for you
    - OpsWorks allows granular control in describing your architecture
1. **HOW** is OpsWorks set up to work?
    - Define stack in given region; just a container for the app to be built
    - Defined in layers, e.g., an app server layer to handle the app, a data layer consisting of RDS instances, and an off-the-shelf 'load balancer' layer supplied by attaching the stack to an existing ELB 
    -  **CREENT:**
  1. **NE:**
1. **HOW** 
  1. YOED 
    1. **CREENT:**
  1. **NE:**

#### TERMS & CONCEPTS
  * **stack:**  content and configuration of each N-tier stack layer and how they relate to one another
  * **layer:**  set of rules for configuring resources; tends to map well with layers in interior architecture
    - obviously, does nothing by itself; EC2 instance must be present to make it useful
    - provisioned with the software and configuration described in layer def.
    - add *ad hoc*, on a schedule or programmatically (e.g., according to load) 
  * **instance:**  Planing
  * **N-Tier architecture:**  web application architecture in which Load Balancer (distribution) layer operates on top of server-clone layer (web app 'server layer'; EC2 instances), in turn on top of database ('data layer')
    - **NOTE** that app servers and web servers sometimes separated into their own layers with their own respective hosts
  * **configuration management:**  principle that it's preferable to scripts as many server provisioning and maintenance tasks as possible; accordingly handles configuration as code
    - **SOME CONFIGURATION MANAGEMENT TOOLS;**
      - Puppet Labs
      - Chef
      - Ansible


