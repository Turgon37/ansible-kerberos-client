Ansible Role Kerberos Client
=========

[![Build Status](https://travis-ci.com/Turgon37/ansible-kerberos-client.svg?branch=master)](https://travis-ci.com/Turgon37/ansible-kerberos-client)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-Turgon37.kerberos_client-blue.svg)](https://galaxy.ansible.com/Turgon37/kerberos_client/)

## Description

:grey_exclamation: Before using this role, please know that all my Ansible roles are fully written and accustomed to my IT infrastructure. So, even if they are as generic as possible they will not necessarily fill your needs, I advice you to carrefully analyse what they do and evaluate their capability to be installed securely on your servers.

This roles configure the MIT kerberos client host keytab and krb5 settings.

## Requirements

Require Ansible >= 2.4

### Dependencies

## OS Family

This role is available for Debian and CentOS

## Features

At this day the role can be used to :

  * produce, fetch and install host keytab
  * configure krb5.conf
  * [local facts](#facts)

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below. To see default values please refer to this file.

| Name                                                  | Types/Values       | Description                                                                          |
| ------------------------------------------------------| -------------------|------------------------------------------------------------------------------------- |
| `kerberos_client__facts`                              | Boolean            | Install the local fact script                                                        |
| `kerberos_client__include_dir`                        | Path               | Directory which contains config parts to include                                     |
| `kerberos_client__include_dirs`                       | List of Path       | List of extra directory (not managed by this role) to include                        |
| `kerberos_client__keytab_principal`                   | String             | The pattern use to produce the host's principal                                      |
| `kerberos_client__keytab_deploy_method`               | Enum in none,remote| Choose the host keytab deploiement method (see below)                                |
| `kerberos_client__keytab_deploy_remote_genkey_command`| String             | The shell command to run to produce the keytab on KDC                                |
| `kerberos_client__keytab_deploy_tmp_file`             | Path               | Path to the temporary keytab ok KDC                                                  |
| `kerberos_client__keytab_deploy_remote_genkey_nolog`  | Boolean            | Set to yes to protect any password in `remote_genkey_command` to be logged by ansible|
| `kerberos_client__keytab_deploy_remote_host`          | String             | The KDC hostname                                                                     |
| `kerberos_client__realms`                             | Dict               | Dict of per realm configurations, fill the ```[realms]``` section                    |
| `kerberos_client__domains`                            | Dict               | Dict of domain-realm mapping, fill the ```[domain_realm]``` section                  |
| `kerberos_client__dbmodules`                          | Dict               | Dict of dbmodules, fill the ```[dbmodules]``` section                                |
| `kerberos_client__plugins`                            | Dict               | Dict of plugins, fill the ```[plugins]``` section                                    |


### Keytab deploiement

This role can deploy generate and deploy the keytab to the current host.

By default the deploiment method is none, so the keytab will not be handled at all by the role.

If you set the method to 'remote', ansible will ssh to the KDC, run the command given by `kerberos_client__keytab_deploy_remote_genkey_command` and fetch the produced keytab in initial host.
You must ensure that the produced keytab will be located at `kerberos_client__keytab_deploy_tmp_file` on KDC


## Facts

By default the local fact are installed and expose the following variables :


```
ansible_local.kerberos_client:
  principals:
    - host/test.example.com@EXAMPLE.COM
```

## Example

### Playbook

Use it in a playbook as follows:

```yaml
- hosts: all
  roles:
    - turgon37.kerberos_client
```

### Inventory

To use this role create or update your playbook according the following example :

```
kerberos_client__keytab_deploy_remote_host: '{{ kdc_host }}'
kerberos_client__keytab_principal: 'host/{{ ansible_fqdn }}@EXAMPLE.COM'
kerberos_client__keytab_deploy_method: remote
kerberos_client__keytab_deploy_remote_genkey_command: >
  echo '{{ enroll_pass }}' | kinit '{{ enroll_user }}';
  echo 'ktadd -k {{ kerberos_client__keytab_deploy_tmp_file }} host/{{ ansible_fqdn }}@EXAMPLE.COM | kadmin;
  ret=$?;
  kdestroy;
  exit $ret
kerberos_client__keytab_deploy_remote_genkey_nolog: true

kerberos_client__defaults:
  default_realm: EXAMPLE.COM
  dns_lookup_realm: 'false'
  dns_lookup_kdc: 'true'
  rdns: 'false'
  ticket_lifetime: 24h
  forwardable: 'true'
  udp_preference_limit: 0

kerberos_client__realms:
  EXAMPLE.COM:
    kdc: '{{ kdc_host }}'
    master_kdc: '{{ kdc_host }}'
    admin_server: '{{ kdc_host }}'
    kpasswd_server: '{{ kdc_host }}'
    default_domain: example.com

kerberos_client__domains:
  example.com: EXAMPLE.COM
  .example.com: EXAMPLE.COM
  '{{ ansible_fqdn }}': EXAMPLE.COM
```

With custom dbmodules or plugins :

```
kerberos_client__dbmodules:
  EXAMPLE.COM:
    db_library: ipadb.so

kerberos_client__plugins:
  certauth = {
    enable_only = ipakdb
    module = ipakdb:kdb/ipadb.so
  }
```

### Role inclusion

This role is included in FreeIPA client configuration role https://github.com/Turgon37/ansible-freeipa-client/blob/master/tasks/configure.yml