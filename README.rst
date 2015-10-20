Provectus DevOps Task Solution
==============================
Task is solved using two core components:

#. Cloudformation (used to launch infrastructure stack)
#. Ansible (used to deploy components)

General Information & Components
--------------------------------

The stack structure was implemented using the following components:

- EC2 instance for Admin App
- EC2 instance for Blog App
- ElastiCache cluster as Redis Server
- RDS (postgres) as Database
- Detailed Monitoring for EC2 instances as Monitoring Server
- Nginx as Proxy Server
- ELB is used in front of Blog App instance

Nginx is not installed on a dedicated server, it is installed on both Admin App and
Blog App instances. Nginx configuration on Blog App instance has rule to proxy requests
to the Admin App instance based on `/admin` endpoints.

Ansible discovers Admin App and Blog App instances' ip addresses using custom AWS Tags,
*Environment* and *Component*.

Deployment scripts prepare insances for Django-based app deployments by creating
specific users, groups, creating virtual environments and installing Django in there.

Prerequisites
-------------

#. ansible 1.9+, tested on 1.9.4
#. boto 2.38+, tested on 2.38
#. ssh keypair named **provectus-test-key** created in AWS EC2 in **eu-west-1** region [1]_
#. AWS credentials set in environment variables or boto configuration file [2]_

How To Launch Infrastructure
----------------------------

**Note** that you **must** change `ansible_python_interpreter` variable's value in `environments/provision`
file according to your python interpreter which has `boto` installed before running the following command.

``$ ansible-playbook -i environments/provision actions/provision.yml``

How To Run Deployment
---------------------

``$ ansible-playbook -i environments/aws actions/deploy.yml -u admin -s --private-key path/to/provectus-test-key.pem``

Directory Structure
-------------------

::

    provectus $ tree -L 3
    .
    ├── actions                            # directory contains ansible playbooks
    │   ├── configure.yml                  # playbook used to execute `configure` actions
    │   ├── deploy.yml                     # playbook used to execute both `install` and `configure` actions
    │   ├── install.yml                    # playbook used to execute `install` actions
    │   ├── provisioning                   # helper directory contains Cloudformation stack json
    │   │   └── provectus-test-stack.json
    │   └── provision.yml                  # playbook used to execute `provision` actions
    ├── ansible.cfg                        # ansible's configuration file
    ├── components                         # directory contains ansible roles
    │   ├── admin_app                      # several actions defined for each component
    │   │   ├── configure                  # all files, templates, tasks needed to execute `configure` action
    │   │   │   ├── defaults
    │   │   │   ├── handlers
    │   │   │   ├── tasks
    │   │   │   └── templates
    │   │   └── install                    # all files, templates, tasks needed to execute `install` action
    │   │       ├── defaults
    │   │       ├── meta
    │   │       └── tasks
    │   ├── blog_app
    │   │   ├── configure
    │   │   │   ├── defaults
    │   │   │   ├── handlers
    │   │   │   ├── tasks
    │   │   │   └── templates
    │   │   └── install
    │   │       ├── defaults
    │   │       ├── meta
    │   │       └── tasks
    │   └── common                         # directory contains common roles
    │       ├── nginx
    │       │   └── install
    │       └── virtualenv
    │           └── install
    ├── environments                       # directory contains ansible's inventory files
    │   ├── aws                            # this environment is provisioned dynamically
    │   │   ├── ec2.ini                    # that's why ec2.py dynamic inventory script is used
    │   │   ├── ec2.py
    │   │   ├── group_vars                 # this directory contains variables specific for this environment
    │   │   └── static_inventory           # this file puts generated host groups into groups with predefined names
    │   └── provision                      # this is static environment file
    └── README.rst

    15 directories, 11 files

Footnotes
---------

.. [1] http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
.. [2] http://docs.ansible.com/ansible/guide_aws.html#authentication
