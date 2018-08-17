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
PrivateSubnet1B: subnet-12345678      # prod
PrivateSubnet1C: subnet-12345679      # prod
vpc_id: vpc-12345678                  # prod
AWSRegion: eu-west-1

ec2_pets:
  group-apptier01-env:                # This will be the server name tag
    description: "App Tier server for {{ Application  }}"
    ssh_key: "my-ssh-key-dev"
    instance_type: t2.micro
    instance_ami: ami-e365fd9a
    limits:                           # Add limits.conf / limits.d
        limits_group:
          domain: "@groupname"
          limit_item: "nofile"
          limit_type: "-"
          value: "65536"
          dest: "/etc/security/limits.d/99-application.conf"
    security_groups: ['groupname_appname_env_sg']
    volumes:
      - device_name: /dev/xvdb
        volume_type: gp2
        volume_size: 20
        mount_point: /usr/sap
        fstype: xfs
        encrypt: "yes"
        resizefs: yes
    monitoring: "yes"
    termination_protection: "no"
    wait: "yes"
    vpc_subnet_id: "{{ PrivateSubnet1A }}"
    yum_packages:
      - sudo
      - "@base"                       # Package group
```

Playbook example:
-----------------
```cat playbook.yaml```
```yaml
---
# This Ansible playbook creates a number AWS instances to be used as server pets.
#
# This module requires that the boto python library is installed, and that Ansible can use awscli.
# You'll also need python boto3 & botocore ("pip install boto3 --user")

- name: "provision EC2 server pets"
  hosts: localhost
  connection: local
  gather_facts: false

  pre_tasks:
    - name: "Include global application variables"
      include_vars: "vars/myvars.yaml"
      tags:
        - ec2_instances

  roles:
    - { role: ec2-groups, tags: ["ec2_groups"] }
    - { role: ec2-instance, tags: ["ec2_instances"] }


- name: Configure provisioned instances (dynamically added by ec2-instance role)
  hosts: ec2_pets
  become: yes
  gather_facts: true
  pre_tasks:
    - name: "Include global application variables"
      include_vars: "vars/myvars.yaml"
      tags:
        - ec2_instances

  roles:
    - { role: yum_packages, tags: ["ec2_groups"] }                                   # Installs packages in yum_packages
    - { role: block_filesystems, tags: ["block_filesystems"], mntpoint_mode: 0755 }  # Expects volumes as defined in vars
```