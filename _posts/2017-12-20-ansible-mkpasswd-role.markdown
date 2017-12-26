---
layout: post
title: "Ansible mkpasswd role"
date: 2017-12-20 00:00:01
---

### default.yml
```yaml
- hosts: localhost
  pre_tasks:

    - set_fact:
        target_host: "localhost"

  roles:
    - { role: misc.mkpasswd,
        password_samples: 20,
        password_length: 40,
        target_host: "localhost",
        env: 'dev' }

    - misc.mkpasswd

  post_tasks:

    - debug: msg="{{ misc_mkpasswd_rand_pw_string }}"
```

### tasks/main.yml
```yaml
- name: Reset variables (password_samples)
  set_fact:
    misc_mkpasswd_password_samples: "{{ (password_samples) | default(20) }}"
  when: password_samples is defined and password_samples > 0

- name: Reset variables (password_length)
  set_fact:
    misc_mkpasswd_password_length: "{{ (password_length) | default(20) }}"
  when: password_length is defined and password_length > 0

- name: Reset variables (delegate_to)
  set_fact:
    misc_mkpasswd_delegate_to: "{{ target_host | default('localhost') }}"
  when: target_host is defined and (target_host | length) > 0

- name: Reset variables (env)
  set_fact:
    env: 'dev'
  when: not(env is defined)

- name: Reset variables
  set_fact:
    misc_mkpasswd_rand_pw_string: ''

- name: Generate new random password
  shell: </dev/urandom tr -dc '1234567890qwertyuiopQWERTYUIOPasdfghjklASDFGHJKLzxcvbnmZXCVBNM*_!#&$%' | head -c{{ misc_mkpasswd_password_length }}; echo
  register: misc_mkpasswd_rand_pw_string
  delegate_to: "{{ misc_mkpasswd_delegate_to }}"
  with_sequence: start=0 end={{ misc_mkpasswd_password_samples | int }}
  no_log: "{{ env == 'dev' }}"

- name: Pick password
  set_fact:
    misc_mkpasswd_rand_pw_string: "{{ misc_mkpasswd_rand_pw_string.results[misc_mkpasswd_password_samples | int | random].stdout }}"

- name: The new password is
  debug:
    msg: "{{ misc_mkpasswd_rand_pw_string }}"
  when: env == 'dev'
```
