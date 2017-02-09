# Code 401 
## Notes 2/2/17

### COURSE NOTES:  [Lynda.com INSTALLING AWS CLI](https://www.lynda.com/Amazon-Web-Services-tutorials/AWS-Command-Line-Interface-CLI/529634/572685-4.html)

#### QUESTIONS & COMMENTS
1. **[OS X INSTALL -- Linux & Windows at this link](http://docs.aws.amazon.com/cli/latest/userguide/installing.html#install-with-pip):** 
  1. Check python version in Terminal:  run `python --version` (^2.6.5 required).
  1. Check for pip in Terminal: run `pip --help` -- [install](http://docs.aws.amazon.com/cli/latest/userguide/installing.html#install-pip) if necessary.
    - Run `curl -O https://bootstrap.pypa.io/get-pip.py` to download pip pkg.
    - Run `sudo python get-pip.py` to compile with python.
    - Confirm install location: `which pip` (s.b. `/usr/local/bin/pip`).     
  1. Install AWS cli using pip:  run `sudo pip install awscli`.
    - If deprecation error: run `sudo pip install awscli --ignore-installed six`.
    - Check for updates: `sudo pip install --upgrade awscli`.
    - Check for successful install: `aws help`.
1. **NOTE:** that AWS stores config options under `~/.aws` once configuration is set up
    - AWS supports named profiles, making junk defaults a good idea if you're managing a number of different facilities
    - Set up main config: run `aws configure` and type 'None' when prompted for each value; prompt returned upon completion
    - Multiple credential and config sets can be added later by using an editor to add them to `~/.aws/config` and `~/.aws/credentials` as shown in the tutorial

#### TERMS & CONCEPTS
  * **named profile:**  on AWS, the credentials for a specific deployment or other AWS data facility you manage, configurable in a single pair of documents, `~/.aws/config` and `~/.aws/credentials`, once credentials have been obtained for each given facility

