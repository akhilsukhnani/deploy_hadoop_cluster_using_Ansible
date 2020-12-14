
<H1>Deploying complete Hadoop Cluster using Ansible</H1>
<img src='https://github.com/akhilsukhnani/deploy_hadoop_cluster_using_Ansible/blob/main/explaination-task11.png'>

# What is Ansible? 
<img src='https://avatars1.githubusercontent.com/u/1507452?s=200&v=4'>
Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy and maintain. Automate everything from code deployment to network configuration to cloud management, in a language that approaches plain English, using SSH, with no agents to install on remote systems. 
Ansible is the simplest way to automate apps and IT infrastructure. Application Deployment + Configuration Management + Continuous Delivery
Ansible is an open-source software provisioning, configuration management, and application-deployment tool enabling infrastructure as code. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows.

<H1>Major Components of Ansible:-</H1>

<H2><Ansible Playbook:-</H2>Playbooks are the files where Ansible code is written. ... Playbooks are one of the core features of Ansible and tell Ansible what to execute. They are like a to-do list for Ansible that contains a list of tasks. Playbooks contain the steps which the user wants to execute on a particular machine.

<H2>Modules:-</H2> Ansible modules are reusable, standalone scripts that can be used by the Ansible API, or by the ansible or ansible-playbook programs. They return information to ansible by printing a JSON string to stdout before exiting. They take arguments in one of several ways.</H2>

<H2>Inventory:-</H2>The Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. The file can be in one of many formats depending on your Ansible environment and plugins. ... If necessary, you can also create project-specific inventory files in alternate locations.

<H2>Variable and Variable File:-</H2>Ansible uses variables to manage differences between systems. With Ansible, you can execute tasks and playbooks on multiple different systems with a single command.You can also create variables during a playbook run by registering the return value or values of a task as a new variable.

# What is Hadoop ?
Apache Hadoop is a collection of open-source software utilities that facilitates using a network of many computers to solve problems involving massive amounts of data and computation. It provides a software framework for distributed storage and processing of big data using the MapReduce programming model.
<H2>Hadoop Namenode:-</H2><p>The NameNode is the centerpiece of an HDFS file system. It keeps the directory tree of all files in the file system, and tracks where across the cluster the file data is kept. It does not store the data of these files itself. ... The NameNode is a Single Point of Failure for the HDFS Cluster.</p>
<H2>Hadoop DataNode:-</H2><p>DataNodes store data in a Hadoop cluster and is the name of the daemon that manages the data. File data is replicated on multiple DataNodes for reliability and so that localized computation can be executed near the data. </p>

# Need of Automation:-
A complete setup of HDFS Cluster is quite time consuming , and involves repetitive steps , to avoid that we used intelligence and power of ansible.
It involves these steps:-

# Steps for configuring a Hadoop Cluster

```At name node-
            
1. download the packages(jdk and hadoop)                                    
2. install the packages(jdk and hadoop)                                       
3. configure hdfs-site.xml file ->                                 
4. configure core-site.xml file ->                     
5. create a directory                              
6. format that directory                                    
7. start the namenode service                  
                                                     
At data-node                                               
                                            
1. download the packages(jdk and hadoop)
2. install jdk and hadoop
3. configure hdfs-site.xml file -> 
4. configure core-site.xml file ->
5. create a directory
7. start the datanode service
8. check the report of the cluster
```

<H1>Variables used in the playbook</H1>

```hadoop_report: "hadoop dfsadmin -report"
start_datanode: "hadoop-daemon.sh start datanode"
hdfs_loc: "/etc/hadoop/hdfs-site.xml"
hdfs_datanode: "/root/hadoop_ansible/hdfs1.xml"
coresite_location: "/etc/hadoop/core-site.xml"
coresite_file: "/root/hadoop_ansible/core-site.xml.j2"
hadoop_install: "rpm -ivh /root/hadoop-1.2.1-1.x86_64.rpm --force"
jdk_install: "rpm -ivh /root/jdk.rpm"
packages: "/root/hadoop_ansible/packages/*"
start_namenode: "hadoop-daemon.sh start namenode"
format_namenode: "yes Y | hadoop namenode -format"
hdfs_namenode: "/root/hadoop_ansible/hdfs.xml"
directory_namenode: "/name_node"
directory_datanode: "/data_node"
```


<H1>Playbook:-</H1>

```
- hosts: name_node
  vars_files:
          - variables.yml
  tasks:
   - name: Copying the necessary softwares
     copy:
        src: "{{ item }}"
        dest: /root/
     with_fileglob:
        - /root/hadoop_ansible/packages/* 
   - name: Creating directory for name_node to store metdata
     file:
        path: "{{ directory_namenode }}"
        state: directory
        mode: '0755'
   - name: Installing the java software
     shell: "{{ jdk_install }}"
     register: result
     ignore_errors: yes

   - name: Installing  hadoop software
     shell: "{{ hadoop_install  }}"
     register: result1
     ignore_errors: yes
   
   - name: Status check
     debug:
             var: 
               - result
               - result1

   - name: Copying the coresite file
     template:
        src: "{{ coresite_file }}"
        dest: "{{ coresite_location}}"

   - name: Copying hdfs-site.xml file
     template:
        src: "{{hdfs_namenode}}"
        dest: "{{ hdfs_loc }}"
   
   - name: disabling the firewalld
     service:
             name: firewalld
             state: stopped
             enabled: False   
   - name: Format the folder
     shell: "{{ format_namenode }}"
     register: format_status
     ignore_errors: yes

   - name: start the namenode
     shell: "{{ start_namenode }}"
     register: format_status
     ignore_errors: yes

   
- hosts: data_node
  vars_files:
          - variables.yml
  tasks:
   - name: Copying the necessary softwares
     copy:
        src: "{{ item }}"
        dest: /root/
     with_fileglob:
        - "{{ packages  }}"
   - name: creating directory for data_node to share the storage
     file:
        path: "{{directory_datanode}}"
        state: directory
        mode: '0755'

   - name: Installing  java
     shell: "{{ jdk_install }}"
     register: result2
     ignore_errors: yes

   - name: Installing hadoop software
     shell: "{{ hadoop_install }}"
     register: result3  
     ignore_errors: yes

   - name: testing
     debug:
             var : 
               - result2
               - result3

   - name: Copying the coresite file
     template:
        src: "{{coresite_file}}"
        dest: "{{ coresite_location }}"

   - name: Copying hdfs-site.xml
     template:
        src: "{{ hdfs_datanode }}"
        dest: "{{ hdfs_loc }}"

   - name: disabling the firewalld
     service:
             name: firewalld
             state: stopped
             enabled: False

   - name: starting data-node
     shell: "{{ start_datanode }}"
     register: datanode_status
     ignore_errors: yes
  
   - name: report of cluster
     shell: "{{ hadoop_report }}"
     register: datanode_status
     ignore_errors: yes
 
```
<H1>Files that would be copied after configuration(core-site.xml,hdfs-site.xml)</H1>

<H2>Core-site.xml</H2>

```
<configuration>
<property>
<name>fs.default.name</name>
{% for i in groups['name_node'] %}
<value>hdfs://{{ i  }}:9001</value>
{% endfor %}
</property>
</configuration>

```

<H2>hdfs-site.xml</H2>

```
<configuration>
<property>
<name>dfs.name.dir</name>
<value>/name_node</value>
</property>
</configuration>
```

<H2>Hosts-file(contains the credentials of the managed nodes):-</H2>
```
[data_node]
<ip_address>  ansible_user=root ansible_ssh_pass=<password> ansible_connection=ssh

[name_node]
<ip_address>  ansible_user=root ansible_ssh_pass=<password> ansible_connection=ssh
```


<H1>And the main command to run the playbook:-</H1>
```
ansible-playbook playbook.yml
```
<img src='https://github.com/akhilsukhnani/deploy_hadoop_cluster_using_Ansible/blob/main/task111.png'>
After that all the changes would be applied on the mentioned systems in the hosts.txt file

That's it our cluster has been configured, we can verfiy by following ways:-
  1.running ```jps``` and ```hadoop``` command
  <img src='https://github.com/akhilsukhnani/deploy_hadoop_cluster_using_Ansible/blob/main/verify_task11(3).png'>
  2.running ```hadoop dfsadmin -report```
  <img src='https://github.com/akhilsukhnani/deploy_hadoop_cluster_using_Ansible/blob/main/verify_task11(2).png'>
  3.going to webUI at  ```http://<ip_of_namenode>:9001``` 
  <img src='https://github.com/akhilsukhnani/deploy_hadoop_cluster_using_Ansible/blob/main/verify_task11(1).png'>
  
  Thanks.....!
  
  See this project running live at:-
  <LinkedIN>
