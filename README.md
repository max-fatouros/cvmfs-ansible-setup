# CVMFS Server Setup (using Ansible)

Set up CVMFS repositories on Stratum 0 machine, Stratum 1 (with reverse proxy) machine, forward caching proxy machine, and client machine.
Based on the following documentation.

1. [Official docs](https://cvmfs.readthedocs.io/en/stable/cpt-repo.html)
2. [CVMFS tutorial](https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/)



## Initial Steps
### Installation
This code uses Ansible version 2.16, as newer versions are not compatible with RHEL 8. This can be installed with pip:
```bash
pip install git+https://github.com/ansible/ansible.git@stable-2.16
```

### Setup
1. Add the Stratum 0, 1, proxy, and client machines to the list defined in the [`hosts.yaml`](https://gitlab.cern.ch/clange/snapshotter-benchmarks/-/blob/master/ansible/hosts.yaml?ref_type=heads) file, whose format is described [here](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html).

3. Test that things are working with the command
    ```bash
    ansible-playbooks ping.yaml
    ```

4. Many of the ansible command require `sudo` privileges. To enable sudo without the use of a password on a remote machine, first run the following command, enter the sudo password for the machine which will have its sudo password removed, then press enter.
    ```bash
    read -s PASS
    ```
    After this, run
    ```bash
    ansible-playbook -l <host> -e "ansible_sudo_pass=$PASS" remove_sudo-pass.yaml
    ```
    where `<hosts>` is the hostname (specified in `hosts.yaml`) of the machine whose sudo password we are removing.



## Usage
1. Configure `vars.yaml`
2. Run
    ```bash
    ansible-playbook add_repository.yaml
    ```
