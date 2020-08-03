Running Ansibullbot
===================

How to deploy and run ansibullbot for a repository or collection.

Requirements
------------

AWS account
Github account
Python2.7 environment

Setup
-----

* Clone this repo (with the correct branch) to your local machine.
* Setup AWS authentication using either a
[boto profile](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html)
or environment variables. See the
[Ansible AWS Guide](https://docs.ansible.com/ansible/devel/scenario_guides/guide_aws.html#authentication)
for more details.
* Copy `examples/ansibullbot.cfg` to the root of the repository.
* Fill in your `repo` value(s), and if applicable, your `crepo` values (for collections).
* Edit `ansibullbot/triagers/ansible.py` to update the `repo_data` dictionary with your branch information
and, if applicable, Shippable project details.

If you are not a member of Ansible Engineering and will be using your own Github account to operate the
bot, you will need to also define your own `host_vars` for the vaulted `ansibullbot_*` credentials
contained in `playbooks/group_vars/ansibullbot.yml`

Deploying to AWS
----------------

This repo ships with playbooks that will deploy the bot to an AWS EC2 instance where it can be run as a daemon.
This step is optional, the bot can be run from any host the operator prefers.

Create a new host_vars file under `playbooks/host_vars/` and populate host-specific variables.  See the existing
files in this directory for examples, common variables to set are:

```yaml
botinstance_name: ansibullbot-mycollection
botinstance_region: us-west-1
botinstance_volume_size: 200
botinstance_type: t3.xlarge
botinstance_ssh_keys:
  - name: mykey
    key_material: ssh-rsa 12345567890aaaaaaaaaaaa...
ansibullbot_fqdn: ansibullbot-mycollection.eng.ansible.com
```

Add your host to `hosts.yml`.

```shell script
ansible-playbook -i hosts.yml --limit ansibullbot-mycollection.eng.ansible.com setup-ansibullbot.yml
```

Running ansibullbot
-------------------

Ansibullbot can be run ad-hoc on the command line or as a daemon.  It is recommended to run the bot
in a virtual environment.  The below assumes you have setup your ansibullbot.cfg as described in the previous
section, and made the necessary edits to `ansibullbot/triagers/ansible.py`.

```shell script
$ cd ansibullbot/
$ python2.7 -m virtualenv .ansibot
$ source .ansibot/bin/activate
$ python2.7 -m pip install -r requirements.txt
$ python2.7 -m pip install epdb  ## Ansibullbot has breakpoints, epbd must be installed if one it hit
$ python2.7 triage_ansible.py --collection ansible-collections/mycollection --repo ansible-collections/mycollection
```

See `python2.7 triage_ansible.py --help` for more options, including daemonize, dry-run, and debug flags.