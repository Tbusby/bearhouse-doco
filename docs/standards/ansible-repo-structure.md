# Ansible Repository Structure

## Overview

The Ansible repository is structured to support repeatable host configuration and Kubernetes node preparation for the homelab platform.

The current layout separates:

- configuration
- inventory
- playbooks
- group variables
- roles

This provides a clean and extensible structure suitable for continued growth.

## Repository Tree

```text
.
├── .ansible
│   ├── collections
│   ├── modules
│   └── roles
├── .ansible-lint
├── .gitignore
├── README.md
├── ansible.cfg
├── bootstrap.sh
├── group_vars
│   ├── all.yml
│   ├── k8s_cluster.yml
│   ├── k8s_control.yml
│   ├── k8s_workers.yml
│   └── ops1.yml
├── inventory
│   └── lab
│       └── hosts.yml
├── playbooks
│   ├── baseline.yml
│   ├── bootstrap.yml
│   ├── k8s_install.yml
│   ├── k8s_prereqs.yml
│   └── site.yml
├── requirements.yml
└── roles
    ├── baseline
    │   ├── defaults
    │   │   └── main.yml
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    ├── k8s_install
    │   ├── defaults
    │   │   └── main.yml
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    └── k8s_prereqs
        ├── defaults
        │   └── main.yml
        ├── handlers
        │   └── main.yml
        └── tasks
            └── main.yml
