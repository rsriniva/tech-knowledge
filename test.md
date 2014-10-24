# Accenture Olympus Project 

## 1. Local JBoss Fuse Installation without Fabric

* Deploy the features XML repository file

    ````
    features:addurl mvn:com.redhat.gpe.poc.accenture.olympus/features/1.0.0/xml/features
    ````

* Install the outbound module / features
    ````
    features:install outbound
    ````
* Install the inbound module / features
    ````
    features:install inbound
    ````
* Open the FMC console and verify that the routes are well deployed

See doc/olympus-routes.png

## 2. Local JBoss Fuse Fabric Installation

* Create the Fabric Ensemble Server
    ````
    fabric:create --zookeeper-password admin--wait-for-provisioning
    ````
* Run the following command under the project module (= root of the project) in your terminal
    ````
    mvn fabric8:deploy
    ```` 
NOTE: If the process fails (org.apache.http.NoHttpResponseException: localhost:8181 failed to respond), check the host name of the jolokia registered in JMX abd use this url with the option
    ````
    mvn fabric8:deploy -DskipTests=true -Dfabric8.jolokiaUrl=http://192.168.1.80:8181/jolokia
    ````   
* The profile accenture -> poc -> outbound should be created under the wiki
    ````
    http://localhost:8181/hawtio/index.html#/wiki/branch/1.0/view/fabric/profiles/accenture/poc/outbound.profile
    ````
* Like also the inbound  accenture -> poc -> inbound
    ````
    http://localhost:8181/hawtio/index.html#/wiki/branch/1.0/view/fabric/profiles/accenture/poc/outbound.profile
     ```` 
* Deploy outbound profile in a new container
* From the JBoss Fuse Console, run this command to create a container and provision it with our profile
````
    JBossFuse:karaf@root> fabric:container-create-child --profile accenture-poc-outbound --profile accenture-poc-datasource root wayne-outbound
    JBossFuse:karaf@root> fabric:container-create-child --profile accenture-poc-inbound --profile accenture-poc-datasource root inbound
```` 
REMARKS : 
- Refer to [fabric8 maven plugin documentation](http://fabric8.io/gitbook/mavenPlugin.html) for any issue
- To delete a container : fabric:container-delete inbound

## Steps for Connecting to AWS VMs

* Open a terminal within the git cloned repository

* Move to `fusepockeys` directory    

* Connect to NAT Instance (= 54.164.100.187)
    ````
    ssh -i fusepoc.pem ec2-user@54.164.100.187
    ````
* From the NAT Instance box, you can now connect to the private Fuse Public machine
    ````    
    ssh -i fuseservers.pem 10.0.0.36
    ````    
and next to one of the Fuse server 1,2 or 3 machines
    ````  
    ssh -i fuseservers.pem ec2-user@10.0.1.216
    ssh -i fuseservers.pem ec2-user@10.0.1.24
    ssh -i fuseservers.pem ec2-user@10.0.2.242
    ````    
#
# Copy JBoss Fuse & SP2 Files to NAT machine
#
  
* JBoss Fuse 6.1 - 379
    ````    
    scp -i fusepoc.pem jboss-fuse-full-6.1.0.redhat-379.zip ec2-user@54.164.100.187:jboss-fuse-full-6.1.0.redhat-379.zip
    ````    
* SP2     
    ````    
    scp -i fusepoc.pem jboss-fuse-6.1.0.redhat-379-p2.zip ec2-user@54.164.100.187:jboss-fuse-6.1.0.redhat-379-p2.zip
    ````    
# Copy files from NAT to Fuse Public Machine
    ````    
    scp -i fuseservers.pem jboss-fuse-full-6.1.0.redhat-379.zip 10.0.0.36:jboss-fuse-full-6.1.0.redhat-379.zip    
    scp -i fuseservers.pem jboss-fuse-6.1.0.redhat-379-p2.zip 10.0.0.36:jboss-fuse-6.1.0.redhat-379-p2.zip
    ````
#
# Missing things on Fuse Server machines
#
    
* Git, wget are not installed
    ````    
    yum install git
    yum install wget
    yum install unzip
    yum install zip
    yum install telnet
    ````    
* Install and configure maven
    
    http://stackoverflow.com/questions/17786422/installing-maven-3-0-5-in-redhat-linux
    ````    
    wget http://mirrors.gigenet.com/apache/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
    su -c "tar -zxvf apache-maven-3.0.5-bin.tar.gz -C /opt/" 
    
    su -c "vi /etc/profile.d/maven.sh"
    
    # Add the following lines to maven.sh
    export M2_HOME=/opt/apache-maven-3.0.5
    export M2=$M2_HOME/bin
    PATH=$M2:$PATH 
    ````    
* Install JDK from Oracle as jre was installed   
    ````  
    wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u71-b14/jdk-7u71-linux-x64.rpm
    sudo rpm -i jdk-7u71-linux-x64.rpm
    sudo /usr/sbin/alternatives --install /usr/bin/java java /usr/java/jdk1.7.0_71/bin/java 20000
    sudo /usr/sbin/alternatives --config java
    ````    
* Change bash_profile file to add JAVA_HOME & MAVEN_HOME
    ````   
   # JAVA
   export JAVA_HOME=/usr/java/jdk1.7.0_71
   export PATH=$JAVA_HOME/bin:$PATH
   
   # MAVEN
   export M2_HOME=/opt/apache-maven-3.0.5
   export PATH=$M2:$PATH
   export MAVEN_OPTS="-Xmx2048m -XX:PermSize=256m -XX:MaxPermSize=512m"
    ````   
#
# Git clone project on Fuse Public Machine
#
   
* Open a terminal and connect to NAT and Fuse Public Machine
    
* Clone Accenture Poc project from BitBucket
    ````    
    git clone https://YOUR_ID@bitbucket.org/cmoulliard/accenture-olympus-poc.git
    
    where YOUR_ID is tour bitbucket account
    ````    
# 
# Launch on Fuse Public machine -> JBoss Fuse
#
    ````
    cd /home/ec2-user/jboss-fuse-6.1.0.redhat-379/bin
    ./admin start root
    ````    
and next call the console
    ````
    ./client 
    ````      
Within the console, create Fabric
    ````
    fabric:create --zookeeper-password admin
    ````      
Leave the client (ctrl-d)

Edit the database.properties file localted under `datasource/src/main/fabric8/database.properties` to uncomment the properties needed to use the DB connection of the DB installed
on Amazon
    ````
    jdbc.driverClassName=org.postgresql.Driver
    #jdbc.url=jdbc:postgresql://localhost/olympus
    #jdbc.username=demo
    #jdbc.password=demo
    jdbc.url=jdbc:postgresql://fusedb.c0zv75ksxeuz.us-east-1.rds.amazonaws.com:5432/poc
    jdbc.username=fusepoc
    jdbc.password=fusepoc01
    ````
Move to the root of the project cloned from bitbucket, deploy the profiles into Fabric
    ````
    mvn fabric8:deploy  -DskipTests=true -Dfabric8.jolokiaUrl=http://ip-10.0.0.36:8181/jolokia
    ````
Return to the JBoss Fuse Console to create the 2 containers using `./client` script under bin directory of JBoss Fuse
    ````
    fabric:container-create-ssh --host 10.0.1.216 --private-key /home/ec2-user/fuseservers.pem --user ec2-user inbound
    fabric:container-create-ssh --host 10.0.1.24 --private-key /home/ec2-user/fuseservers.pem --user ec2-user outbound
     ````  
Remark:
   The deployment of the `inbound` profile into the container `inbound` does not succeed the first time. Stop and restart the container, that should work. Apparently CXF is not able to find the HTTP Transport service.
    
    
    
