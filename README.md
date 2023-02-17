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

## Creating VPC Play

```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: Import vpc variables
      include_vars: vars/vpc_setup
    - name: create vpc
      ec2_vpc_net:
        name: "{{ vpc_name }}"
        cidr_block: "{{ vpcCidr }}"
        region: "{{ region }}"
        dns_hostnames: yes
        dns_support: yes
        tenancy: default
        state: "{{ state }}"
      register: vpcout
      when: vpcout.changed
    
        
```

![image](https://user-images.githubusercontent.com/96833570/219666502-3809bf70-b866-42a8-9507-1b460ab5eced.png)


## Creating Subnets Play


```
# Note: Subnet1 play
    - name: create public subnet 1 in zone 1
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id}}"
        cidr: "{{ PubSub1Cidr }}"
        region: "{{ region }}"
        az: "{{ zone1 }}"
        state: "{{ state }}"
        resource_tags:
          Name: ada-pubsub1

      register: pubsub1_out

# Note: Subnet2 play
    - name: create public subnet 2 in zone 2
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id}}"
        cidr: "{{ PubSub2Cidr }}"
        region: "{{ region }}"
        az: "{{ zone2 }}"
        state: "{{ state }}"
        resource_tags:
          Name: ada-pubsub2
      register: pubsub2_out

# Note: Subnet3 play
    - name: create public subnet 3 in zone 3
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id}}"
        cidr: "{{ PubSub3Cidr }}"
        region: "{{ region }}"
        az: "{{ zone3 }}"
        state: "{{ state }}"
        resource_tags:
          Name: ada-pubsub3
      register: pubsub3_out

# Note: PrivSubnet1 play
    - name: create public priv subnet 1 in zone 1
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id}}"
        cidr: "{{ PrivSub1Cidr }}"
        region: "{{ region }}"
        az: "{{ zone1 }}"
        state: "{{ state }}"
        resource_tags:
          Name: ada-privsub1
      register: privsub1_out

# Note: PrivSubnet2 play
    - name: create public priv subnet 2 in zone 2
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id}}"
        cidr: "{{ PrivSub2Cidr }}"
        region: "{{ region }}"
        az: "{{ zone2 }}"
        state: "{{ state }}"
        resource_tags:
          Name: ada-privsub2
      register: privsub2_out

# Note: PrivSubnet2 play
    - name: create public priv subnet 3 in zone 3
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id}}"
        cidr: "{{ PrivSub3Cidr }}"
        region: "{{ region }}"
        az: "{{ zone3 }}"
        state: "{{ state }}"
        resource_tags:
          Name: ada-privsub3
      register: privsub3_out

        
```
## Creating igw and the route table

```
# Note: igw
    - name: Create Internet gateway
      amazon.aws.ec2_vpc_igw:
        vpc_id: "{{ vpcout.vpc.id}}"
        region: "{{ region }}"
        state: "{{ state }}"
        resource_tags:
        Name: ada-igw
      register: igw_out

# Note: create route table

    - name: Set up public subnet route table
      amazon.aws.ec2_vpc_route_table:
        vpc_id: "{{ vpcout.vpc.id}}"
        region: "{{ region }}"
        tags:
          Name: Ada-route-table
        subnets:
          - "{{ pubsub1_out.subnet.id }}"
          - "{{ pubsub2_out.subnet.id }}"
          - "{{ pubsub3_out.subnet.id }}"
        routes:
          - dest: 0.0.0.0/0
            gateway_id: "{{ igw.gateway_id }}"
      register: pubRT_out
```

![image](https://user-images.githubusercontent.com/96833570/219719816-a0e803a2-27b9-4778-8b8a-02088c4002c0.png)


![image](https://user-images.githubusercontent.com/96833570/219750701-a651a1e4-07c2-49fe-a0e5-e87fc4841e36.png)

