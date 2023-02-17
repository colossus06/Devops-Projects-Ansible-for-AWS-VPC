# Devops-Projects-Ansible-for-AWS-VPC

## Requirements

* boto

* boto3

* python >= 2.6

ref: https://docs.ansible.com/ansible/2.9/modules/ec2_key_module.html#examples

## Creating the playbook

```
sudo apt update
sudo apt install python3-boto -y
sudo apt install python3-boto3 -y
```

```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: create a new ec2 key pair, returns generated private key
      ec2_key:
        name: my_keypair
      register: keyout

    - debug:
        var: keyout
    - name: store login priv key
      copy:
        content: "{{ keyout.key.private_key }}"
        dest: ./mykey.pem
      when: keyout.changed


```

![image](https://user-images.githubusercontent.com/96833570/219637363-f4153b0d-6523-4699-9327-058ff2beb553.png)

![image](https://user-images.githubusercontent.com/96833570/219638921-8bccabf5-765c-4c6c-b997-7a5414ccbf85.png)

![image](https://user-images.githubusercontent.com/96833570/219643360-4462f52f-e4bf-4a9a-bdc7-b56fcda26215.png)


I will be deleted this key pair, no worries.

Since we added `when: keyout.changed` line we won't get an error in future playbook runs.

![image](https://user-images.githubusercontent.com/96833570/219644766-12ec76ac-dcb3-45ae-8c55-be8af460aa57.png)


