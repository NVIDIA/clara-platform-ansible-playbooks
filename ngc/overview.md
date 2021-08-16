## Clara Ansible Installation Scripts

### 1. Install Ansible

To install Ansible follow the link [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

**Note:** This playbook requires Ansible 2.9 or higher, which can be installed via system package manager such as apt or via Python's package manager pip.

### 2. Installing Clara

#### 2.1 A note on the NVIDIA Driver
It is assumed the host system has a working NVIDIA driver prior to installing Clara. If your host does not have the NVIDIA driver, you can install the correct version for this version of clara via the `playbooks/driver.yml` file. To install the driver using Ansible, see section 2.2


#### 2.2 NVIDIA Driver installation via Ansible

The recommended version of the NVIDIA driver for this version of Clara is specified in the `playbooks/driver.yml`. If you already have this driver, skip this step and continue to the Clara installation section below.

**Note:**  It is recommended to only install the NVIDIA driver via Ansible if you do not already have an NVIDIA driver present. Once the driver is instaled or uninstalled using Ansible you will be prompted to reboot the machine. If you are installing Clara locally, Ansible will not force a reboot and you will need to run the reboot command manually. It is required to reboot the system after installation to ensure successful installation of Clara.

Before running the driver installation playbook, edit the `playbooks/driver.yml` file and ensure the name of the system user that Ansible should login as is set. 

To install the driver locally, run:
```
ansible-playbook -K driver.yml 
```

To install the driver on remote hosts, change the value of `hosts` to `all` in `/playbooks/clara.yml`, set the IP address(es) of the remote machine(s) in `/playbooks/clara_hosts` then run the following command:
```
ansible-playbook -K driver.yml -i clara_hosts
```

#### 2.3 Uninstalling NVIDIA Driver via ansible

**Note:** Due to multiple driver installation methods, do not attempt to run the driver uninstall playbook if the driver was not instaled by Ansible.

Before running the driver installation playbook, edit the `playbooks/driver.yml` file and ensure the name of the system user that Ansible should login as is set. 

To uninstall the driver locally, run:
```
ansible-playbook -K driver.yml -e 'execute=uninstall'
```

To uninstall the driver on remote hosts, change the value of `hosts` to `all` in `/playbooks/clara.yml`, set the IP address(es) of the remote machine(s) in `/playbooks/clara_hosts` then run the following command:
```
ansible-playbook -K driver.yml -e 'execute=uninstall' -i clara_hosts
```


#### 2.3 Installing Clara locally

To run the playbooks locally (on the same system with Ansible), first, edit `playbooks/clara.yml` and set the following values:

- `clara_user` - Set this to the non-root user of the machine
- `ngc_api_key` - If you do not already have an NGC account/key see [this page](https://docs.nvidia.com/ngc/ngc-overview/index.html#registering-activating-ngc-account) to get started

Once the values are set, `cd` into the `/playbooks` directory and run:
`ansible-playbook -K clara.yml`

**Note:** Editable values can be configured in the `clara.yml` file (eg. `ngc_org: ea-nvidia-clara` for early access users, additional Clara components, Kubernetes networking settings)

#### 2.3 Installing Clara on remote hosts

##### 2.3.1 Setting SSH Connectivity

Ansible works on remote machines over SSH connection. To install Clara onto remote machines, create a local SSH key and make sure it is copied to all remote hosts in which you aim to install Clara on.

For example, you can generate a key locally, then copy it to the remote target host using the folowing two commands:
```
# ssh-keygen
# ssh-copy-id -i ~/.ssh/mykey user@targethost
```

#### 2.3.2 Run install playbook
To run the playbooks onto one or more remote hosts from your local machine, first, add the IP address(es) of the remote machine(s) in the file `playbooks/clara_hosts`. Then edit `playbooks/clara.yml` and set the following values:

- `hosts` - Set this to `all`
- `clara_user` - Set this to the non-root user of the machine
- `ngc_api_key` - If you do not already have an NGC account/key see [this page](https://docs.nvidia.com/ngc/ngc-overview/index.html#registering-activating-ngc-account) to get started

Once the values are set, `cd` into the `/playbooks` directory and run:
`ansible-playbook -K clara.yml`

**Note:** Editable values can be configured in the `clara.yml` file (eg. `ngc_org: ea-nvidia-clara` for early access users, additional Clara components, Kubernetes networking settings)

### 3 Uninstalling Clara

To uninstall Clara locally, run: 
```
ansible-playbook -K clara.yml -e 'execute=uninstall' 
```

To uninstall Clara on remote hosts, run: 
```
ansible-playbook -K clara.yml -e 'execute=uninstall' -i clara_hosts
```

This will prompt to remove:
- `.clara` data directory
- Helm
- Kubernetes & Kubernetes Binaries
- NVIDIA Docker
- Docker

#### Tasks

The Clara Ansible playbook will:
* Install NVIDIA Driver (optional)
* Install Docker CE
* Install NVIDIA Docker 2 (optional)
* Turn Swap space off
* Install OpenShift Python libraries to enable the `k8s` Ansible module
* Install and Kubernetes
* Install Helm3
* Install Clara CLI Binaries and Components
* Pull Clara Components from [NGC](https://docs.nvidia.com/ngc/ngc-overview/index.html) (requires NGC API key)
* Start Clara

#### Assumptions

* The playbook is designed for Ubuntu on the `x86_64` architecture, however, the user may update or enhance the playbooks to fit their target systems.