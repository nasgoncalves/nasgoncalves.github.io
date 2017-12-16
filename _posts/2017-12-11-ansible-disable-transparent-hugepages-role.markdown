---
layout: post
title: "Ansible Disable Transparent Huge Pages Role"
date: 2017-12-11 00:00:01
---

```yaml
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


* [Checking if Hugepages are enabled in Linux
][r1]
* [How to use, monitor, and disable transparent hugepages in Red Hat Enterprise Linux 6?][r2]
* [Transparent Hugepage Support][r3]

[r1]: https://techoverflow.net/2013/08/01/checking-if-hugepages-are-enabled-in-linux/
[r2]: https://access.redhat.com/solutions/46111
[r3]: https://www.kernel.org/doc/Documentation/vm/transhuge.txt
