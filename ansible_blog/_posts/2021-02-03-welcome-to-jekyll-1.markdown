---
layout: post
title: "Ansible Networking - The Basics"
date: 2021-02-03 15:59:11 -0700
categories: jekyll update
---

## Getting Started

Getting started with Ansible is exciting but can also be overwhelming for new users. In this post we'll go over setting up the basics as well as getting into more intermediate topics that should allow you to hit the ground running rather than just spinning your wheels.

Don't have Ansible installed already? No problem, visit [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html), select your OS and follow the steps.

### Setting Up Your Environment

Before we get started we should install and configure a few things to make this easier:

#### Visual Text Editor

The single most powerful tool! There are many options out there but by far [Microsoft Visual Studio Code](https://code.visualstudio.com/) is the most popular, along with honorable mentions of [Sublime Text](https://www.sublimetext.com/) and [Atom](https://atom.io/).

Whatever your choice is, this is your ideal method of writing playbooks and working with Ansible related files. I'll be using Microsoft VSCode and along with this I have the following highly recommended extensions:

- YAML
- Better Jinja (Samuel Colvin)
- Prettier - Code Formatter

### Ansible Inventories

When we run a playbook, which we'll discuss further on, you usually need an inventory to tell it what hosts to execute the tasks on.

## Let's take a look at an inventory file

### Fact Collection and Config Parsing
