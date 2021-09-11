
# WordPress Provision through Ansible role on  amazon linux and centos
[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Description:
Here is a simple documentation for wordpress site provision through ansible-role. Here I have written  this role for centos and amazon linux.  Furthermore, the ansible role is  with complete packages of the lamp stack.

## Features:

- All packages apache, mysql,php, wordpress are installed through ansibile role.
- Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users

## Components and Resources:
- Apache (Used webserver)
- PHP and dependencies
- MySQL
- WordPress 5.7
- Ansible2 (Ansible-Master)
- Centos
- Amazon linux

Here is the role I have created for this project, the role name is "lamp"
```sh
lamp
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── inventory.txt
├── key.pem
├── lamp-wordpress.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   ├── amazon.yml
│   ├── centos.yml
│   ├── main.yml
│   └── wordpress.yml
├── templates
│   ├── httpd.conf.temp
│   ├── virtualhost.conf.temp
│   └── wp-config.php.temp
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```



The tasks directory contains  4 task o amazon.yml , centos.yml, main,yml and wordpress.yml.
Here wordpress.yml contain default installtion steps of wordpress.


```sh
---
- name: Installation of wordpress
  become: true
  hosts: all
  roles:
    - lamp
  tasks:
    - name: "Wordpress - Downloading Archive File"
      get_url:
        url: https://wordpress.org/wordpress-5.8.tar.gz
        dest: /tmp/wordpress.tar.gz

    - name: "Wordpress - Extracting Wordpress"
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /tmp/
        remote_src: true

    - name: "Wordpress - Copying Contents To Docroot"
      copy:
        src: /tmp/wordpress/
        dest: /var/www/html/{{ domain_name }}/
        remote_src: true
        owner: "{{ httpd_user }}"
        group: "{{ httpd_group }}"


    - name: "Wordpress - Creating wp-config.php"
      template:
        src: wp-config.template
        dest: "/var/www/html/{{ domain_name }}/wp-config.php"
        owner: "{{ httpd_user }}"

```
In amazon.yml and centos.yml  contains the code for the  installtion of lamp stack.

Here I have pasted the code in amazon.yml file.

```sh
---

- name: "Task-01: Installing Apache webserver"
  yum:
    name: httpd
    state: present
  notify:
    - restart-httpd

- name: "Task-02: custom httpd config file for httpd"
  template:
    src: httpd.conf.temp
    dest: /etc/httpd/conf/httpd.conf
  notify:
    - restart-httpd

- name: " Task-03:Creating Virtualhost from template"
  template:
    src: virtualhost.conf.tmp
    dest: "/etc/httpd/conf.d/{{ domain_name }}.conf"
  notify:
    - restart-httpd

- name: "Task-03: DocumentRoot Creation"
  file:
    path: "/var/www/{{ domain_name }}"
    state: directory
    owner: "{{ httpd_user }}"
    group: "{{ httpd_group }}"
  notify:
    - restart-httpd

- name: "Task-01: Installing latest php"
  command: amazon-linux-extras install php7.4 -y
  notify:
    - restart-httpd

- name: "Task-09:Installing mariadb"
  yum:
    name:
      - mariadb-server
    state: present

- name: "Task-09:Installing mariadb"
  yum:
    name:
      - mariadb-server
      - MySQL-python
    state: present

- name: "Task-10:Restarting/enabling mysql"
  service:
    name: mariadb
    state: restarted
    enabled: true

- name: "Task-11:mariadb - Setting RootPassword"
  ignore_errors: true
  mysql_user:
    login_user: "root"
    login_password: ""
    user: "root"
    password: "{{ root }}"
    host_all: true

- name: "Task-12:mariadb - Removing Anonymous Users"
  mysql_user:
    login_user: "root"
    login_password: "{{ root }}"
    user: ""
    state: absent
    host_all: true
```

In vars  we initialize the variables required for the role. The variables that we used in this project is pasted below.

```sh
---
# vars file for lamp
httpd_user: apache
httpd_group: apache
httpd_port: 80
mariadb_root: "mysqlroot@123"
domain_name: "site1.sruthy123.xyz"
username: "wordpress"
password: "wordpress"
database: "wordpress"
```

In handler section I have added code for httpd restart.

```sh
---
# handlers file for lamp
- name: "restart-httpd"
  service:
    name: httpd
    state: restarted
    enabled: true
```

In inventory file I have added the login details of  amazon-linux  and  centos server.


We can check the syntax of playbook "lamp-wordpress.yml" using  following command

```sh
ansible-playbook -i inventory.txt lamp-wordpress.yml --syntax-check
```

Then run the following command to install lamp and wordpress on amazon linux and centos

```sh
ansible-playbook -i inventory.txt lamp-wordpress.yml
```

Conclusion:

Ansible roles allow you to develop reusable automation components by grouping and encapsulating related automation artifacts, like configuration files, templates, tasks, and handlers. Because roles isolate these components, it's easier to reuse them and share them with other people.

