# cpp-ansible-test-environment

This repo is an Ansible project with a Vagrantfile to enable local development of Ansible roles/playbooks. It is setup to mimic Ansible projects within Crime using CentOS7 as the machine OS, Ansible 2.9 & a Python PIP requrements file lifted from IDAM.ansible. 


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


### Create Python3 Virtual Environment

Ansible is written in python so it needs a python installation to work. Ansible 2.9 is old and as such the Python versions its compatible with have since become deprecated.

The highest compatible Python version with Ansible 2.9 is Python 3.8. Mac installation of Python is usually through Homebrew however Python 3.8 has now been disabled. The easiest way to install Python 3.8 with this in mind is using Pyenv.


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


#### Using Pyenv

```bash
pyenv install 3.8.20
pyenv global 3.8.20
```

This installs the last minor version on Python 3.8 and makes it globally active for use. Pyenv symlinks to **python** and not **python3**.


#### Create the venv

Create the virtualenv and activate it:
```bash
python -m venv ./venv/
source ./venv/bin/activate
```

You'll now see a preprend in your shell showing we've loaded into our venv. Something similar to this:
```bash
(venv) user.name
```
Exit the venv at any time by running **deactivate**


#### Update Pip

Pip is the package manager for Python. Since Python 3.8 the repo used to pull python packages has moved. Tell pip the new repo domain and to update itself:

```bash
pip install --trusted-host pypi.org --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --upgrade pip
```


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