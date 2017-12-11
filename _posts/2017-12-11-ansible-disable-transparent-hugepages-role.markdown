---
layout: post
title: "Ansible Disable Transparent Hugepages Role"
date: 2017-12-11 00:00:01
---

```
- name: Disable thp
  shell: echo 'never' > "{{ item }}"
  with_items:
    - /sys/kernel/mm/transparent_hugepage/enabled
    - /sys/kernel/mm/transparent_hugepage/defrag
    - /sys/kernel/mm/redhat_transparent_hugepage/enabled
    - /sys/kernel/mm/redhat_transparent_hugepage/defrag
  when: "{{ disable_transparent_hugepages }}"
  ignore_errors: true
  become: yes

- name: Disable thp after every init script had initialized
  lineinfile:
    dest: /etc/rc.d/rc.local
    insertafter: EOF
    state: present
    create: yes
    line: |
      if test -f {{ item }}; then
      echo never > {{ item }}
      fi
    mode: 0744
  with_items:
    - /sys/kernel/mm/transparent_hugepage/enabled
    - /sys/kernel/mm/transparent_hugepage/defrag
    - /sys/kernel/mm/redhat_transparent_hugepage/enabled
    - /sys/kernel/mm/redhat_transparent_hugepage/defrag
  become: yes

```
