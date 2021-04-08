# Cloud Network
# This is a collection of Linux scripts and Ansible Scripts to make a Virtual Network with 3 DVWA servers.

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Diagram](https://raw.githubusercontent.com/solidcode747/Cloud-Network/main/Images/Diagram_Final_Renzo_Tragsiel.png?token=AR7YBNHH2E6EOUVPIKW6PDDANZCRI)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the configuration file may be used to install only certain pieces of it, such as Filebeat.

filebeat-config.yml

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly **available, stable, and with plenty of redundancy**, in addition to restricting **inbound access** to the network.  The load balancer works as a traffic manager distributing HTTP traffic to the three DVWA Web Servers as evenly as possible increasing stability.  It also works as a single point of entry for all HTTP traffic to the DVWA Web Servers, making it easier to monitor inbound and outbound traffic by limiting the points of entry to one point, port 80, in the network.
Through the use of a Jumpbox and an additional Ansible Docker-container (inside the Jumpbox), we protect the DVWA Web Servers by only allowing the sysadmin team to be able to access the DVWA Web Servers only through the Ansible container (inside the Jumbox).  This is done by setting up firewall rules that only allow SSH access on port 22 to the jumbox and the Ansible container only to a predetermined IP address, keeping an airtight approach to cloud management.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the files and system resources.
- Filebeat is a system designed to monitor changes to any and all **files** in the DVWA servers on an on-going basis.
- Metricbeat monitors changes to any and all **system resources** in the DVWA Web servers ongoingly.
- Together, Filebeat and Metricbeat and the ELK server increase the security of the servers by carrying out all monitoring activities through their respective containers (See diagram above), further limiting any direct interaction with the DVWA Web Servers, as well as watch system metrics, such as CPU usage; attempted SSH logins; sudo escalation failures; etc.


The configuration details of each machine may be found below.

| Name         | Function  | IP Address | Operating System               |
|--------------|-----------|------------|--------------------------------|
| Jump Box     | Gateway   | 10.0.0.4   | Linux/Ubuntu 18.04 LTS Minimal |
| Web Server 1 | Web Server| 10.0.0.5   | Linux/Ubuntu 18.04 LTS Minimal |
| Web Server 2 | Web Server| 10.0.0.6   | Linux/Ubuntu 18.04 LTS Minimal |
| Web Server 3 | Web Server| 10.0.0.7   | Linux/Ubuntu 18.04 LTS Minimal |
| ELK Server   | Moniroting| 10.2.0.4   | Linux/Ubuntu 18.04 LTS Minimal |

In addition to the above, Azure has provisioned a **load balancer** in front of all machines except for the Jumpbox. The load balancer's targets are organized into the following availability zones:
- **Availability Zone 1**: Web Server 1 + Web Server 2 + Web Server 3
- **Availability Zone 2**: ELK Server


## ELK Server Configuration

The ELK VM exposes an Elastic Stack instance. **Docker** is used to download and manage an ELK container.

Rather than configure ELK manually, we opted to develop a reusable Ansible Playbook to accomplish the task. This playbook is duplicated below.

To use this playbook, one must log into the Jump Box, then issue: `ansible-playbook install_elk.yml elk`. This runs the `install_elk.yml` playbook on the `elk` host.

### Access Policies
The machines on the internal network are _not_ exposed to the public Internet. Only the three DVWA containers (housed inside the DVWA servers) and the ELK-STACK container (housed inside the Elk-Server) are exposed.

Only the **Jumpbox** machine can accept connections from the Internet. Access to this machine is only allowed from predetermined IP addresses belonging to the system administrator team and from computers that have the pgp Key used to create the Jumpbox.

Machines _within_ the peered networks can only be accessed by **each other**. The DVWA Web Server 1, DVWA Web Server 2, and the DVWA Web Server 3 send traffic to the ELK server.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses                       | 
|----------|---------------------|--------------------------------------------|
| Jump Box | Yes                 | XXX.XXX.32.82 (sysadmin's home IP)         |
| ELK      | No                  | 10.0.0.1-254 & 10.2.0.1-254                |
| DVWA 1   | No                  | 10.0.0.1-254 & 10.2.0.1-254                | 
| DVWA 2   | No                  | 10.0.0.1-254 & 10.2.0.1-254                | 
| DVWA 3   | No                  | 10.0.0.1-254 & 10.2.0.1-254                | 

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because ansible allows us to automate and quickly redeploy all dockers in our network. 

The playbook implements the following tasks:
- Installs needed files
- Configures the server and programs
- launches the docker container that has the configured Elk Stack (ex: Filebeat, Metricbeat)

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![docker ps](https://raw.githubusercontent.com/solidcode747/Cloud-Network/main/Images/docker_ps_output.png?token=AR7YBNCKKDFRSHP4YQSDI5LANZCMY)

The elk-server playbook is shown below.
```yaml
---
# install_elk.yml
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: sysadmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```
### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- DVWA Web Server 1 at `10.0.0.5`
- DVWA Web Server 2 at `10.0.0.6`
- DVWA Web Server 3 at `10.0.0.7`

These Beats allow us to collect the following information from each machine:
- **Filebeat**: Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.
- **Metricbeat**: Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login attempts, failed `sudo` escalations, and CPU/RAM statistics.

The playbook below installs filebeat on the target hosts:
```yaml
---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

    # Use command module
  - name: Setup filebeat
    command: filebeat setup

    # Use command module
  - name: Start filebeat service
    command: service filebeat start
        # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
```


The playbook below installs metricbeat on the target hosts:
```yaml
  ---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
```

To recap:

The playbook for elk-server is the install_elk.yml

The playbook for filebeat is the filebeat-playbook.yml

The playbook for metricbeat is the metricbeat-playbook.yml


## Using the Playbook

In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the filebeat-config.yml file to the ansible container into the correct location as indicated inside the file above.

- Use a similar process for the elk-server and when installing metricbeat.

- Update the hosts file in the ansible container in order to run the playbook on a specific VM.

- To find the hosts file and the Playbook files from inside the ansible container, run:

    `root@1c9a7e246652:~# cd /etc/ansible`

- Once inside the ansible directory, run `ls -la` and you’ll see all the necessary files as in this example: 

```
root@1c9a7e246652:/etc/ansible# ls -la
total 48
-rw-r--r-- 1 root root 19988 Mar 16 02:45 ansible.cfg      
-rw-r--r-- 1 root root  1232 Mar 20 20:24 hosts         <---
-rw-r--r-- 1 root root     0 Mar 20 20:31 install-elk.yml    <---
-rw-r--r-- 1 root root   683 Mar 16 03:46 myplaybook.yml    <---
drwxr-xr-x 2 root root  4096 Dec  4  2019 roles
```

- Update the host file to include the correct address for the ELK server and webservers as shown below:

```bash
    hosts
    [webservers]
    ## beta.example.org
    ## 192.168.1.100
    10.0.0.5 ansible_python_interpreter=/usr/bin/python3
    10.0.0.6 ansible_python_interpreter=/usr/bin/python3
    10.0.0.7 ansible_python_interpreter=/usr/bin/python3
    
    [elk]
    10.2.0.4 ansible_python_interpreter=/usr/bin/python3
```

- Run the playbook with this command from directory that contains the specific playbook you are running.  for example, run the install_elk.yml playbook from the ansible directory as shown below:
`root@1c9a7e246652:/etc/ansible# ansible-playbook install_elk.yml elk`

- Use these commands to run the playbooks:

    `$ ansible-playbook install_elk.yml elk` (when installing the elk container)

    `$ ansible-playbook filebeat-playbook.yml webservers`  (when installing the filebeat container)

    `$ ansible-playbook metricbeat-playbook.yml webservers` (when installing the metricbeat container)


- Navigate to the specific Docker host you've configured via SSH to check that the installation worked as expected.  Once inside the host VM, You can then run this:

    `$ sudo docker ps` 
    
    This is also shown above along with a sample it’s the output.

- To check our Elk-server with Kibana, we run the following from the elk-server:
    
    `sysadmin@ELK-SERVER:~$ curl localhost:5601/app/kibana`

- Alternatively, We can verify that we can load the elk stack server from our web browser at the kibana site by running this:  

    `http://[your.ELK-VM.External.IP]:5601/app/kibana`  
    [Example:  `http://40.78.125.144:5601/app/kibana`]

