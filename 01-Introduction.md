# Introduction

Ansible is an Open Source configuration management, software provisioning and application deployment toolset create at 2012 and further acquired by RedHat

It is possible to divide ansible in the follow core components:

- `Modules` - Extensible Library openly available for consumption (Cloud Computing, Networking, Container and much more)
- `Ansible Executable` - A versatile tool that acts as a swiss army knife for automation
- `Ansible PlayBook` - Human readable configuration, deployment and orchestration language (Configuration that is desired to achieve). Simple set of tasks
- `Ansible inventories` - An inventory of targets (Hosts, Network Switches, Containers etc)

Also Ansible is an agentless tools, so in order to it work a secure and trustworthy connectivity should be established between the agent. Usually a SSH connection is used. 