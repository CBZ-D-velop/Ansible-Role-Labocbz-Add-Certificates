# Ansible role: labocbz.add_certificates

![Licence Status](https://img.shields.io/badge/licence-MIT-brightgreen)
![CI Status](https://img.shields.io/badge/CI-success-brightgreen)
![Testing Method](https://img.shields.io/badge/Testing%20Method-Ansible%20Molecule-blueviolet)
![Testing Driver](https://img.shields.io/badge/Testing%20Driver-docker-blueviolet)
![Language Status](https://img.shields.io/badge/language-Ansible-red)
![Compagny](https://img.shields.io/badge/Compagny-Labo--CBZ-blue)
![Author](https://img.shields.io/badge/Author-Lord%20Robin%20Crombez-blue)

## Description

![Tag: Ansible](https://img.shields.io/badge/Tech-Ansible-orange)
![Tag: Debian](https://img.shields.io/badge/Tech-Debian-orange)
![Tag: SSL/TLS](https://img.shields.io/badge/Tech-SSL%2FTLS-orange)
![Tag: CA](https://img.shields.io/badge/Tech-CA-orange)

An Ansible role install a CA your server and/or get cryptographic content.

The Ansible role enables the import of SSL certificate bundles, which are files either in ZIP format or any other archive format, both locally and remotely. When the type of bundle is specified as a CA (Certificate Authority), the role automatically includes it in the cacerts of the target machines, enhancing the security and trust of SSL communications.

In the context of SSL communication, Certificate Authorities play a crucial role in establishing trust between clients and servers. When a certificate is signed by a trusted CA, it verifies the authenticity and integrity of the certificate, enabling secure connections between systems. The role's ability to automatically add the CA bundle to the cacerts ensures that the target machines trust the certificates issued by that particular Certificate Authority.

By seamlessly updating the cacerts with the CA bundle, the role streamlines the process of establishing trust between the target system and the Certificate Authority. This ensures that SSL connections are secured and authenticated, allowing clients to confidently trust the certificates presented by the servers.

In addition to handling CA bundles, the role efficiently manages both local and remote files, performing necessary fetch and placement operations. The inclusion of basic authentication support enhances the security of remote file handling, ensuring that SSL certificate bundles are retrieved securely.

In summary, the Ansible role provides a comprehensive solution for SSL certificate management, offering flexibility in handling various types of bundles, including CAs. The automatic inclusion of CA bundles in the cacerts of target machines streamlines the trust establishment process, ensuring secure and authenticated SSL connections within the environment.

## Folder structure

By default Ansible will look in each directory within a role for a main.yml file for relevant content (also man.yml and main):

```PYTHON
.
├── README.md  # Contains an overview of the role and its purpose.
├── defaults
│   ├── main.yml  # Contains default variables for the role that can be overridden by users.
│   └── README.md  # Contains documentation for the default variables.
├── files
│   └── README.md  # Contains documentation for the files in the directory.
├── handlers
│   ├── main.yml  # Contains handlers that can be called by tasks within the role.
│   └── README.md  # Contains documentation for the handlers.
├── meta
│   ├── main.yml  # Contains metadata about the role, including dependencies and supported platforms.
│   └── README.md  # Contains documentation for the role metadata.
├── tasks
│   ├── main.yml  # Contains tasks to be executed by the role on the managed nodes.
│   └── README.md  # Contains documentation for the tasks.
├── templates
│   └── README.md  # Contains documentation for the templates.
└── vars
    ├── main.yml  # Contains variables that are specific to the role and are not meant to be overridden.
    └── README.md  # Contains documentation for the role variables.
```

## Tests and simulations

### Basics

You have to run multiples tests. *tests with an # are mandatory*

```MARKDOWN
# lint
# syntax
# converge
# idempotence
# verify
side_effect
```

Executing theses test in this order is called a "scenario" and Molecule can handle them.

Molecule use Ansible and pre configured playbook to create containers, prepare them, converge (run the role) and verify its execution.
You can manage multiples scenario with multiples tests in order to get a 100% code coverage.

This role contains a ./tests folder. In this folder you can use the inventory or the tower folder to create a simualtion of a real inventory and a real AWX / Tower job execution.

### Command reminder

```SHELL
# Check your YAML syntax
yamllint -c ./.yamllint .

# Check your Ansible syntax and code security
ansible-lint --config=./.ansible-lint .

# Execute and test your role
molecule lint
molecule create
molecule list
molecule converge
molecule verify
molecule destroy

# Execute all previous task in one single command
molecule test
```

## Installation

To install this role, just copy/import this role or raw file into your fresh playbook repository or call it with the "include_role/import_role" module.

## Usage

### Vars

Some vars a required to run this role:

```YAML
---
add_certificates__tmp_workspace: "/tmp/add_certificates"
add_certificates__ca_location: "/usr/local/share/ca-certificates"

add_certificates__bundle_name: "My-CA-Authority"
add_certificates__bundle_type: "CA"
add_certificates__bundle_src: "/tmp/ssl/{{ add_certificates__bundle_name }}.zip"
add_certificates__bundle_dest: "/tmp/deployed_ssl"
add_certificates__bundle_src_user: "myUser"
add_certificates__bundle_src_password: "secretPassword"
add_certificates__bundle_dest_user: "root"
add_certificates__bundle_dest_group: "root"
add_certificates__bundle_dest_mode: "0755"

add_certificates__artifactory_login: "myUser"
add_certificates__artifactory_password: "myUserPassword"

```

The best way is to modify these vars by copy the ./default/main.yml file into the ./vars and edit with your personnals requirements.

You can set vars in the template model in Ansible AWX / Tower or just surchage them during the playbook call.

In order to surchage vars, you have multiples possibilities but for mains cases you have to put vars in your inventory and/or on your AWX / Tower interface.

```YAML
# From inventory
inv_add_certificates:
  - name: "My-Local-Ansible-Root-CA.pem.crt"
    type: "CA"
    src: "/tmp/My-Local-Ansible-Root-CA.pem.crt"

  - name: "My-Local-Ansible-Intermediate-CA-1.pem.crt"
    type: "CA"
    src: "/tmp/My-Local-Ansible-Intermediate-CA-1.pem.crt"

  - name: "My-Local-Ansible-Intermediate-CA-2.pem.crt"
    type: "CA"
    src: "/tmp/My-Local-Ansible-Intermediate-CA-2.pem.crt"

  - name: "my-cluster.domain.tld"
    src: "/tmp/my-cluster.domain.tld.zip"
    dest: "/tmp/ssl"
    type: "cert"
    #src_user: "***"
    #src_password: "***"
    dest_user: "root"
    dest_group: "root"
    dest_mode: "0755"
```

```YAML
# From AWX / Tower
---

```

### Run

To run this role, you can copy the molecule/default/converge.yml playbook and add it into your playbook:

```YAML
- name: "Include labocbz.add_certificates"
  tags:
  - "labocbz.add_certificates"
  loop: "{{ inv_cert_bundles }}"
  loop_control:
    loop_var: bundle
  when: inv_cert_bundles is defined
  vars:
    add_certificates__bundle_name: "{{ bundle.name }}"
    add_certificates__bundle_type: "{{ bundle.type }}"
    add_certificates__bundle_src: "{{ bundle.src }}"
    add_certificates__bundle_dest: "{{ bundle.dest }}"
    add_certificates__bundle_src_user: "{{ bundle.src_user }}"
    add_certificates__bundle_src_password: "{{ bundle.src_password }}"
    add_certificates__bundle_dest_user: "{{ bundle.dest_user }}"
    add_certificates__bundle_dest_group: "{{ bundle.dest_group }}"
    add_certificates__bundle_dest_mode: "{{ bundle.dest_mode }}"
  ansible.builtin.include_role:
    name: "labocbz.add_certificates"
```

## Architectural Decisions Records

Here you can put your change to keep a trace of your work and decisions.

### 2023-05-27: First Init

* First init of this role with the bootstrap_role playbook by Lord Robin Crombez

### 2023-06-01: Adding credentials for remote files

* Role handle now remote auth for protected urls

### 2023-06-02: Deploy bundles

* Import/Deploy ZIP bundles on hosts for tests, validation, backup, etc

### 2023-10-06: New CICD, new Images

* New CI/CD scenario name
* Molecule now use remote Docker image by Lord Robin Crombez
* Molecule now use custom Docker image in CI/CD by env vars
* New CICD with needs and optimization

### 2023-02-20: New CI, new Images

* Added support for Ubuntu 22
* Added support for Debian 11/12
* Added new CI
* Refresh vars __

### 2024-05-19: New CI

* Added Markdown lint to the CICD
* Rework all Docker images
* Change CICD vars convention
* New workers
* Removed all automation based on branch

## Authors

* Lord Robin Crombez

## Sources

* [Ansible role documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
* [Ansible Molecule documentation](https://molecule.readthedocs.io/)
