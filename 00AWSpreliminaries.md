# Code 401 
## Notes 2/2/17

### AWS PRELIMINARY INSTALLATIONS AND CONFIG:

#### QUESTIONS & COMMENTS
1. **AWS CLI INSTALL OS X [(Linux & Windows HERE)](http://docs.aws.amazon.com/cli/latest/userguide/installing.html#install-with-pip):** 
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

1. **INITIAL SETUP:** 
  1. Sign up for AWS at [aws.amazon.com](https://aws.amazon.com).
    - To pay only nominal costs, sign up for basic service and follow the teardown instructions carefully in the lynda.com tutorial.
  1. Set up or clone a GitHub repo with the code you want to use for your app, and one for the deployment.
    - **NOTE** that both [a client](https://github.com/brandon-rich-aws-deployment-course/art_gallery) and [an API connection](https://github.com/brandon-rich-aws-deployment-course/cloudformation-demo) repo are available to clone if you'd like to configure along step by step with the tutorial.

  1. If configuring the sample repos to follow along with the tutorial: 
    - **Install the bundle** in Ruby's answer to `npm install` -- running from the Gemfile, Ruby's answer to package.json -- [the `bundle` command](http://bundler.io/v1.14/man/bundle-install.1.html) installs the project dependencies listed in the Gemfile and sets some roles and test script commands.
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
    

#### TERMS & CONCEPTS
  * **named profile:**  on AWS, the credentials for a specific deployment or other AWS data facility you manage, configurable in a single pair of documents, `~/.aws/config` and `~/.aws/credentials`, once credentials have been obtained for each given facility

