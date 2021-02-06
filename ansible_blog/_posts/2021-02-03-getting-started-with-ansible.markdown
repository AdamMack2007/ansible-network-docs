---
layout: post
title: "Ansible Networking - Getting Started"
date: 2021-02-03 15:59:11 -0700
toc: true
image: "/assets/images/ansible.jpg"
hero_darken: true
categories: jekyll update
summary: |-
  Covers the basics of getting started with Ansible such as creating Inventories, Playbooks, Variables, etc
---

<img src="/assets/images/ansible.jpg" />

# Getting Started With Ansible

Getting started with Ansible is exciting but can also be overwhelming for new users. In this post I'm going to go over the basics of setting up your work environment, creating ansible configuration and inventory files, create a playbook and render out a Jinja2 template. The intended audience is for those that are first getting started with Ansible and want to work with networking devices.

- [Getting Started With Ansible](#getting-started-with-ansible)
  - [Setting Up Your Environment](#setting-up-your-environment)
    - [Visual Text Editor](#visual-text-editor)
    - [A Working Directory](#a-working-directory)
    - [A Network Device](#a-network-device)
    - [Testing Ansible](#testing-ansible)
  - [Ansible Configuration File](#ansible-configuration-file)
  - [Ansible Inventories](#ansible-inventories)
    - [The Inventory File](#the-inventory-file)
      - [The Formats](#the-formats)
        - [INI](#ini)
        - [YAML](#yaml)
      - [The Concepts](#the-concepts)
        - [INI](#ini-1)
        - [YAML](#yaml-1)
  - [The Ansible Playbook](#the-ansible-playbook)
    - [Creating A Playbook.](#creating-a-playbook)
    - [Running A Playbook](#running-a-playbook)
      - [The Output](#the-output)
    - [Ansible Facts](#ansible-facts)
    - [Debugging Outputs](#debugging-outputs)
      - [The Output](#the-output-1)
    - [Ansible Templates](#ansible-templates)
  - [Wrapping Up](#wrapping-up)

## Setting Up Your Environment

In this post I will not be covering the installation of Ansible itself. I would recommend checking out [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html), select your OS and follow the steps and this should help get you all set up.

In order to make this easier we'll first need to get a few things squared away.

<br>

#### Visual Text Editor

The single most powerful tool (next to Ansible) that you'll have! There are many options out there but by far [Microsoft Visual Studio Code](https://code.visualstudio.com/) is the most popular, along with honorable mentions of [Sublime Text](https://www.sublimetext.com/) and [Atom](https://atom.io/).

Whatever your choice is you'll quickly find that the text editor is extremely useful and can really make things easier.

I'll be using Microsoft VSCode and along with this I have the following highly recommended extensions:

- YAML
- Better Jinja (Samuel Colvin)
- Prettier - Code Formatter

<br>

#### A Working Directory

To keep everything organized let's create a directory to store our configurations, playbooks, inventory and other objects.

In this example I am creating the "ansible_101" folder on my user's Desktop:

```
   amack@MacBook-Pro ~ % mkdir ~/Desktop/ansible_101
   amack@MacBook-Pro ~ % cd ~/Desktop/ansible_101
   amack@MacBook-Pro ansible_101 % ls
   amack@MacBook-Pro ansible_101 %
```

<br>

#### A Network Device

To test Ansible we'll need a network device. This can be a physical device or a virtual device hosted in GNS3, EVE-NG, VIRL, etc.

Make sure that this device is reachable by the device you have Ansible configured otherwise you will not be able to manage it.

<br>

#### Testing Ansible

We'll need to make sure that Ansible is properly installed, we'll check this by running a simple command:

`ansible --version`

And the output should look like this:

```
amack@MacBook-Pro ansible_101 % ansible --version
ansible 2.9.15
  config file = /Users/amack/Desktop/ansible_101/ansible.cfg
  configured module search path = ['/Users/amack/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/python@3.7/3.7.9_3/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.7.9 (default, Feb  1 2021, 20:09:54) [Clang 12.0.0 (clang-1200.0.32.29)]
```

<br>
If this does not show up you may need to circle back on the install steps and see if anything was missed.
<br>

## Ansible Configuration File

<br>
The first file we'll create is the Ansible configuration file (ansible.cfg). This file contains settings such as checking ssh keys, ssh pipelining and connection timeouts among many other settings.

There is a default one typically found in /etc/ansible/ansible.cfg but I like to customize mine and keep it relative to the playbooks I am working with.

Here is an example ansible.cfg file:

```
[defaults]
host_key_checking = False
stdout_callback = yaml
roles_path = ./roles

[paramiko_connection]
look_for_keys = False

[ssh_connection]
pipelining = True

[persistent_connection]
connect_timeout = 15
command_timeout = 20
```

<br>
Here are some quick notes on the important settings from above:

- **host_key_checking**: Tell Ansible not to check the validity of the host key. This is up to you and in certain production environments this could be a security issue.
- **pipelining**: Reduces the amount of SSH connections that are spawned to a host, this helps speed up job executions.
- **connect_timeout**: How long until Ansible times out the connection attempt.
- **command_timeout**: How long until Ansible times out a command attempt.

## Ansible Inventories

When we run a playbook, which we'll discuss further on, you will more often than not need an inventory to tell it what hosts to execute the tasks on.

The default location for this will be "/etc/ansible/hosts" but we will be creating one in our directory we created above.

<br>

### The Inventory File

Before we dive into creating the file, let's take a look at some of the formats and concepts of the inventory file to get a better understanding.
<br>
<br>

#### The Formats

The two most common ways to format your inventory is INI and YAML formats. The former likely being the most recognized and the latter being written in the same language as Ansible playbooks.

Whatever you feel comfortable going with is fine, to some YAML is an unfamiliar language so they'll stick with INI and there is nothing wrong with that.

Before we start I'd like to first point you to [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) that will show you more detail, but I hope to simplify it even more to make it easier to comprehend.

###### INI

```
[datacenter1]
dc1_arista1 ansible_host=192.168.10.1 ansible_network_os=eos
dc1_arista2 ansible_host=192.168.10.2 ansible_network_os=eos

[datacenter1:vars]
ansible_user="admin"
ansible_password="admin"
ansible_connection=network_cli
ansible_become=yes
ansible_become_method=enable
```

<br>

###### YAML

```yaml
all:
  children:
    datacenter1:
      hosts:
        dc1_arista1:
          ansible_host: 192.168.10.1
          ansible_network_os: eos
        dc1_arista2:
          ansible_host: 192.168.10.2
          ansible_network_os: eos
      vars:
        ansible_user: "admin"
        ansible_password: "admin"
        ansible_connection: network_cli
        ansible_become: yes
        ansible_become_method: enable
```

<br>

#### The Concepts

Now that we've seen the format and layouts of an inventory file, let's dissect them to get a better understanding.

<br>

###### INI

<img src="/assets/images/ansible_getting_started/ansible_inventory.png" />

- **group**: how you control separation of duties such as different data centers, prod versus dev, switches versus routers, etc
- **hostname**: the name that will be shown in Ansible outputs for this host
- **host_vars**: these are variables that apply only to a specific host
- **ansible_host**: the IP or FQDN of the host that tells Ansible how to reach it. **REQUIRED**
- **ansible_network_os**: Tells Ansible what OS plugin to use in the backend. **REQUIRED**

> For ansible_network_os there are a lot of options, please refer to [Ansible Network Platform Index](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.html#settings-by-platform)

<br>
<br>

Now we'll cover the second half that is the group_vars section:
<br>
<br>

<img src="/assets/images/ansible_getting_started/group_variables.png" />

- **group_vars**: specifying the variables that apply to a specific group, any device in the group "datacenter1" will automatically inherit these values.
- **ansible_user**: the user Ansible will connect as
- **ansible_password**: the password Ansible will connect with
- **ansible_connection**: the plugin that we'll use, for SSH connections we'll use "network_cli"
- **ansible_become**: instructs Ansible to escalate privileges to make changes to the device
- **ansible_become_method**: what method Ansible will use to become privileged user, for Cisco and others this will likely be "enable".

<br>

###### YAML

<img src="/assets/images/ansible_getting_started/ansible_inventory_yaml.png" />
Similar to the INI style above, we'll touch on the few differences:

- **all**: this is implicit in the INI but is included in the YAML one, running a playbook against "all" will include all groups and hosts in the inventory.
- **children**: specifies the groups that belong to this top-level group, you could have child groups under datacenter1 such as switches, routers, firewalls, etc.
- **hosts**: simply lists the hosts that are in a group.

## The Ansible Playbook

Now we get to the fun part! Let's create our first playbook in the directory that we created above. By now you should have the following 2 files in that directory:

1. ansible.cfg
2. inventory

<br>

### Creating A Playbook.

Let's create a file called "facts.yml" in the same directory as the 2 files above and we'll put in the following text:

```yaml
- name: Collect Network Facts
  hosts: datacenter1
  gather_facts: yes
```

- **name**: name of the playbook to distinguish what it does, always name your tasks.
- **hosts**: the name of the individual host or group to run against, I will run against my datacenter1 group from above.
- **gather_facts**: facts are various pieces of information that will be pulled from the device such as:
  - serial number
  - version
  - hardware model
  - running configuration

<br>

Save this file and open a terminal in your text editor or in the terminal app and navigate to the directory you stored the files in and we'll run this playbook.

> YAML is very picky when it comes to indentation so use double spaces instead of tabs. If you want to save yourself the headache, set your text editor to indent your tab to 2 spaces and make sure to use the YAML plugin.

### Running A Playbook

To run this playbook we will run the command:

`ansible-playbook backup.yml -i inventory`

The "-i inventory" argument is to specify the inventory file that we created above rather than using the default found in /etc/ansible/hosts.

#### The Output

<img src="/assets/images/ansible_getting_started/first_playbook.png" />

The first thing we'll see is the PLAY which I have labelled "Collect Network Facts".

After that we will see TASK then the tasks that will be executed, in this case we are just Gathering Facts on the 2 devices in my inventory. In a real playbook this could be anyhwere from 1 to 10+ tasks.

Finally we have the Play Recap which shows us the overall status of the playbook. If you get a green OK that is perfect, that means Ansible can reach your device and parse it out. If you receive any errors check your password and username as well as the IP of the device.

Awesome, you ran your first playbook but where are all of these "facts"? Well during a playbook facts are hidden behind the scenes or in the case of Ansible Tower they can be stored in a database.

In this next section we will first review the facts structure then we will show the outputs from the fact gathering.
<br>

### Ansible Facts

Before we go about printing all of the facts let's first take a look at the output structure of the facts.

For the majority of network devices in Ansible, there are a handful of facts that share the same variable name. This allows us to reference a similar variable to pull a version or serial number, for example.

Here are a few that share the same naming scheme:

- **ansible_net_hostname**: the configured hostname of the device
- **ansible_net_config**: the running configuration of the device
- **ansible_net_model**: the hardware model of the device
- **ansible_net_serialnum**: the serial number of the device
- **ansible_net_version**: the OS version of the device

There are many more that share the "ansible_net_X" scheme, but these are the basic ones that most engineers will typically use when starting out.

To see all of the facts for a platform you can take a look at the "**os**\_facts" module. So for example if we are working on an Arista it will be "eos_facts" or for a Cisco Nexus it would be "nxos_facts".

The quickest way to find these modules is to Google "ansible arista facts" or "ansible nexus facts".

> Don't worry about the modules just yet, we'll discuss these in another post. They can be overwhelming at first glance but just scroll down to "Examples" and you can get a clearer picture of how it would look in a playbook.

### Debugging Outputs

Sometimes we need to print something to the output to display a message or to debug a variable or output, enter the debug module.

We're going to use the same facts.yml playbook above and we're going to add a task with the debug module to print out the version and models of our devices:

```yaml
- name: Collect Network Facts
  hosts: datacenter1
  gather_facts: yes

  tasks:
    - name: Print Device Version
      debug:
        msg: {% raw %}"{{ ansible_net_version }}" {% endraw %}

    - name: Print Device Model
      debug:
        msg: {% raw %}"{{ ansible_net_model }}" {% endraw %}
```

<br>

First let's talk about the elephant in the room, the curly braces that you see around the variable we're calling.

There is another way to debug that doesn't have the curly braces, but this shows you what it looks like to reference variables in various modules in a playbook.

When referring to a variable in Ansible we must have quotes and double curly braces around it. This is because Ansible is using [Jinja2](https://jinja.palletsprojects.com/en/2.11.x/) for variable rendering.

> We'll discuss Jinja2 more in-depth in another post when we start talking about filters, templating and other related topics.

Next I want to touch on a few things you might have noticed:

1. Ansible runs these tasks in order and will execute the task on all hosts as required before moving on.
2. Since facts are host_vars, or variables that are specific to only one host, we can get varying information from them. What if we just called them switch1 and switch2 with switch1 being an Arista and switch2 being a Cisco Nexus? They would have different values of course because of their different platforms.

<br>

##### The Output

<img src="/assets/images/ansible_getting_started/first_playbook_variables.png" />

> If your playbook broke but worked with gathering facts before we added these steps, it is very likely you did not format it correctly. If it helps copy and paste my code and make sure to match the spacing.

Notice the version "4.24.1.1F" and the model "vEOS" being printed to the screen, see how easy that was to pull something simple from a device?

This is a simple win but imagine once you had your inventory built out how quick you could get Ansible to pull all of this information and compile an inventory report.

Let's try it!
<br>

### Ansible Templates

One last thing I wanted to show but not go too in-depth on is a simple template example. If you are not familiar with Jinja2 it is a templating language that is very easy to use to model out a report or even device configuration code.

Before we start pushing changes to our devices let's take a safer approach and just render out a very simple inventory report.

Create a file called "report.j2" in the same directory as the other files and add this to it:

```jinja
{% raw %}
Host: {{ inventory_hostname }}
Version: {{ ansible_net_version }}
Model: {{ ansible_net_model }}
Serial#: {{ ansible_net_serialnum }}
{% endraw %}
```

<br>

Create a file called report.yml and put the following in it:

```yaml
- name: Print Device Report
  hosts: datacenter1
  gather_facts: yes

  tasks:
    - name: Print Device Report
      debug:
        msg: {% raw %}"{{ lookup('template','report.j2') }}"{% endraw %}
```

<br>

Now we'll render the template out with the facts that we pulled from our devices.

In this example I'll add a Nexus device just to show the difference that will be output:

<br>

```
TASK [Print Device Report] ******************************************
ok: [nexus1] =>
  msg: |-
    Host: nexus1
    Version: 7.0(3)I7(7)
    Model: Nexus9000 9000v Chassis
    Serial#: 9ZE3BGXG0XL

ok: [arista1] =>
  msg: |-
    Host: arista1
    Version: 4.24.1.1F
    Model: vEOS
    Serial#:
```

<br>

Pretty cool right? Even though one is an Arista and one is a Nexus, we can use the same template file and same variable names. Templating with Ansible is incredibly simple but I still want to touch on a few things:

- Jinja2 files do not require quotes around variables, only the double curly braces. Ansible requires the quotes and double curly braces around variables all the time except for conditional statements.

- The lookup command is a plugin that is used to access data from other sources such as a jinja2 file, a text file, a JSON file, etc. We're telling it to render a template and the file is report.j2.

- inventory_hostname is a [special variable](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) that cannot be set by a user, but in this case it references the name given to the host in inventory

## Wrapping Up

I covered a lot in this post but I hope it helped make the approach to Ansible a little easier to understand. With all tools and processes there are learning curves but with Ansible once you're past that initial learning phase it is really simple to use and can really make your life easier.

I'll be creating more posts with more in-depth examples of different aspects of Ansible such as:

- host_vars and group_vars
- filters and plugins
- templating
- lists, dictionaries and loops

Thanks for checking this post out and feel free to add me on [LinkedIn!](https://www.linkedin.com/in/adam-mack07/)
