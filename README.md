# clouderainstallation_ansible
This repository includes ansible-playbooks used for cloudera 5.14 installation at offline sites. 


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

* All target hosts which will create cloudera cluster should be installed with Centos 7.x OS with Infrastructure Manager    installation.

* Ansible Control Machine -ansible host- should be installed on your local PC/Laptop.

* You should configure your inventory file -hosts file under /etc/ansible directory-. Following is an example for 4 hosts cloudera cluster.

            ▪ cloudera-manager: Host where related cloudera-server packages will be installed
            ▪ dbserver: Host where config db -mysql- will be installed. (Mostly same host with cloudera-manager)
            ▪ cloudera-hosts: Hosts where cloudera-client packages will be installed.

* You should ssh each host once with defined ansible_user(which is root in our case) in your hosts file to create ssh key between ansible control machine and related ansible targets. You can also try ssh key-gen for each target.

* You should copy all playbooks to your local **/etc/ansible** directory.

* You should copy all **“templates”** directory content to your local **/etc/ansible/templates** directory.

        ◦ CDH.5.14/ - folder
        ◦ clouderainstall/ - folder
        ◦ createrepoRPMs/ -folder
        ◦ hosts.j2 -should be updated with relevant info-
        ◦ jdk-8u131-linux-x64.rpm
        ◦ KAFKA/ - folder
        ◦ my.cnf.j2 
        ◦ mysql5.7/ - folder
        ◦ ntp.conf
        ◦ redis-3.0.0-tar.gz
        ◦ redis.conf.j2
        ◦ redis_init.script.j2
        ◦ RPMs/ - folder 
        
* You should copy all **“files”** directory content under your local **/etc/ansible/files** directory. 
        ◦ mysql-connector-java-5.1.35-bin.jar
        
* You should copy **“install_cloudera.sh”** script on your local PC and give **execution** permission. 
        

### Execution of Application


```

```




## Built With

* Shell

* Ansible 2.7



## Authors

* **Erkan Cetiner** - *Initial work* - [ecetiner87](https://github.com/ecetiner87)


## License

This project is publicly available. You can download it and make any changes for your own purposes.

