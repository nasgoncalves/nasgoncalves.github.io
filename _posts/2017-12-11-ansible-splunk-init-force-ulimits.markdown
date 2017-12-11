---
layout: post
title: "Ansible Splunk Init Force Ulimits"
date: 2017-12-11 00:00:01
---

```yaml
- name: Read ulimit value
  shell: "cat /etc/security/limits.conf | grep 'hard nofile' | awk '{print $4}'"
  register: configured_ulimit

- set_fact:
    config_file: "/etc/init.d/splunk"

- name: Add script init
  blockinfile: 
    dest: "{{ config_file }}"
    marker: "# {mark}"
    insertafter: "^splunk_start()*"
    content: |
      {{'  '}}echo Assert ulimits...
      {{'  '}}ulimit -Hn {{ configured_ulimit.stdout }}
      {{'  '}}ulimit -Sn {{ configured_ulimit.stdout }}

      {{'  '}}echo Assert THP at boot time
      {{'  '}}if test -f /sys/kernel/mm/redhat_transparent_hugepage/enabled; then
      {{'  '}}  echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
      {{'  '}}fi
      {{'  '}}if test -f /sys/kernel/mm/redhat_transparent_hugepage/defrag; then
      {{'  '}}  echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
      {{'  '}}fi
  notify: Restart Splunk
```
