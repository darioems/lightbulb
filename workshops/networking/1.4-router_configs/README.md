# Exercise 1.4 - Additional router configurations

Previous exercises showed you the basics of Ansible Core. In the exercise, let’s build upon that and introduce additional Ansible concepts that allow you to add flexibility and power to your playbooks.

## Table of Contents
 - [Intro](#intro)
 - [Section 1 - Adding variables to your playbook](#section-1---adding-variables-to-your-playbook)
 - [Section 2: Review](#section-2-review)

## Intro

Ansible exists to make tasks simple and repeatable. We also know that not all systems are exactly alike and often require some slight change to the way an Ansible playbook is run.

- **Variables** are how we deal with differences between your systems, allowing you to account for a change in port, IP address or directory.
- **Loops** enable us to repeat the same task over and over again. For example, lets say you want to install 10 packages. By using an ansible loop, you can do that in a single task.
- **Blocks** allow for logical grouping of tasks and even in play error handling. Most of what you can apply to a single task can be applied at the block level, which also makes it much easier to set data or directives common to the tasks.
- **When** clause: sometimes you will want to skip a particular step on a particular host. This could be something as simple as not installing a certain package if the operating system is a particular version, or it could be something like performing some cleanup steps if a filesystem is getting full.

    This is easy to do in Ansible with the when clause, which contains a raw Jinja2 expression without double curly braces (see Variables).

**jinja-who?** - Not to be confused with 2013’s blockbuster "Ninja II - Shadow of a Tear", jinja2 is used in Ansible to enable dynamic expressions and access to variables.

For a full understanding of variables, loops, blocks, conditionals, and jinja2; check out our Ansible documentation on these subjects:
- [Ansible Variables](http://docs.ansible.com/ansible/playbooks_variables.html)
- [Ansible Loops](http://docs.ansible.com/ansible/playbooks_loops.html)
- [Ansible Handlers](http://docs.ansible.com/ansible/latest/playbooks_blocks.html)
- [Ansible Conditionals](http://docs.ansible.com/ansible/latest/playbooks_conditionals.html#the-when-statement)

## Section 1 - Adding variables to your playbook

To begin, we are going to create a new playbook, and call it router_configs.yml

### Step 1: Navigate to the networking-workshop directory to create a new playbook

```bash
cd ~/networking-workshop
vim router_configs.yml
```

### Step 2: Add the play definition and some variables to your playbook as shown below.

This time instead of using the rtr1 and rtr2 public IP addresses, we need to **dynamically** grab the private_ips of the **host1** node and the **tower**/**control** node.  Wonder where those values are being set?

The fine where the value we need for host1_private_ip and control_private_ip we need to look at the inventory for the **host1** node.
 - The IP address can be determined by private_ip=x.x.x.x located at: `/home/studentXX/networking-workshop/lab_inventory/studentXX.WORKSHOP_NAME.hosts` (where XX is your student number and WORKSHOP_NAME was provided by your instructor).  
 - This inventory file is being selected by the global configuration located at `/etc/ansible/ansible.cfg`

```yml
---
- name: Router Configurations
  hosts: routers
  gather_facts: no
  connection: local
  vars:
    ansible_network_os: ios
    dns_servers:
      - 8.8.8.8
      - 8.8.4.4
    host1_private_ip: "{‌{hostva‌rs['host1']['private_ip']}}"
    control_private_ip: "{‌{hostvars['tower']['private_ip']}}"
```      

### Step 3: Add the first task to capture the ios_facts.

```yml
tasks:
  - name: gather ios_facts
    ios_facts:
```
**Note**:  There is no requirement in Ansible 2.4 and later to register the ios_facts to be able to use them.  We registered them in a previous exercise simply to show how to debug and print the facts to the terminal window.

### Step 4: Create a block and add the tasks for rtr1 with conditionals. We’ll also add a comment for better documentation.

```
    ##Configuration for R1
    - block:
      - name: Static route from R1 to R2
        net_static_route:
          prefix: "{‌{host1_private_ip}}"
          mask: 255.255.255.255
          next_hop: 10.0.0.2
      - name: configure name servers
        net_system:
          name_servers: "{‌{item}}"
        with_items: "{‌{dns_servers}}"
      when:
        - '"rtr1" in inventory_hostname'
```

 What the Helsinki is happening here!?
 - `vars:` You’ve told Ansible the next thing it sees will be a variable name.
 - `dns_servers` You are defining a list-type variable called dns_servers. What follows is a list of those the name servers.
 - `{‌{ item }}` You are telling Ansible that this will expand into a list item like 8.8.8.8 and 8.8.4.4.
 - `with_items: "{‌{ dns_servers }}` This is your loop which is instructing Ansible to perform this task on every `item` in `dns_servers`
 - `block:` This block will have a number of tasks associated with it.
 - `when:` We’re tying the when clause to the block. We’re telling ansible to run all the tasks within the block only when certain conditions are met.
  - Condition - the hostname must contain 'rtr1'

### Step 5: Configuring R2.
There will be 4 tasks in this block
- net_interface
- ios_config
- net_static_route
- net_system

```yml
##Configuration for R2
- block:
  - name: enable GigabitEthernet2 interface if compliant
    net_interface:
      name: GigabitEthernet2
      description: interface to host1
      state: present
  - name: dhcp configuration for GigabitEthernet2
    ios_config:
      lines:
        - ip address dhcp
      parents: interface GigabitEthernet2
  - name: Static route from R2 to R1
    net_static_route:
      prefix: "{‌{control_private_ip}}"
      mask: 255.255.255.255
      next_hop: 10.0.0.1
  - name: configure name servers
    net_system:
      name_servers: "{‌{item}}"
    with_items: "{‌{dns_servers}}"
  when:
    - '"rtr2" in inventory_hostname'
```

**So…​ what’s going on?**
 - `net_interface:` This module allows us to define the state of the interface (up, admin down, etc.) in an agnostic way. In this case, we are making sure that GigabitEthernet2 is up and has the correct description.
 - `ios_config:` We’ve used this module in previous playbooks. We could technically combine the two tasks (ip addr + static route). However, it’s sometimes preferred to break out the tasks according to what is being accomplished.
 - `net_system:` This module, similar to the net_interface allows us to manage the system attributes on network devices in an agnostic way. We’re utilizing this module along with loops to feed in the name_servers we want the router to have.
 - `net_static_route:` This module is utilized for managing static IP routes on network devices. It provides declarative management of static IP routes on network devices.

## Section 2: Review

Your playbook is done! But don’t run it just yet, we’ll do that in our next exercise. For now, let’s take a second look to make sure everything looks the way you intended. If not, now is the time for us to fix it up.

To view and run the completed playbook move on to [Exercise 1.5!](../1.5-run_routing_configs)

[Click Here to return to the Ansible Lightbulb - Networking Workshop](../README.md)
