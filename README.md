# cpp-ansible-test-environment

This repo is an Ansible project with a Vagrantfile to enable local development of Ansible roles/playbooks. It is setup to mimic Ansible projects within Crime using CentOS7 as the machine OS, Ansible 2.3 & a Python PIP requrements file lifted from IDAM.ansible. 


## Directory Structure
* [group_vars/](./group_vars/) - group variable files
* [lookup_plugins/](./lookup_plugins/) - lookup plugins folder
  * [lookup_plugins/vault.py](./lookup_plugins/vault.py) Hashicorp Vault lookup plugin script      
* [roles/](./roles/) - target directory for role installation
  * [roles/requirements.yaml](./roles/requirements.yaml) - requirements file used to declare remote roles to download using ansible-galaxy command
* [test/](./test/) - stuff used for vagrant testing
  * [test/centos7-base.repo](test/centos7-base.repo) - EOL repo config used in pre-tasks to get package manager working
* [ansible.cfg](./ansible.cfg) - ansible global config file
* [host.yaml](./hosts.yaml) - ansible inventory file. Sets SSH connection settings too
* [pip.requirements](./pip.requirements) - pip requirements file for installing constrained package versions in venv to match Crime environment
* [playbook.yaml](./playbook.yaml) - ansible playbook that gets executed
* [Vagrantfile](./vagrantfile) - vagrant config


## Local Environment Setup

Your local environment will need setting up to use the repo by performing the following:


### Create Python2 Virtual Environment

Ansible is written in python so it needs a python installation to work. Ansible 2.3 is old and as such the Python versions its compatible with have since become deprecated.

Ansible 2.3 is compatible with Python 2 and Python 3 however we want to match the Python version used in the main Crime PlatOps Jenkins instance. This is Python 2.7.15.

Mac installation of Python is usually through Homebrew however Python 2.7.15 is disabled. The easiest way to install this is using Pyenv.


#### Installing Pyenv

Pyenv is a wrapper for python, such as tfswitch/tfenv are a wrapper for Terraform. It compiles python from source so allows us to install older versions as well as switch between versions.

Install Pyenv with brew:
```bash
brew update
brew install pyenv
```

- For **bash**:

    Stock Bash startup files vary widely between distributions in which of them source
    which, under what circumstances, in what order and what additional configuration they perform.
    As such, the most reliable way to get Pyenv in all environments is to append Pyenv
    configuration commands to both `.bashrc` (for interactive shells)
    and the profile file that Bash would use (for login shells).

    First, add the commands to `~/.bashrc` by running the following in your terminal:

    ```bash
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
    echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
    echo 'eval "$(pyenv init -)"' >> ~/.bashrc
    ```

    Then, if you have `~/.profile`, `~/.bash_profile` or `~/.bash_login`, add the commands there as well.
    If you have none of these, add them to `~/.profile`.

    * to add to `~/.profile`:
      ```bash
      echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
      echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
      echo 'eval "$(pyenv init -)"' >> ~/.profile
      ```

    * to add to `~/.bash_profile`:
      ```bash
      echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
      echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
      echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
      ```

- For **Zsh**:

  Run the following to add the commands to `~/.zshrc`:
  ```bash
  echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
  echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
  echo 'eval "$(pyenv init -)"' >> ~/.zshrc
  ```

Then reload your shell.

#### Installing Pyenv Virtualenv

Pyenv-virtualenv is a pyenv plugin that provides features to manage virtualenvs environments for Python.

Install through brew:

```bash
brew install pyenv-virtualenv
```

Run the following command:

```bash
eval "$(pyenv virtualenv-init -)"
```

#### Using Pyenv and Pyenv Virtualenv

```bash
pyenv install 2.7.15
pyenv global 2.7.15
```

This installs Python 2.7.15 and makes it globally active for use. Pyenv symlinks to **python**.

```bash
pyenv virtualenv 2.7.15 venv-2.7.15
```

This will create a virtualenv based on Python 2.7.15 under $(pyenv root)/versions in a folder called venv-2.7.15. $(pyenv root) defaults to .pyenv hidden folder in the user's home directory.

```bash
pyenv virtualenvs 
```

This will list the virtualenvs available to pyenv. Your virtual environment venv-2.7.15 should be listed as available.

```bash
pyenv activate venv-2.7.15
```

This will activate the python virtual environment.

You'll now see a preprend in your shell showing we've loaded into our venv. Something similar to this:
```bash
(venv-2.7.15) user.name
```
Exit the venv at any time by running **source deactivate**

#### Install packages using pip.requirements

Now install the required packages with the provided pip.requirements file. This will install Ansible and its dependencies. It can take a few minutes to complete, be patient:

```bash
pip install -r pip.requirements
```

### Install Virtualbox

The Vagrantfile in the repo is configured to use the VirtualBox driver. At the time of writing (29/10/2024) the driver is compatible only up to version 7.0 of Virtualbox but the MOJ Self Service is a newer version than this.

Install VirtualBox 7.0 by downloading it from [old builds](https://www.virtualbox.org/wiki/Download_Old_Builds_7_0) and running the installer in the .dmg


### Install Vagrant

Install the latest version of Vagrant using Homebrew

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/hashicorp-vagrant
```

### Optional Setup

#### Hashicorp Vault
If fetching secrets from Hashicorp Vault using the vault lookup plugin, configure your local env with the variables needed. For example, for Crime nonlive:

```bash
VAULT_ADDR=https://secret.mdv.cpp.nonlive
VAULT_SKIP_VERIFY=1
VAULT_TOKEN=<vault_token>
OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

## Using the repo

playbook.yaml is executed on our test instance. The pre-tasks in this allow the CentOS machine to pull packages & resolve DNS correctly.

### Ansible-Galaxy 

If pulling roles remotely, run ansible galaxy to pull them into the /roles/ directory:

```bash
ansible-galaxy install -r ./roles/requirements.yaml
```

### Vagrant
To run the playbook, run vagrant commands from the root directory (where the Vagrantfile is):

#### Bring the machine up
Mirror/package manager errors may be reported back when it has built - this is not an error and is fixed by the pre-taks in the playbook:
```bash
vagrant up
```

#### Provision the machine with the Ansible playbook

Ansible will run and show its output:
```bash
vagrant provision
```

#### Destroy the machine

Save resources on your local machine and destroy your VirtualBox VM when finished testing:
```bash
vagrant destroy
```


#### Test your things

This repo is a template to enable everyone to get a local test env going easier. Please use it as a skeleton when developing your own Ansible code.

Uncomment the verbosity line in vagrantfile to enable verbose output from Ansible

Use a branch to save your configuration so you can checkout to it again if needed in the future. Call the branch the name of the role being tested or something suitable