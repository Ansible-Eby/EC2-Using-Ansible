# EC2-Using-Ansible
This is an Ansible Playbook to Spin-Up an EC2 instance.

Requirements:
  * python >= 2.6
  * Ansible master server with python-boto3 installed. Ansible's EC2 module uses python-boto library to call AWS API, and boto  needs AWS credentials in order to function.


```---
- name: "Spinning-Up an Ec2 Instance using Ansible"
  hosts: localhost

  vars:
    debug_enabled: false
    key_pair: ohio-ansible
    region: us-east-2
    security_group: ansible-sg
    instance_type: t2.micro
    ami_id: ami-02ccb28830b645a41
    ami_user: ec2-user
    instance_name: webserver

  tasks:
    - name: "KeyPair Generation"
      ec2_key:
        region: "{{ region }}"
        name: "{{ key_pair }}"
        state: present
      register: key_out

    - debug: var=key_out
      when: debug_enabled

    - name: "Saving Key Pair"
      copy:
        content: "{{ key_out.key.private_key }}"
        dest: "{{ key_pair }}.pem"
        mode: "0400"
      when: key_out.changed

    - name: "Security Group Creation"
      ec2_group:
        name: "{{ security_group }}"
        description: "Security Group for Ansible Ec2"
        region: "{{ region }}"
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0

          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0

    - name: "EC2 Instance Creation"
      ec2:
        group: "{{ security_group }}"
        instance_type: "{{ instance_type }}"
        image: "{{ ami_id }}"
        wait: true 
        region: "{{ region }}"
        keypair: "{{ key_pair }}"
        count_tag:
          name: "webserver"
        instance_tags:
          name: "webserver"
        count: 1
      register: ec2

```
