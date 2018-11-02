# efs-share 

Ansible role to create AWS EFS shares.



Variables example
---------

```cat vars/myvars.yaml```
 ```yaml
EnvironmentClass: "dev"
CostCenter: "12345"
Application: "cylon_laser"
OwnerEmail: "helpful.robot@scania.com"
ApplicationTier: "app"
PrivateSubnet1B: subnet-12345678
PrivateSubnet1C: subnet-12345679
vpc_id: vpc-12345678
AWSRegion: eu-west-1

# EFS Parameters
# Uncomment below to create one or more shares.
efs_shares:
  general_purpose_efs01:
    name: "general_purpose_efs01"
    performance_mode: "general_purpose"
    state: present                                                                  # present or absent (absent removes the share!)
    encrypt: "yes"
    targets:
      - { subnet_id: "{{PrivateSubnet1A}}", security_groups: [ "sg-123456789" ] }

```

Playbook example:
-----------------
```cat playbook.yaml```
```yaml
---
#
# This module requires that the boto python library is installed, and that Ansible can use awscli.
# You'll also need python boto3 & botocore ("pip install boto3 --user")
# Execute with parameters: -e "CI_COMMIT_REF_NAME=dev" or similar to source the correct vars/ -file

- name: "provision AWS Objects for {{ CI_COMMIT_REF_NAME }}"
  hosts: localhost
  connection: local
  gather_facts: false

  pre_tasks:
    - name: "Include global application variables vars/vars-global.yaml"
      include_vars: "vars/vars-global.yaml"
      tags:
        - efs_shares
    - name: "Include variables vars/vars-{{ CI_COMMIT_REF_NAME }}.yaml"
      include_vars: "vars/vars-{{ CI_COMMIT_REF_NAME }}.yaml"
      tags:
        - efs_shares
  roles:
    - { role: efs-share, tags: ["efs_shares"] }

```