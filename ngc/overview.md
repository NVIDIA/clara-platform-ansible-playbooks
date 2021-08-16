## Clara Ansible Installation Scripts

### 1. Install Ansible In Control Node

To install Ansible in the control node follow the link [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

**Note:** This playbook requires Ansible 2.9 or higher, which can be installed via system package manager such as apt or via Python's package manager pip.

### 2. Configuring Remote Hosts

#### 2.1 Setting Up Hosts/Groups for installing Clara on remote hosts

The file `playbooks/clara_hosts` includes an inventory file, with variables which are shared amongst all hosts. You can run the included hosts file by passing `-i clara_hosts`. If you have an existing Ansible hosts file, update `/etc/ansible/hosts` with the included variables and list of preferred hosts on which you want to perform the Clara installation. The playbooks assume that there is a group name `clara_hosts`, however, all variables are configurable within the playbooks. (See [https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)).


#### 2.2 Setting SSH Connectivity

Ansible works on remote machines over SSH connection. To install Clara onto remote machines, create a local SSH key and make sure it is copied to all remote hosts in which you aim to install Clara on. You can skip this step if running Ansible on the same machine.

For example, you can generate a key locally, then copy it to the remote target host using the folowing two commands:
```
# ssh-keygen
# ssh-copy-id -i ~/.ssh/mykey user@targethost
```

#### 2.3 Setting Up Hosts/Groups for installing Clara locally

To run the playbooks locally, within the `playbooks/clara_hosts`, instead of passing the IP address of a remote system, uncomment the line `localhost ansible_connection=local`. For example: `ansible-playbook -K install-clara.yml`


## 3. Installing Clara

### 3.1 NVIDIA Driver information

**Note:** The `driver.yml` is included to install the recommended version of the NVIDIA driver matching the current release of Clara. It is recommended to only install the NVIDIA driver via Ansible if you do not already have an NVIDIA driver present. Once the driver is instaled or uninstalled, Ansible will ask if you want to reboot the machine, this step recommended to ensure that the driver is operational. If you are running this playbook on the same machine which the driver is to be installed, the playbook will quit at the final step (reboot). Once this is complete, please reboot your machine manually before proceeding to install Clara.

Before running the `driver.yml` playbook, edit the `driver.yml` file and add the name of the user on the machine that Ansible should login as. This presumes the steps outlined in section 2 are completed first.

You may override the default Nvidia driver version under `roles/nvidia-driver/defaults/main.yml` using the `-e` command line option eg:
```
ansible-playbook -K driver.yml -e 'nvidia_driver_version=XXX' -i clara_hosts
```
where `XXX` is your preferred version (eg. 460).


To uninstall the driver, run 
```
ansible-playbook -K driver.yml -e 'execute=uninstall' -i clara_hosts
```

### 3.2 Installing Clara Components

To install basic Clara components run the following command:

```
ansible-playbook -K clara.yml -i clara_hosts
```

Clara components available for installation:
- `platform`:
- `dicom`:
- `console`:
- `render`

The default Clara components are `platform` and `dicom`. This can be modified in the `clara_components` variable within `ansible/playbooks/clara_hosts` file.

#### Tasks

The Clara Ansible playbook will:
* Install NVIDIA Driver (optional)
* Install Docker CE
* Install NVIDIA Docker 2
* Turn Swap space off
* Install OpenShift Python libraries to enable the `k8s` Ansible module
* Install and start Kubernetes (`kubeadm`, `kubectl`, `kubelet`)
* Install Helm
* Install Clara CLI Binaries and Components
* Start Clara

#### Assumptions

* The playbook is designed for Ubuntu on the `x86_64` architecture, however, the user may update or enhance the playbooks to fit their target systems.

### 3.3 Uninstalling Clara

To uninstall Clara, run: 
```
ansible-playbook -K clara.yml -e 'execute=uninstall' -i clara_hosts
```

This will prompt to remove:
- `.clara` data directory
- Helm
- Kubernetes & Kubernetes Binaries
- NVIDIA Docker
- Docker
- NVIDIA Driver (optional)