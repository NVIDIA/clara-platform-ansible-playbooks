## Clara Ansible Installation Scripts

### 1. Install Ansible In Control Node

To install Ansible in the control node follow the link [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

**Note:** This playbook requires Ansible 2.9 or higher, which can be installed via system package manager such as apt or via Python's package manager pip.

### 2. Configuring Remote Hosts

#### 2.1 Setting Up Hosts/Groups for installing Clara on remote hosts

The file `playbooks/clara_hosts` includes an inventory file, with variables which are shared amongst all hosts. You can run the included hosts file by passing `-i clara_hosts` or update `/etc/ansible/hosts` with the included variables and list of preferred hosts on which you want to perform the Clara installations. The playbooks assume that there is a group name `clara_hosts` each with a user `nvidia`, you may choose to change these entries in the playbooks. (See [https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)).


#### 2.2 Setting SSH Connectivity

Create a local SSH key and share place it in all the hosts in which you aim to perform the deployment.

For example, you can generate a key and copy it using the folowing:
```
# ssh-keygen
# ssh-copy-id -i ~/.ssh/mykey user@host
```

#### 2.3 Setting Up Hosts/Groups for installing Clara locally

To run the playbooks locally, within the `playbooks/clara_hosts`, instead of passing the IP address of a remote system, add `localhost ansible_connection=local`. This will allow you to run the playbook locally. You will also need to pass the flag `-ask-become-pass` with all `ansible-playbook` commands to give Ansible the needed sudo permissions to install packages. For example: `ansible-playbook -K install-clara.yml --ask-become-pass`

**Note:** The playbook`install-nvidia-driver.yml` forces a reboot after installation to ensure the driver is set up correctly. If installing locally, after the system reboot, continue the installation with the next playbook `install-clara-prereqs.yml` and `install-clara.yml`

## 3. Installation Options

### 3.1 Installing NVIDIA Drivers

To install NVIDIA Drivers
```
ansible-playbook -K install-nvidia-driver.yml -i clara_hosts
```
At the prompt enter the `sudo` password for the host machine. At the end of the installation the target host(s) will be restarted for the changes to take effect.

You may override the default Nvidia driver version under `roles/nvidia-driver/defaults/main.yml` using the `-e` command line option as
```
ansible-playbook -K install-nvidia-driver.yml -e 'nvidia_driver=nvidia-driver-XXX' -i clara_hosts
```
where `XXX` is your preferred version.

**Note:** Do not run this playbook if an NVIDIA driver is present on your machine, as it will not remove the existing driver.

### 3.2 Installing Clara Prerequisites

To install Clara prerequisites run
```
ansible-playbook -K install-clara-prereqs.yml -i clara_hosts
```

#### Assumptions

* It is assumed that before the Clara prerequisites playbook is run, a compatible version of the NVIDIA drivers exists in all remote machines affected.
* The playbook is designed for Ubuntu on the `x86_64` architecture, however, the user may update or enhance the script to fit their target systems.

#### Tasks

The prerequisites installation script will:
* install Docker CE
* install NVIDIA Docker 2
* turn Swap space off
* install OpenShift Python libraries to enable the `k8s` Ansible module
* install Kubernetes (`kubeadm`, `kubectl`, `kubelet`)
* install Helm

### 3.3 Installing Clara Components

To install basic Clara components run
```
ansible-playbook -K install-clara.yml -i clara_hosts
```

Clara components available for insallation:
- `platform`:
- `dicom`:
- `monitor`:
- `console`:
- `render`

By default only `platform` and `dicom` are installed, however, the user may override the default variable `clara_components` as
```
ansible-playbook -K install-clara.yml -i clara_hosts -e 'clara_components=["platform", "dicom", "render"]'
```
or by updating the default value of `clara_components` in `ansible/playbooks/roles/clara-components/default/main.yml`.
