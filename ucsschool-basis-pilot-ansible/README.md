# 00424 Basis Pilot

This project provides a playbook and associated configuration to automate the rollout of a minimal
UCS@school domain according to the "Bildungscloud" Scenario 
(see https://docs.software-univention.de/ucsschool-scenarios/5.0/de/scenarios.html#szenario-1-bildungscloud)

## Prerequisites

A *Debian VM* to execute the Ansible playbook.

* Current Debian 12 (Bookworm) release.
* Installed packages: ansible
* Installed from Ansible Galaxy: Univention roles and modules. (FIXME: command line to do that)

A *UCS Domain* consisting of one Primary, one Backup, one Replica.

* UCS installed, latest patchlevel.
* Server role chosen for each of the servers
* UCS configured (Caution: not joined)

(FIXME: what else should be written to the customer, just to really get a reliable start environment?)

## Usage

### Install requirements

`ansible-galaxy install -r requirements.yml`

### Edit required parameters

Edit the `.yaml` files in `inventories/sample/inventory` to set the required parameters.
Note that all yaml files are commented.

1. `all.yaml`

* set domain name and LDAP base DN
* name additional apps to be installed on all roles

2. `primary.yaml`

* set desired school parameters (FIXME: comment all possible parameters)

3. the other files

* more/less apps as needed
* configuration parameters for apps
  * [backup] -> letsencrypt

### Obtain License

License file has to be obtained from Univention sales on behalf of the customer, and copied into (FIXME: file name)
on the Debian host. The playbook will import it into the UCS domain.

### Configure Server

```
ansible-playbook \
--private-key ~/ansible-root/tech.pem \
-u root \
-i ~/ansible-root/inventory/basis-pilot.ini \
-i ~/ansible-root/inventory/basis-pilot-kvm.ini \
00424-basis-pilot/ucsschool-basis-pilot-ansible/ansible/playbook.yaml
```
