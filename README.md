EMC XtremIO Flocker Vagrant Enviroment
======================
This project Vagrant environment for trying EMC XtremIO Flocker intergration.

## Description
ClusterHQ/Flocker provides an efficient and easy way to connect persistent store with Docker containers. This project provides a sample vagrant environment for trying out solution.

## EMC XtremIO Flocker Intergration Block Diagram
![EMC XtremIO Flocker Intergration Block Diagram Missing] 
(https://github.com/emccorp/vagrant-xtremio-flocker/blob/master/EMCXtremIOFlocker.jpg)



## Installation
Tested with Vagrant 1.7.2
- Clone source code from git repository
 * git clone https://github.com/emccorp/vagrant-xtremio-flocker.git
- Change directory
 * cd vagrant-xtremio-flocker
- Bring up vagrant machines
 * vagrant up<br>
   This shall create two ubuntu trusty64 host and install all needed iSCSI software on the host

- Check the status of nodes<br>
    vagrant status (it should print following)<br>
        Current machine states:<br>
        node                      running (virtualbox)<br>
        node2                     running (virtualbox)<br>
- Test login to the host<br>
 vagrant ssh node1
 * vagrant ssh node2<br>
The node1 gets a preassigned ip address node1: 192.168.33.10 and node2: 192.168.33.11<br>

- Discover iSCSI XtremIO portal on the host
 * vagrant ssh node1
 * /vagrant/Config/iSCSIDiscover <EMC XtremIO iSCSI Portal IP>
 * /vagrant/Config/iSCSILogin <EMC XtremIO iSCSI Portal IP>
 * lsssci (this should print XtremIO as one of the storage arrays)
 * exit
 * vagrant ssh node2
 * /vagrant/Config/iSCSIDiscover <EMC XtremIO iSCSI Portal IP>
 * /vagrant/Config/iSCSILogin <EMC XtremIO iSCSI Portal IP>
 * lsssci (this should print XtremIO as one of the storage arrays)

- Install ClusterHQ/Flocker<br>
TBD

- Install EMC Plugin for XtremIO<br>
TBD

## Usage Instructions
Please refer to ClusterHQ/Flocker documentation for usage. A sample deployment and application file for Cassandra server is present with this code.<br>
- Deploying Cassandra Database on node1:
* vagrant ssh node1
* flocker-deploy 192.168.33.10 /vagrant/cassandra-deployment.yml /vagrant/cassandra-application.yml<br>
    The default deployment node on /vagrant/cassandra-deployment.yml is 192.168.33.10.
* sudo docker ps (you should now see cassandra docker deployed)
* sudo docker inspect flocker--cassandra (this shall show the volume connected, mounted as file-system on the host)

- Check status of the Cassandra node
* sudo docker exec -it flocker--cassandra nodetool status (you should get output as below)
* sudo docker exec -it flocker--cassandra nodetool status<br>
	Status=Up/Down |/ State=Normal/Leaving/Joining/Moving<br>
	--  Address       Load       Tokens  Owns    Host ID                               Rack<br>
	UN  172.17.0.162  130.26 KB  256     ?       ef92d409-ee9f-4773-9ca7-bbb5df662b77  rack1<br>
- Create sample keyspace in Cassandra database:
 * sudo docker exec -it flocker--cassandra cqlsh
 * The above shall give you a cqlsh prompt
 * Copy paste following to create database and table<br>
 CREATE KEYSPACE EMCXtremIO WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 0};
 CREATE TABLE EMCXtremIO.users (userid text PRIMARY KEY, first_name text);
- Check the schema created
  * desc keyspace EMCXtremIO
  
- Migrate Cassandra database to node2<br>
  ClusterHQ flocker provides a way to migrate data from one node to another<br>
  * Modify cassandra-deploy.yml file present in the root folder to specify target host at 192.168.33.11.
  * vagrant ssh node1
  * flocker-deploy /vagrant/cassandra-deploy.yml /vagrant/cassandra-application.yml
  * Check the existence of database EMCXtremIO
     ** sudo docker exec -it flocker--cassandra cqlsh
     ** desc keyspace EMCXtremIO

- Protecting Cassandra Node with Docker<br>
  EMC XtremIO comes Snapshotting capabilities which can be extended to Docker Cassandra <br>
  installation for supporting application consistent snapshots <br>
  * sudo docker exec -it flocker-cassandra nodetool snapshot
  * sudo docker inspect | grep -i data (you should a data folder mapped to location mount point e.g.<br>
      /flocker/121c60df-0c03-083d-2693-c251f15fdfb2/
  * ls -l /flocker/121c60df-0c03-083d-2693-c251f15fdfb2/data/emcxtremio/users-bc224f500abd11e58c4e4f5a89e1ffdd/snapshots/<br>
    to get cassandra snapshot
  * XtremIO snapshot: This can be performed using their management GUI or curl CLI interface.<br>
   If performed from management GUI look for volume name block-121c60df-0c03-083d-2693-c251f15fdfb2, right click and <br>    
    snapshot. While taking snapshot move it to a new folder VOL_FOLDER_SNAPSHOT
  * Delete local cassandra snapshot<br>
   The local cassandra snapshot can be deleted since we have an array preserved snapshot
   ** sudo docker exec -it flocker-cassandra nodetool clearsnapshot
   ** ls -l /flocker/121c60df-0c03-083d-2693-c251f15fdfb2/data/emcxtremio/users-bc224f500abd11e58c4e4f5a89e1ffdd/
   should show 0 files.

- To automate snapshot management platform kindly try using tool<br>
https://github.com/evanbattle/XtremIOSnap

  
## Future
- Add Chap protocol support for iSCSI
- Add 

## Contribution
Create a fork of the project into your own reposity. Make all your necessary changes and create a pull request with a description on what was added or removed and details explaining the changes in lines of code. If approved, project owners will merge it.

Support
-------

