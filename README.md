# clouderainstallation_ansible
This repository includes **ansible-playbooks** used for **Cloudera 5.14** installation at offline sites. 


Cloudera Manager is an end-to-end application for managing CDH clusters. Cloudera Manager provides granular visibility into and control over every part of the CDH cluster—empowering operators to improve performance, enhance quality of service, increase compliance, and reduce administrative costs. With Cloudera Manager, you can easily deploy and centrally operate the complete CDH stack and other managed services. The application automates the installation process, reducing deployment time from weeks to minutes; gives you a cluster-wide, real-time view of hosts and services running; provides a single, central console to enact configuration changes across your cluster; and incorporates a full range of reporting and diagnostic tools to help you optimize performance and utilization. Cloudera Manager also provides an API you can use to automate cluster operations.

## Getting Started

**Ansible** is very useful for IT automation, system configuration and deployment purposes. To minimize human error throughout installation of CDH, following playbooks are implemented. Note that; this installation is used for **offline** sites where no internet connection takes place. Thus instead of resolving dependencies with **yum** ansible module; all related files/packages are determined and located before starting installation.

**Playbooks:**

        ◦ main.yml
        ◦ reboot.yml
        ◦ redisinstall.yml
        ◦ mysqlinstall.yml
        ◦ mysql_secure_installation.yml
        ◦ createdb.yml
        ◦ mysqlconnector.yml
        ◦ clouderaconfig.yml
        ◦ clouderaserverinstall.yml


### Prerequisites

* All target hosts which will create cloudera cluster should be installed with **Centos 7.x OS** with **Infrastructure Manager**    installation.

* Ansible Control Machine -ansible host- should be installed on your local PC/Laptop.

* You should configure your inventory file -hosts file under /etc/ansible directory-. Following is an example for 4 hosts cloudera cluster.

            ▪ cloudera-manager: Host where related cloudera-server packages will be installed
            ▪ dbserver: Host where config db -mysql- will be installed. (Mostly same host with cloudera-manager)
            ▪ cloudera-hosts: Hosts where cloudera-client packages will be installed.

* You should ssh each host once with defined ansible_user(which is root in our case) in your hosts file to create **ssh key** between ansible control machine and related ansible targets. You can also try **ssh key-gen** for each target.

* You should copy all playbooks to your local **/etc/ansible** directory.

* You should copy all **“templates”** directory content to your local **/etc/ansible/templates** directory.

        ◦ CDH.5.14/ - folder - please check directions in txt files 
        ◦ clouderainstall/ - folder
        ◦ createrepoRPMs/ -folder
        ◦ hosts.j2 -should be updated with relevant info-
        ◦ jdk-8u131-linux-x64.rpm -- you can download online
        ◦ KAFKA/ - folder- please check directions in txt files
        ◦ my.cnf.j2 
        ◦ mysql5.7/ - you can download all packages online and put related mysql folder into /etc/ansible/templates folder
        ◦ ntp.conf
        ◦ redis-3.0.0-tar.gz
        ◦ redis.conf.j2
        ◦ redis_init.script.j2
        ◦ RPMs/ - folder 
        
* You should copy all **“files”** directory content under your local **/etc/ansible/files** directory. 
       
        ◦ mysql-connector-java-5.1.35-bin.jar 
         
* You should copy **“install_cloudera.sh”** script on your local PC and give **execution** permission. 
        
**Important Note:** Since customers request offline installation -no available internet connection- it is not possible to use yum module with online registration in ansible playbooks. Thus, if you have hosts with Centos minimal installation or if you think there are missing packages; you should include them with their dependencies under **/etc/ansible/templates/RPMs** folder also. 

### Execution of Application

**Playbooks in Detail**

**Main.yml**
This playbook covers all necessary preconfiguration tasks before cloudera installation on related hosts. Following items are covered:

            • Update /etc/hosts 
            • Set Enforcing to 0
            • Disable SELINUX 
            • Stop Firewalld 
            • Disable Firewall 
            • Update /etc/sysctl.conf, /sys/kernel/mm/transparent_hugepage/defrag and /etc/local.rc files with cloudera configs 
            • Install NTP and all related RPMs 
            • Set Timezone 
            • Configure NTP on all clients 
            • Remove Existing Java Installation if exists and Install JDK1.8 and select it as default with alternatives. 
```
ansible-playbook main.yml --extra-vars "ntp_IP=192.168.4.234" (ntp_IP variable should be passed with desired NTP source)
```
**Effected Hosts:** cloudera-hosts

**reboot.yml**
Some actions taken in main.yml playbook require reboot to be effective. Thus; reboot.yml playbook take this action on each target host.
```
ansible-playbook reboot.yml
```
**Effected Hosts:** cloudera-hosts

**redisinstall.yml**
redisinstall.yml playbook covers following items:

    • Copy redis3.0 tarball to target hosts
    • Untar related tar.gz
    • Install redis3.0
    • Move compiled redis binaries under usr/local/bin
    • Configure redis-server
    • Create redis service
    • Run redis service
```
ansible-playbook redisinstall.yml
```
**Effected Hosts:** cloudera-hosts

**mysqlinstall.yml**
mysqlinstall.yml playbook covers following items:

    • Remove any previously installed mysql/mariadb/postfix packages
    • Copy all necessary rpms to target hosts
    • Install related mysql rpms
    • Start and enable mysql service
```
ansible-playbook mysqlinstall.yml 
```
**Effected Hosts:** dbserver

**mysq_secure_installation.yml**
With mysql5.7, it is required to run **mysqlsecureinstallation.sh** to configure mysql deamon and related root privileges. This playbook covers related items just like mysqlsecureinstallation.sh

    • Get temporary root password from /var/log/mysqld.log
    • Set the new mysql root password with given parameter -mysql_root_password-
    • Delete anonymous users from mysql
    • Remove test database
    • Restart mysql server deamon.
```
ansible-playbook mysql_secure_installation.yml –extra-vars "mysql_root_password=password" (mysql_root_password variable should be passed with desired password)
```
**Effected Hosts:** dbserver

**createdbfinal.yml**
Before installation of cloudera packages; some configuration DBs for cloudera components should be created with relevant user on dbserver -where mysql installed and configured-
**Note:** Please update **my.cnf.j2** file under templates directory before executing **install_cloudera.sh** script.

    • Give all privileges on all dbs to root user
    • Change password validation to easy way 
    • Disable validate plugin
    • Create ‘amon’ database and ‘amon’ user with related grant options. 
    • Create ‘rman’ database  and ‘rman’ user with related grant options. 
    • Create ‘metastore’ database  and ‘metastore’ user with related grant options. 
    • Create ‘sentry’ database  and ‘sentry’ user with related grant options. 
    • Create ‘nav’ database  and ‘nav’ user with related grant options. 
    • Create ‘navms’ database  and ‘navms’ user with related grant options. 
    • Create ‘oozie’ database  and ‘oozie’ user with related grant options. 
    • Create ‘hue’ database  and ‘hue’ user with related grant options. 
    • Create ‘scm’ database  and ‘scm’ user with related grant options. 
    • Flush all privileges for all users.
```
ansible-playbook createdb.yml --extra-vars "mysql_root_password=password" (mysql_root_password variable should be passed with desired password)
```
**Effected Hosts:** dbserver  
 
**mysqlconnector.yml**
Mysql-connector-java is needed plugin for cloudera installation and should be present on dbserver. This playbook includes following actions:

    • Check if mysql-connector-java.jar is present on dbserver
    • If not; copy related file to /usr/share/mysql-java directory
    • Move related file under /usr/share/java and rename it.
```
ansible-playbook mysqlconnector.yml
```   
**Effected Hosts:** dbserver 

**clouderaconfig.yml**
This playbook used for final configurations such as local repo creation, web server config and cloudera rpms distribution on cloudera hosts.

    • Copy all cloudera rpms to /home/clouderainstall folder of target hosts.
    • Copy createrepo rpms and its dependencies.
    • Install createrepo on target hosts
    • Add CM local repository with yum_repository module.
    • Create local repo for CM.
```
ansible-playbook clouderaconfig.yml
```   
**Effected Hosts:** cloudera-hosts

**clouderaserverinstall.yml**
Final playbook which used for cloudera manager installaton on cloudera-manager host.

    • Install CM packages by using local CM repo
    • Prepare Database with scm_prepare_database.sh via shell module.
    • Create related directories for CDH and KAFKA on local webserver.
    • Copy all necessary items to CDH.5.14 folder.
    • Copy all necessary items to KAFKA folder.
    • Create directories for datanodes.
    • Start apache and enable it via service module.
    • Start cloudera-scm-server via shell module.
```
ansible-playbook clouderaserverinstall.yml
```   
**Effected Hosts:** cloudera-manager

**Execution Demo:**

1) Run related script from the path where you copied it. Use tee command to see related ansible output and to log them in a          file.
```
$ ./install_cloudera.sh | tee “install_cloudera.log”
```
2) After script execution finished; check the related log file too see if you have any error. Some errors and warnings have “ignoring...” tag; you can skip them.

3) If you haven’t any error in log file (other than “ignoring” ones) you should login cloudera manager from GUI successfully with URL https://cloudera-managerIP:7180

4) Login with admin/admin user/pass configuration

5) Continue with express-wizard and verify that Cloudera Manager find all nodes that you supply in your /etc/hosts file.

6) Enter local repo URLs to Cloudera Manager as Repository to point your own parcel files that you created via ansible.
        ◦ For CDH: IP/CDH.5.14/CDH.5.14
        ◦ For KAFKA: IP/KAFKA/KAFKA
        
7) Install Agents and verify all nodes installed successfully.

8) Setup Databases for hive/oozie/hue as following:
        ◦ Database Type: MySQL; 
        ◦ Database Name/User: hue/oozie/hue     hue/oozie/hue 
        
9) Finalize Cluster Setup  

10) You can control your cluster dashboard now.


## Built With

* Shell

* Ansible 2.7



## Authors

* **Erkan Cetiner** - *Initial work* - [ecetiner87](https://github.com/ecetiner87)


## License

This project is publicly available. You can download it and make any changes for your own purposes.

