# CVMFS Server Setup (using Ansible)

Set up CVMFS repositories on a Stratum-0 machine, Stratum-1 (with reverse proxy) machines, forward caching proxy machines, and client machines.
Based on the following documentation.

1. [Official docs](https://cvmfs.readthedocs.io/en/stable/cpt-repo.html)
2. [CVMFS tutorial](https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/)


## Initial Steps
### Installation

Install this collection with
```bash
ansible-galaxy collection install git+https://github.com/max-fatouros/cvmfs-ansible-setup.git
```

### Setup
1. Create a [`hosts.yml`](hosts.yml) file. 
The link to the left shows an example of this file filled out.
Add the stratum-0, stratum-1, proxy, and client machines to it. 
The format of this file is described [here](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html).

2. Create a copy of this [`ansible.cfg`](ansible.cfg) file.


3. With both the `hosts.yml` and `ansible.cfg` files in your working directory.
Test that things are working with the command
    ```bash
    ansible-playbook cvmfs.setup.ping
    ```

4. Many of the ansible commands require `sudo` privileges. 
    If you would like to disable the sudo password on the remote machines so that the playbooks in this collection can be ran without a password, first run the following command, enter the sudo password for the machine which will have its sudo password removed, then press enter.
    ```bash
    read -s PASS
    ```
    After this, run
    ```bash
    ansible-playbook -l <host> -e "ansible_sudo_pass=$PASS" cvmfs.setup.remove_sudo-pass
    ```
    where `<hosts>` is the hostname (specified in `hosts.yaml`) of the machine whose sudo password we are removing.



## Usage
1. Run
    ```bash
    ansible-playbook cvmfs.setup.all
    ```

    this is equivalent to running
    ```bash
    ansible-playbook cvmfs.setup.stratum_0
    ansible-playbook cvmfs.setup.stratum_1s
    ansible-playbook cvmfs.setup.proxies
    ansible-playbook cvmfs.setup.clients
    ```
    after running `cvmfs.setup.all`, any of the individual playbooks above case be re-ran if needed. However, they all rely on `cvmfs.setup.stratum_0` having been ran once in order for the stratum-0 public keys to be copied over properly.
