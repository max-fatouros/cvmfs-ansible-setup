# CVMFS Server Setup (using Ansible)

Set up CVMFS repositories on a [Stratum-0](https://cvmfs.readthedocs.io/en/stable/cpt-repo.html#) machine, [Stratum-1](https://cvmfs.readthedocs.io/en/stable/cpt-replica.html) (with [reverse proxy](https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/03_stratum1_proxies/#313-configuring-apache-and-squid-proxy)) machines, [forward caching proxy](https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/03_stratum1_proxies/#32-setting-up-a-proxy) machines, and [client](https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/02_stratum0_client/#22-setting-up-a-client) machines.

The setup is based on the following documentation:

1. [Official docs](https://cvmfs.readthedocs.io/en/stable/cpt-repo.html)
2. [CVMFS tutorial](https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/)

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents:**

- [CVMFS Server Setup (using Ansible)](#cvmfs-server-setup-using-ansible)
  - [Initial Steps](#initial-steps)
    - [Installation](#installation)
    - [Setup](#setup)
  - [Usage](#usage)
  - [Testing](#testing)
    - [Installation / Setup](#installation--setup)
    - [Test execution](#test-execution)

<!-- markdown-toc end -->

## Initial Steps

### Installation

Install this collection with

```bash
ansible-galaxy collection install git+https://github.com/max-fatouros/cvmfs-ansible-setup.git
```

### Setup

1. Copy the [`hosts.yml`](hosts.yml) file.
Replace the hosts under the stratum-0, stratum-1, proxy, client, and cvmfs groups with the ones you want to use.
If you would like to not use stratum-1s and/or proxies, leave the values of their `host:` key blank, but do not remove the `stratum-1:` or `proxies:` keys entirely.
The format of this file is described in
[How to build your inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html).
This file is also where the parameters for the setup are configured.

2. Create a copy of this [`ansible.cfg`](ansible.cfg) file.

3. With both the `hosts.yml` and `ansible.cfg` files in your working directory.
Test that things are working with the command

    ```bash
    ansible-playbook cvmfs.setup.ping
    ```

4. Many of the ansible commands require `sudo` privileges on the remote machines.
    If you would like to disable the sudo password on the remote machines so that the playbooks in this collection can be run without a password, first run the following command, enter the sudo password for the machine which will have its sudo password removed, then press enter.

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

    after running `cvmfs.setup.all`, any of the individual playbooks above case be re-run if needed. However, they all rely on `cvmfs.setup.stratum_0` having been run once in order for the stratum-0 public keys to be copied over properly.

2. To ensure that the environment was set up correctly, you could try propagating a test file through all the machines. i.e. from

    ``` bash
    stratum-0 -> stratum-1 -> proxy -> client
    ```

    By following [these](https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/02_stratum0_client/#216-adding-files-to-the-repository) instructions, we can do this with the following commands

    1. **On the Stratum-0**

        Set an environment variable to the name of your repository.

        ```bash
        MY_REPO_NAME=<repo.org.tld>
        ```

        Then, run these commands[^1]

        ```bash
        cvmfs_server transaction ${MY_REPO_NAME}
        echo '#!/bin/bash' > /cvmfs/${MY_REPO_NAME}/hello.sh
        echo 'echo hello' >> /cvmfs/${MY_REPO_NAME}/hello.sh
        chmod a+x /cvmfs/${MY_REPO_NAME}/hello.sh
        cvmfs_server publish ${MY_REPO_NAME}
        ```

    2. **On a Stratum-1**

       Synchronize the file with[^2]

       ```bash
       cvmfs_server snapshot repo.org.tld>
       ```

        again, replacing `<repo.org.tld>` with your cvmfs-repository name.

    3. **On one of the Client machines**

        Test that you can access the file with the command

        ```bash
        /cvmfs/<repo.org.tld>/hello.sh
        ```

        You can also run the following test commands on the client[^3] [^4].

        ```bash
        cvmfs_config chksetup  # should return OK
        cvmfs_config stat -v <repo.org.tld>
        ```

        The output of the stat command should contain a line with the following

        ```bash
        Connection: http://<STRATUM1_IP>/cvmfs/repo.organization.tld through proxy http://<PROXY_IP>:3128 (online)
        ```

## Testing

This repository uses the [Ansible Molecule](https://ansible.readthedocs.io/projects/molecule/) framework for testing.

### Installation / Setup

<!-- With [`podman`](https://podman.io/) already installed, I was able to perform the tests after running -->

1. Install `pip`, `ansible`, and `podman`. On Debian, they can all be installed with `apt`.

    ``` bash
    sudo apt install -y pip podman ansible
    ```

    Otherwise, follow these instructions to install podman and molecule
    - podman: <https://podman.io/docs/installation>
    - molecule: <https://ansible.readthedocs.io/projects/molecule/installation/>

2. Create a python environment that you can use from root.

    ``` bash
    python3 -m venv cvmfs
    sudo su
    . cvmfs/bin/activate
    pip install ansible molecule molecule-podman
    ```

3. While still acting as root, and in this directory, run

    ``` bash
    ansible-galaxy collection install . --force
    ```

### Test execution

1. Move into the [`extensions`](extensions/) directory

    ``` bash
    cd extensions/
    ```

2. Then, still as root, run

    ``` bash
    molecule test
    ```

    to keep the test environment around after the tests are complete, you can instead run

    ```bash
    molecule destroy && molecule test --destroy=never
    ```

[^1]: <https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/02_stratum0_client/#216-adding-files-to-the-repository>

[^2]: <https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/03_stratum1_proxies/#316-manually-synchronize-the-stratum-1>

[^3]: <https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/02_stratum0_client/#223-mounting-the-repositories>

[^4]: <https://cvmfs-contrib.github.io/cvmfs-tutorial-2021/03_stratum1_proxies/#333-test-the-new-configuration>
