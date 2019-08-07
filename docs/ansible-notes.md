# Ansible

### Contents

- Ansible
  * [What Ansible is?](#what-ansible-is)
  * [Good to know](#good-to-know)
  * [Ansible Modules](#ansible-modules)
  * [Ansible Tasks](#ansible-tasks)
  * [Ansible playbook](#ansible-playbook)
  * [Ansible inventory](#ansible-inventory)

---

<img src="img/ansible_logo.png" width="300" />

#### What Ansible is?

 Described as a configuration management tool, and is typically mentioned in the same breath as Chef, Puppet, and Salt. When we talk about configuration management, we are typically talking about writing some kind of state description for our servers, and then using a tool to enforce that the servers are, indeed, in that state: the right packages are installed, configuration files contain the expected values and have the expected permissions, the right services are running, and so on.

![productionLine](img/automation_line.jpg)

When people talk about deployment, they are usually referring to the process of taking software that was written in-house, generating binaries or static assets (if necessary), copying the required files to the server(s), and then starting up the services.
Ansible is a great tool for deployment as well as configuration management. Using a single tool for both configuration management and deployment makes life simpler for the folks responsible for operations.

Ansible works as a push-based tool.


-----
#### Good to know

Some stuffs which good be nice to know because they let you have a soft way through Ansible are:

* Connect to a remote machine using SSH
* Interact with the Bash command-line shell (pipes and redirection)
* Install packages
* Use the sudo command
* Check and set file permissions
* Start and stop services
* Set environment variables
* Write scripts (any language)

-----
#### Ansible modules

Modules are a great part of Ansible and they encapsulate the behavior of common operation tasks to be applied over the OS and let you specify the arguments to use at execution time. Some examples are:
 - apt - *handle tasks of the apt ubuntu package manager*
 - systemd - *used to start and enable services to be running at boot time*
 - service - *hanlde actions over services like start, stop, restart*
 - templates - *handle jinja templates to be used on the server once the interpolation gives the right values*
 - copy - *used to copy files to the server or between internal server locations*

-----
#### Ansible tasks

To achieve the Ansible's target which is have the servers on the right state we define __tasks__, some examples of tasks that could be needed for a server with a simple page running are:

1. Install Nginx
2. Generate a Nginx config file
4. Start the Nginx service

And these activities will be translated to a Ansible, composed at least by a name, and a module with some of its options set.

```ansible
- name: Install nginx
  apt: name=nginx
```

This task will install the nginx webservers on the host following this steps:

1. Generate a Python script that installs the Nginx package.
2. Copy the script to our server __webappserver__
3. Execute the script on __webappserver__
4. Wait for the script completition on all hosts:
  - Ansible runs each task in parallel across all hosts.
  - Waits until all hosts have completed a task before moving to the next.
  - Tasks runs in the order you specify them.

-----
#### Ansible playbook

When we place one or more tasks next to the definition of hosts to be affected we have a __playbook__ this playbook can contains tasks, variables, pre_tasks and so on.

```ansible
---
- name: Configure webservers
  hosts: webappserver
  become: yes
  tasks:
    - name: Install Nginx
      apt: name=nginx
```

-----
#### Ansible Inventory

To get some information about our current server let's run the command:

```Bash
> vagrant ssh-config

# and you will get something like

Host webappserver
  HostName 127.0.0.1
  User vagrant
  Port 2203
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/volivares/git/workshops/ws-vagrant-ansible/.vagrant/machines/webappserver/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

Create a file called __hosts__ and put some lines of the previous output this files is defining a host named webappserver which point to our VM:

```text
webappserver ansible_host=127.0.0.1 ansible_port=2203 ansible_user=vagrant ansible_private_key_file=../../.vagrant/machines/webappserver/virtualbox/private_key
```

If you try to run them using indicating the inventory to use:

```bash
> ansible-playbook first_playbook.yml -i hosts
```

Maybe you will get the following error:

```bash
fatal: [webappserver]: FAILED! => {"changed": false,
"module_stderr": "Shared connection to 127.0.0.1 closed.\r\n",
"module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n",
"msg": "The module failed to execute correctly, you probably need to set the interpreter.\n
See stdout/stderr for the exact error", "rc": 127}
```

In some cases the operative system ansible looks for python2.7 by default. So we should specify the right version we are gonna use in the playbook setting a variable __ansible_python_interpreter__:

```ansible
---
- name: Configure webservers
  hosts: webappserver
  become: yes
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

At the end it should look like this:

```ansible
---
- name: Configure webservers
  hosts: webappserver
  become: yes
  vars:
    ansible_python_interpreter: /usr/bin/python3
  tasks:
  - name: install nginx
    apt:
      name: nginx
      update_cache: yes

  - name: copy nginx config file
    copy:
      src: files/nginx.conf
      dest: /etc/nginx/sites-available/default

  - name: enable configuration
    file:
      dest: /etc/nginx/sites-enabled/default
      src: /etc/nginx/sites-available/default
      state: link

  - name: copy index.html
    template:
      src: templates/index.html.j2
      dest: /usr/share/nginx/html/index.html
      mode: 0644

  - name: restart nginx
    service:
      name: nginx
      state: restarted

```
.
.
.


<img src="img/good_job_bear.jpg" width="500" />
