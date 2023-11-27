# 00424 Basis Pilot

This project provides a playbook and associated configuration to automate the rollout of a minimal
UCS@school domain according to the "Bildungscloud" Scenario 
(see https://docs.software-univention.de/ucsschool-scenarios/5.0/de/scenarios.html#szenario-1-bildungscloud)

## CAUTION -- work in progress!

There are some framework bugs in the referenced Univention roles and -modules. As long as these aren't fixed,
this repository holds some patched files which should be copied into the checked-out galaxy area. The following
has to be executed after the galaxy checkout is done AND the 'Obtain Ansible parts' (below) is done:
```
cd 00424-basis-pilot/ucsschool-basis-pilot-ansible/patches/
cp main.yml /root/.ansible/collections/ansible_collections/univention/ucs_roles/roles/get_installed_apps/tasks/
cp convert_app_list.j2 /root/.ansible/collections/ansible_collections/univention/ucs_roles/roles/get_installed_apps/templates/
cp univention_app.py /root/.ansible/collections/ansible_collections/univention/ucs_modules/plugins/modules/
cd
```
When these bugs have been resolved (and rolled out to the Ansible Galaxy) this part can be omitted, and the `patches` subdirectory
of this repository can be removed. Note that the Jenkins job does these copies too, so it has to be corrected too.

## Prerequisites

This describes what a customer has to provide in their environment.

A *Debian VM* to execute the Ansible playbook.

* Current Debian 12 (Bookworm) release.
* Installed packages: ansible
* Installed from Ansible Galaxy: Univention roles and modules:
  ```
  deb:~# ansible-galaxy collection install univention.ucs_roles
  deb:~# ansible-galaxy collection install univention.ucs_modules
  ```

A *UCS Domain* consisting of one Primary, one Backup, one Replica.

* UCS installed, latest patchlevel.
* Server role chosen for each of the servers.
* UCS configured, all machines rebooted (Caution: not joined).
* All servers must be able to reach the internet. As the machines are required to be at the
  latest patch level it is probably already fulfilled.
* The UCS machines have to be reachable from the Debian machine by `ssh` using public keys:
  ```
  deb:~# ssh-keygen
     ... (accept all prompts by pressing Enter)

  deb:~# ssh-copy-id root@primary
     ... (enter root password of primary)
  (repeat this step for backup and replica)
  ```

For reachability from the public internet:

* The *Backup* role must carry the `letsencrypt` app, and this has to be configured in
  `backup.yaml` (the file is sufficiently commented).
* There must be a reverse proxy or port forwarding in place that reaches the backup role.

(FIXME: what else should be written to the customer, just to really get a reliable start environment?)

## Usage

### Obtain Ansible parts

Not yet clear how we get the sources of *this* repository onto the Debian server. For the automatic
enrollment using Jenkinks there is a command that clones the internal repo
`git.knut.univention.de/univention/prof-services/basis-pilot-projekt/00424-basis-pilot.git` directly
into root's home, creating the `00424-basis-pilot` directory. Perhaps the easiest way would be:

* Download the code accessing the web frontend at git.knut.univention.de, using the `download source code`
  function.
* Transfer this file to the debian host (or ask the customer to do that)
* Unpack it into the same directory.

### Install requirements

`ansible-galaxy install -r ansible-galaxy install -r 00424-basis-pilot/ucsschool-basis-pilot-ansible/ansible/requirements.yml`

### Edit required parameters

Edit the `.yaml` files in `inventories/sample/inventory` to set the required parameters.
Note that all yaml files are commented.

1. `all.yaml`

* set domain name and LDAP base DN
* name additional apps to be installed on all roles

2. `primary.yaml`

* set desired school parameters
* change the license file name to the actual one

3. the other files

* more/less apps as needed
* configuration parameters for apps
  * [backup] -> letsencrypt

### Obtain License

License file has to be obtained from Univention sales on behalf of the customer, and copied into
`/root/00424-basis-pilot/ucsschool-basis-pilot-ansible/files/`
on the Debian host. The playbook will import it into the UCS domain. Note that the name of the
license file (without path) has to be written into `primary.yaml` (Caution: Please do not use
spaces or other unusual characters in the file name, and exactly honor upper/lowercase filename.)

### Prepare inventory and configuration

These two steps are also carried out by Jenkins when this is embedded into a Jenkins job. For a
customer environment, the steps have to be done manually. (Maybe this can be delegated to the
customer? Better yet: even if the customer shall prepare this: you should always verify that
everything is in place.

* *Prepare inventory file*: create a file `/root/ansible-root/inventory/basis-pilot-kvm.ini` with
  the contents:
  ```
  [primary]
  the.ip.of.primary

  [backups]
  the.ip.of.backup

  [replicas]
  the.ip.of.replica
  ```
* *Prepare Ansible configuration*: create a file `/root/ansible-root/inventory/basis-pilot.ini` with
  the contents:
  ```
  domainname='the.dns.domainname.of.ucs.domain'
  windows_domain='NETDOMAIN'
  ldap_base='dc='the',dc=dns,dc=domainname,dc=of,dc=ucs,dc=domain'
  root_password='Administrator-pw'
  interface_name='eth0'
  interface_mode='dhcp'
  primary_hostname='how they named their primary'
  primary_ip='the.ip.of.primary'
  ```
  and a configuration file `/root/.ansible.cfg` with the contents:
  ```
  [defaults]
  host_key_checking = False
  ```
  (NOTE TO QA: maybe these variables double the function of the `all.yaml` and the other
  `.yaml` files in this directory. I did not check which of these variables are really
  in use, or can be omitted here.)

### Configure Server

This is the whole configuration process. The customer should have been advised to take snapshots
of the three UCS machines before running this. (Theorecically, a playbook should be idempotent,
but if something goes wrong it would be the easiest way to roll back and rerun the whole playbook.)

```
ansible-playbook \
--private-key ~/ansible-root/tech.pem \
-u root \
-i ~/ansible-root/inventory/basis-pilot.ini \
-i ~/ansible-root/inventory/basis-pilot-kvm.ini \
00424-basis-pilot/ucsschool-basis-pilot-ansible/ansible/playbook.yaml
```
