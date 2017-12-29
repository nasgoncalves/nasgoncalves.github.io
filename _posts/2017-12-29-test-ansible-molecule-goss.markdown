---
layout: post
title: "Test Ansible Roles using molecule and goss"
date: 2017-12-29 00:00:01
---

<br>

[Github Project](https://github.com/nasgoncalves/r-nasg.dummy)

## Preparation

Create new python environment if you are using python 3 (molecule at this point only supports python 2.7).
```bash
$ conda create -n ansible python=2.7
$ # for some reson molecule doesn't install
$ # functools_lru_cache automagically
$ source activate ansible
$ pip install ansible molecule docker-py backports.functools_lru_cache
$ # install goss
$ curl -fsSL https://goss.rocks/install | sh
```
<br>

## Setup

Create a some role with ansible-galaxy,...

```bash
(ansible) $ ansible-galaxy init r-nasg.dummy
```
Create your default playbook (default.yml) with the following content:

```yaml
---
- hosts: localhost
  roles:
    - r-nasg.dummy

```

In the tasks file (tasks/main.yml) add something simple just to test the concept...

```yaml
---
- name: Add flag
  blockinfile:
    path: /root/ansible-flag
    create: yes
    block: |
      flag
```

<br>

## Create tests

Create the tests using molecule using the following command:

```bash
(ansible) $ # molecule init scenario -r <role-name> -s <scenario-name> -d <driver (docker | vagrant | etc.)> --verifier-name <(goss | testinfra | etc.)>
(ansible) $ molecule init scenario -r r-nasg.dummy -s default -d docker --verifier-name goss
```

molecule will create a series of folders and files, open the tests file (playbook_dir/molecule/default/tests/test_default.yml) and add the following:

```yaml
---
file:
  /root/ansible-flag:
    exists: true
    owner: root
    group: root
    contains:
      - flag
```

<br>

## Run tests

Finally run:
```bash
(ansible) $ molecule test
```
Output:

```bash
...
"msg": [
    "File: /root/ansible-flag: exists: matches expectation: [true]",
    "File: /root/ansible-flag: owner: matches expectation: [\"root\"]",
    "File: /root/ansible-flag: group: matches expectation: [\"root\"]",
    "File: /root/ansible-flag: contains: matches expectation: [flag]",
    "",
    "",
    "Total Duration: 0.001s",
    "Count: 4, Failed: 0, Skipped: 0"
]
...
```

### Notes

You can always skip ansible-lint check in any taks uou pick add the tag `skip_ansible_lint` in the `tags` statement.

```yaml
tags:
  - skip_ansible_lint
```

<br>

### Refs

* [molecule](https://github.com/metacloud/molecule)
* [goss](https://github.com/aelsabbahy/goss)
* [testinfra](https://github.com/philpep/testinfra)
