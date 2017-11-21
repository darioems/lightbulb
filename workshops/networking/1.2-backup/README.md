# Exercise 1.2 - Backing up Configurations

We are going to write our first Ansible **playbook**. The playbook is where you can take some of those ad-hoc commands from exercise 1.1 and make them a repeatable set of plays and tasks.

A playbook can have multiple plays and a play can have one or multiple tasks. The goal of a play is to map a group of hosts. The goal of a task is to implement modules against those hosts.

For our first playbook, we will create a backup of the two routers.

## Table of contents
- [Playbook 1 - Backup.yml](#playbook-1---backupyml)
  - [Section 1: Defining Your Play](#section-1-defining-your-play)
  - [Section 2: Adding Tasks to Your Play](#section-2-adding-tasks-to-your-play)
  - [Section 3: Review](#section-3-review)
  - [Section 4: Running the playbook](#section-4-running-the-playbook)
- [Playbook 2 - host-routes.yml](#playbook-2---host-routesyml)
  - [Section 1: Defining the 2nd play](#section-1-defining-the-2nd-play)
- [Answer Key](#answer-key)

## Playbook 1 - Backup.yml
A playbook for backing up Cisco IOS configurations.

**What you will learn:**
 - ios_facts module
 - register keyword
 - debug module

 ---

#### Step 1: Navigate to the networking_workshop directory

```bash
cd ~/networking_workshop
```

#### Step 2: Understand your inventory.

Inventories are crucial to Ansible as they define remote nodes on which you wish to run your playbook(s). Cat out (or vim into) your inventory file to understand the hosts file we’ll be working with.

```bash
cat ~/networking_workshops/lab_inventory/hosts
```

You’ll notice that we are working with 3 groups. The control group, which is the tower node that we are currently ssh’d into. The routers group, which is a grouping of two routers (rtr1 and rtr2). And finally the hosts group, which has another linux node residing in a separate Amazon Virtual Private Cloud or [VPC](https://aws.amazon.com/vpc/) for short.

### Section 1: Defining Your Play

Let’s create our first playbook and name it backup.yml.

```bash
vim backup.yml
```

Now that we are editing [backup.yml](backup.yml), let’s begin by defining the play and then understanding what each line accomplishes

```yml
---
- name: backup router configurations
  hosts: routers
  connection: local
  gather_facts: no
```  

 - `---` Let’s us know that the following is a yaml file.
 - `hosts:` routers Defines the host group in your inventory on which this play will run against
 - `name:` backup router configurations This describes our play
 - `gather_facts: no` Tells Ansible to not run something called the setup module. The setup module is useful when targeting computing nodes (Linux, Windows), but not really used when targeting networking devices. We would use the necessary platform_facts module depending on type of nodes we’re targeting.
 - `connection: local` tells Ansible to execute this python module locally (target node is not capable of running Python)

### Section 2: Adding Tasks to Your Play

Now that we’ve defined your play, let’s add the necessary tasks to backup our routers.

Make sure all of your playbook statements are aligned in the way shown here.
If you want to see the entire playbook for reference, skip to the end of Section 4 of this exercise.

{% raw %}
```bash
  tasks:
    - name: gather ios_facts
      ios_facts:
      register: version

    - debug:
        msg: "{{version}}"

    - name: Backup configuration
      ios_config:
        backup: yes
```
{% endraw %}      

 - `tasks:` This denotes that one or more tasks are about to be defined
 - `name:` Each task should be given a name which will print to standard output when you run your playbook. Therefore, give your tasks a name that is short, sweet, and to the point

 The following section is using the ios_facts ansible module to gather IOS related facts. [Click here](http://docs.ansible.com/ansible/latest/ios_facts_module.html) to learn more about the ios_facts module.  The facts (i.e. information) is now available to us to use in subsequent tasks if we wish to do so.  Next, we are making a debug statement to display the output of what information is actually captured when using the ios_facts module so we know what is available to use.

{% raw %}
 ```bash
    - name: gather ios_facts
      ios_facts:
      register: version

    - debug:
        msg: "{{version}}"
```
{% endraw %}

The next three lines are calling the Ansible module ios_config and passing in the parameter backup: yes to capture the configuration of the routers and generate a backup file. Click here to see all options for the ios_config module.

```bash
    - name: Backup configuration
      ios_config:
        backup: yes
```

### Section 3: Review

Now that you’ve completed writing your playbook, it would be a shame not to keep it.

Use the write/quit method in vim to save your playbook, i.e. hit Esc then `:wq!`

And that should do it. You should now have a fully written playbook called backup.yml. You are ready to automate!

Yaml can be a bit particular about formatting especially around indentation/spacing.  Take note of the spacing and alignment:
{% raw %}
```
---
- name: backup router configurations
  hosts: routers
  connection: local
  gather_facts: no

  tasks:
    - name: gather ios_facts
      ios_facts:
      register: version

    - debug:
        msg: "{‌{version}}"

    - name: Backup configuration
      ios_config:
        backup: yes
```       
{% endraw %}

### Section 4: Running the playbook

We are now going to run the new playbook on both routers. To do this, you are going to use the **ansible-playbook** command.

#### Step 1: From your playbook directory ( ~/networking_workshops ), run your playbook.

```bash
ansible-playbook backup.yml
```
In standard output, you should see something that looks very similar to the following:
![Figure 2: backup playbook stdout](playbook-output.png)

Want to test a playbook to see if your syntax is correct before executing it on remote systems?

 Try using `--syntax-check` If you run into any issues with your playbook running properly help find those issues like so:
 ```bash
ansible-playbook backup.yml --syntax-check
```

#### Step 2: List the files in the backup directory
You can view the backup files that were created by listing the backup directory.

```bash
ls backup
```

You can also view the contents of the backed up configuration files:
```bash
less backup/rtr1*
```
or

```bash
less backup/rtr2*
```
## Playbook 2 - host-routes.yml
A playbook for configuring static Routes.

What you will learn:
 - lineinfile module
 - handlers
 ---

### Section 1: Defining the 2nd play

Let’s create our 2nd playbook and name it `host-routes.yml`

```bash
vim host-routes.yml
```

For our 2nd playbook we need to add a routes from VPC-1 (172.16.0.0/16) to VPC-2 (172.17.0.0/16) and vice versa.  For this exercise we will also illustrate handlers.

We need two routes:
 - From the `ansible` control node to `rtr1` for the `172.17.0.0/16` subnet
 - From the `host1` node to `rtr2` for the 172.16.0.0/16 subnet

For this playbook we will be running only on the `ansible` and `host1` nodes.  Start off the playbook like the backup.yml but make sure to remove `connection:local`.  Lets call this playbook `host-routes.yml` since we are adding static routes on the two hosts.
```yml
---
- name: add route on ansible
  hosts: ansible
  gather_facts: no
  become: yes
```

The `ansible` host is running Red Hat Enterprise Linux Server.  To add a static route we just need to add a line using the [Ansible lineinfile module](http://docs.ansible.com/ansible/latest/lineinfile_module.html) with the subnet and destination under `/etc/sysconfig/network-scripts/route-eth0`.  The ```create: yes``` will create the file if its not already created.  We will also use `notify: "restart network"` to run a handler if this file changes.

```yml
  tasks:
    - name: add route to 172.17.0.0/16 subnet on ansible node
      lineinfile:
        path: /etc/sysconfig/network-scripts/route-eth0
        line: "172.17.0.0/16 via {{hostvars['rtr1']['private_ip']}}"
        create: yes
      notify: "restart network"
```
Next we need to create a handler to restart networking if routes are changed.  The name matters and must match what we are notifying in the task displayed above.  In this case it has to be `restart network` but is user defined and as long as it matches the handler will be run.

```yml
  handlers:
    - name: restart network
      systemd:
        state: restarted
        name: network
```

Now we need to repeat for `host1`:

```yml
- name: add route on host1
  hosts: host1
  gather_facts: no
  become: yes

  tasks:
    - name: add route to 172.16.0.0/16 subnet on host1 node
      lineinfile:
        path: /etc/sysconfig/network-scripts/route-eth0
        line: "172.16.0.0/16 via {{hostvars['rtr2']['private_ip']}}"
        create: yes
      notify: "restart network"

  handlers:
    - name: restart network
      systemd:
        state: restarted
        name: network
```
Now run the playbook:
```bash
ansible-playbook host-routes.yml
```

# Complete
You have completed lab exercise 1.2

# Answer Key
- For backup.yml [click here](https://github.com/network-automation/lightbulb/blob/master/workshops/networking/1.2-backup/backup.yml).
- For host-routes.yml [click here](https://github.com/network-automation/lightbulb/blob/master/workshops/networking/1.2-backup/host-routes.yml)

 ---
[Click Here to return to the Ansible Lightbulb - Networking Workshop](../README.md)
