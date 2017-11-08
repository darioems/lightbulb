# Exercise 1.2 - Backing up Configurations

Now that you’ve gotten a sense of how ansible works, we are going to write our first ansible **playbook**. The playbook is where you can take some of those ad-hoc commands you just ran and put them into a repeatable set of plays and tasks.

A playbook can have multiple plays and a play can have one or multiple tasks. The goal of a play is to map a group of hosts. The goal of a task is to implement modules against those hosts.

For our first playbook, we will create a backup of our two routers.

## Table of contents
- [Section 1: Creating a Directory Structure and Files for your Playbook](#section-1-creating-a-directory-structure-and-files-for-your-playbook)
- [Section 2: Defining Your Play](#section-2-defining-your-play)
- [Section 3: Adding Tasks to Your Play](#section-3-adding-tasks-to-your-play)
- [Section 4: Review](#section-4-review)
- [Section 5: Running your playbook](#section-5-running-your-playbook)

## Section 1: Creating a Directory Structure and Files for your Playbook

### Step 1: Navigate to the networking-workshop directory

```bash
cd ~/networking-workshop
```

### Step 2: Understand your inventory. Inventories are crucial to Ansible as they define remote nodes on which you wish to run your playbook(s). Cat out (or vim into) your inventory file to understand the hosts file we’ll be working with.

```bash
cat /home/studentXX/networking-workshop/lab_inventory/*hosts
```
**Fill in XX with your student number**

You’ll notice that we are working with 3 groups. The control group, which is the tower node that we are currently ssh’d into. The routers group, which is a grouping of two routers (R1 and R2). And finally the hosts group, which has another linux node residing in a separate Amazon Virtual Private Cloud or [VPC](https://aws.amazon.com/vpc/) for short.

## Section 2: Defining Your Play

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

## Section 3: Adding Tasks to Your Play

Now that we’ve defined your play, let’s add the necessary tasks to backup our routers.

Make sure all of your playbook statements are aligned in the way shown here.
If you want to see the entire playbook for reference, skip to the end of Section 4 of this exercise.

```bash
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

 - `tasks:` This denotes that one or more tasks are about to be defined
 - `name:` Each task should be given a name which will print to standard output when you run your playbook. Therefore, give your tasks a name that is short, sweet, and to the point

 The following section is using the ios_facts ansible module to gather IOS related facts about the router we are targeting. [Click here](http://docs.ansible.com/ansible/latest/ios_facts_module.html) to learn more about the ios_facts module.  We are taking the ios_facts that the module provides and registering it to a variable called facts. This information is now available to us to use in subsequent tasks if we wish to do so. Next, we are making a debug statement to display the output of what information is actually captured when using the ios_facts module.

 ```bash
    - name: gather ios_facts
      ios_facts:
      register: version

    - debug:
        msg: "{‌{version}}"
```

The next three lines are calling the Ansible module ios_config and passing in the parameter backup: yes to capture the configuration of the routers and generate a backup file. Click here to see all options for the ios_config module.

```bash
    - name: Backup configuration
      ios_config:
        backup: yes
```

## Section 4: Review

Now that you’ve completed writing your playbook, it would be a shame not to keep it.

Use the write/quit method in vim to save your playbook, i.e. hit Esc then `:wq!`

And that should do it. You should now have a fully written playbook called backup.yml. You are ready to automate!

Ansible (well, YAML really) can be a bit particular about formatting especially around indentation/spacing. When you all get back to the office, read up on this YAML Syntax a bit more and it will save you some headaches later. In the meantime, your completed playbook should look like this. Take note of the spacing and alignment.
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

## Section 5: Running your playbook

We are now going to run your brand spankin' new playbook on your two routers. To do this, you are going to use the **ansible-playbook** command.

### Step 1: From your playbook directory ( ~/networking-workshop ), run your playbook.

```bash
ansible-playbook backup.yml
```

 - `--syntax-check` If you run into any issues with your playbook running properly; you know, from that copy/pasting that you didn’t do because we said "don’t do that"; you could use this option to help find those issues like so:
 ```bash
 ansible-playbook backup.yml --syntax-check
```

In standard output, you should see something that looks very similar to the following:
![Figure 2: backup playbook stdout](playbook-output.png)

Feel free to scroll back up and take a look at the facts that the ios_facts module collected.

Also, notice that the play and each task is named so that you can see what is being done and to which router it is being done to.

### Step 2: List the files in the backup directory
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

# Complete
You have completed lab exercise 1.2

[Click Here to return to the Ansible Lightbulb - Networking Workshop](../README.md)
