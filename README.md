
## Automated ELK Stack Deployment
The files in this repository were used to configure the network depicted below.
![TODO: Update the path with the name of your diagram](Images/diagram_filename.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

[Installing Elk](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/install-elk.yml)

[Filebeat Playbook](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/filebeat.yml)


[Metricbeat Playbook](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/metricbeat.yml)


This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build
### Description of the Topology
The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.
Load balancing ensures that the application will be highly accessible, in addition to restricting entrance to the network.
The Load balancers have security aspects to protect Applications from emerging threats , Authenticate User Access, Simplify PCI compliance, and protect against DDos attack. [1](https://lumecloud.com/what-does-a-load-balancer-do/)

Jumpbox has the advantage of improving productivity by making it possible to work on multiple systems without the time-wasting process of logging out and logging back into each privileged area. It also enhances security by separating the user's workstation that is susceptible to being compromised and the privileged within the network. The separation helps to isolate privileged assets from being in contact with potentially sensitive areas. [2](https://securityboulevard.com/2020/02/privileged-access-workstation-vs-jump-server/#:~:text=Improve%20security%3A%20Jump%20servers%20create,contact%20with%20potentially%20compromised%20workstations.)


**Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the log files and system metrics.**

The configuration details of each machine may be found below.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.1.6   | Linux            |
| Web 1    | Gateway  | 10.0.1.4   | Linux              |
| Web 2    | Gateway  | 10.0.1.5   | Linux               |
| Web 3    | Gateway  | 10.0.0.5   | Linux 
|Project-1-ELK| Gateway  | 10.1.0.4   | Linux

              
### Access Policies
The machines on the internal network are not exposed to the public Internet. 
Only the jump box provisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following **IP address 50.99.147.170 which is the host IP address
- _TODO: Add whitelisted IP addresses_**
Machines within the network can only be accessed by the Jump box provisioner.

The ELK VM Machine can be accessed from the Jump Box machine that will connect to the Jumpbox container (cool_bassi) which can connect to the ELK VM. The public IP address for the Jump Box is: 52.188.9.8 , while the private IP address is: 10.0.1.6

A summary of the access policies in place can be found in the table below.

| Name        | Publicly Accessible| Allowed IP Addresses |
|-------------|--------------------|----------------------|
|Jump Box     |     ?              | ?                    |
|Load Balancer|     Yes            | **52.146.32.179**    |
|Web 1        |     No             | 10.0.1.4             |
|Web 2        |     No             | 10.0.1.5             |
|Web 3        |     No             | 10.0.0.5             |
|Project-1-ELK|     **Yes**        | 10.1.0.4             |



### Elk Configuration
Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it frees the focus on efforts on tasks that are more important, agentless, Idempotent, and declarative not Procedural.

The playbook implements the following tasks:
1. cofinguruation of Web VM with Docker

```
---
- name: Config Web VM with Docker
  hosts: elkservers
  remote_user: sysadmin
  become: true
  tasks:

  ```
2. Installation of docker.io, pip3, and docker
```
- name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      update_cache: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

```

3. Downloading and launching a docker web container

```
 - name: download and launch a docker web container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
      published_ports: 
        - 5601:5601
        - 9200:9200
        - 5044:5044
```

4. Enabling docker service
```
  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes
```

5. Increasing virtual memory
```
  - name: Use more memory
    sysctl:
      name: vm.max_map_count
      value: "262144"
      state: present
      reload: yes
```

You can access the full script from here: [install-elk.yml](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/install-elk.yml)

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.


![Alt text](https://github.com/Zeinab-ajh/project-1/blob/main/Images/docker%20ps%20output.png)


### Target Machines & Beats
This ELK server is configured to monitor the following machines:
1. Web 1: 10.0.1.4
2. Web 2: 10.0.1.5
3. Web 3: 10.0.0.5 

We have installed the following Beats on these machines:
* Filebeat
* Metricbeat

Filebeat allows us to collect log events and forward them to either Logstash or Elasticssearch. For example, it is expected to see files that are generated by MySQL databases and Apache. 

Metricbeat will gather metrics and statistics to monitor the server's performance. For example, Metricbeat can analyze and monitor memory and CPU.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 
SSH into the control node and follow the steps below:

* Run `nano hosts` and update the `/etc/ansible/hosts` file by adding the IP addressess of the Virtual machines, and specify the machine that will install the ELK server on [elkservers], and which to install Filbeat on [webservers]

```
# Ex 2: A collection of hosts belonging to the 'webservers' group

[webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110
10.0.1.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.1.4 ansible_python_interpreter=/usr/bin/python3

[elkservers]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
```

* Copy the playbook files "[filebeat.yml](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/filebeat.yml) and [metricbeat.yml](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/metricbeat.yml)" to* `/etc/ansible/roles`*

* Filebeat
1. Copy the filebeat-config.yml to `/etc/ansible/roles/files` directory after logging into the docker container from the Jumpbox provisioner.

Run the following:
    
```
- Jumpbox login: *ssh azureuser@[Public IP]*
- Start the docker container: *docker start [container name]*
- Attach to the docker container: *docker attach [container name]*
- cp filebeat-config.yml to /etc/ansible/roles/files
```

**Note: Run the first three commands if you are not logged into the docker container.**

2. Run: `nano filebeat-config.yml` in `/etc/ansible/roles/files` directory and update the [filebeat-config.yml](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/filebeat-config.yml) to include the private IP address of ELK servers in line #1106 & #1806.

3. Run the playbook by running *`playbook-ansible filebeat-config.yml` *in `/etc/ansible/roles/files directory`.

![Alt text](https://github.com/Zeinab-ajh/project-1/blob/main/Images/filebeat.yml%20output.png)


* Metricbeat
1. Copy the metricbeat-config.yml to /etc/ansible/roles/files directory after logging into cool_bassi docker container from the Jumpbox provisioner.

Run the following:
    
```
- Jumpbox login: *ssh azureuser@[Public IP]*
- Start the docker container: *docker start [container name]*
- Attach to the docker container: *docker attach [container name]*
- cp metricbeat-config.yml to /etc/ansible/roles/files
```

**Note: Run the first three commands if you are not logged into the docker container.**

2. Run: `nano metricbeat-config.yml` in `/etc/ansible/roles/files` directory and update the [metricbeat-config.yml](https://github.com/Zeinab-ajh/project-1/blob/main/Ansible/metricbeat-config.yml) to include the private IP address of ELK servers in line #62 & #96.


3. Run the playbook by running *`playbook-ansible metricbeat-config.yml` *in `/etc/ansible/roles/files` directory.

![Alt text](https://github.com/Zeinab-ajh/project-1/blob/main/Images/metricbeat.yml%20output.png)


* Navigate to http://13.64.19.9:5601/app/kibana to check that the ELK server is running.
      **Note: Use your ELK public IP address**

![Alt text](https://github.com/Zeinab-ajh/project-1/blob/main/Images/Filebeat%20module%20status.png)

![Alt text](https://github.com/Zeinab-ajh/project-1/blob/main/Images/Metricbeat%20module%20status.png)
