#### Make Ambari REST calls from Ambari view
Demo view to manage Ambari via REST calls using some [sample operations](https://github.com/abajwa-hw/blueprints-view#run-the-sample-operations):
- Export blueprint
- Get status of components
- Stop/Start services
- [Add your own operations](https://github.com/abajwa-hw/blueprints-view#add-your-own-operations)

	 
		
##### Setup

- Download HDP 2.2 sandbox VM image (Sandbox_HDP_2.2_VMware.ova) from [Hortonworks website](http://hortonworks.com/products/hortonworks-sandbox/)
- Import Sandbox_HDP_2.2_VMware.ova into VMWare and set the VM memory size to 8GB
- Now start the VM
- After it boots up, find the IP address of the VM and add an entry into your machines hosts file e.g.
```
192.168.191.241 sandbox.hortonworks.com sandbox    
```
- Connect to the VM via SSH (password hadoop) and start Ambari server
```
ssh root@sandbox.hortonworks.com
/root/start_ambari.sh
```

- Install Maven
```
mkdir /usr/share/maven
cd /usr/share/maven
wget http://mirrors.koehn.com/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz
tar xvzf apache-maven-3.2.5-bin.tar.gz
ln -s /usr/share/maven/apache-maven-3.2.5/ /usr/share/maven/latest
echo 'M2_HOME=/usr/share/maven/latest' >> ~/.bashrc
echo 'M2=$M2_HOME/bin' >> ~/.bashrc
echo 'PATH=$PATH:$M2' >> ~/.bashrc
export M2_HOME=/usr/share/maven/latest
export M2=$M2_HOME/bin
export PATH=$PATH:$M2
```

- To deploy the iFrame view, run below. On non-sandbox env, these steps should be run on node running Ambari server
```
#Pull code (pom.xml, view.xml, index.html)
cd
git clone https://github.com/abajwa-hw/blueprints-view.git
cd blueprints-view

#Tell maven to compile against ambari jar (double check that the jar exists in this location, first)
mvn install:install-file -Dfile=/usr/lib/ambari-server/ambari-views-1.7.0.169.jar -DgroupId=org.apache.ambari -DartifactId=ambari-views -Dversion=1.3.0-SNAPSHOT -Dpackaging=jar

#Compile view
mvn clean package

#move jar to Ambari dir
cp target/*.jar /var/lib/ambari-server/resources/views
   
```
- Restart Ambari
```
#on HDP 2.2 sandbox
service ambari restart

#on non-sandbox
service ambari-server restart
```

- Now open Ambari and navigate to the Views and select 'Blueprint View'
http://sandbox.hortonworks.com:8080
![Image](../master/screenshots/blueprint-view.png?raw=true)

---------------------

##### Run the sample operations

- First enter your cluster parameters:
  - host/port (e.g. sandbox.hortonworks.com:8080) 
  - cluster name (e.g. Sandbox)
  - Username/pass (e.g admin/admin)

- Export cluster blueprint:
  - Select "Full Cluster Blueprint" from the dropdown and click Submit. Notice the entire definition of the cluster is returned
  - ![Image](../master/screenshots/export-BP.png?raw=true)
    
- Stop HBase (assuming its already started):
  - Select "Stop HBase" from the dropdown and click Submit. Notice "1 ops" started running
  - ![Image](../master/screenshots/stop-Hbase.png?raw=true)

- After a few seconds, check HBase status:
  - Select "HBase status" from the dropdown and click Submit. Notice state is now "INSTALLED" which means is not running
  - ![Image](../master/screenshots/status-Hbase.png?raw=true)
  
- Start HBase (assuming its stopped):
  - Select "Start HBase" from the dropdown and click Submit. Notice "1 ops" started running
  - ![Image](../master/screenshots/start-Hbase.png?raw=true)

- Check HBase status:
  - Select "HBase status" from the dropdown and click Submit. Notice that state is now "STARTED"
  - ![Image](../master/screenshots/status-Hbase-started.png?raw=true)

- Notice that the status check APIs did not require a request body but the stop/start ones did


---------------------

##### Add your own operations

- You can add your own operations by modifying the JSON in [/var/lib/ambari-server/resources/views/work/BLUEPRINT_VIEW{1.0.0}/index.html](https://github.com/abajwa-hw/blueprints-view/blob/master/src/main/resources/index.html#L7) containing the sample operations, and refreshing the view. This will add your HTTP request to the dropdown list

- You can also update the default values of [hostname](https://github.com/abajwa-hw/blueprints-view/blob/master/src/main/resources/index.html#L138) and [cluster name](https://github.com/abajwa-hw/blueprints-view/blob/master/src/main/resources/index.html#L143) in the same file

- Note the values of $host and $cluster will get replaced at runtime. Also the single quotes in the JSON body will be replaced by double quotes in the view
