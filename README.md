### HEALTH CARE DEMO GOAL: Actively monitor ICU patients for risk indicators of Sepsis

#### Motivation

In 2011, the US spent $20.3 billion dollars on hospital care for patients with sepsis.

Sepsis is a potentially life-threatening complication of an infection. Sepsis occurs when chemicals released into the bloodstream to fight the infection trigger inflammatory responses throughout the body. This inflammation can trigger a cascade of changes that can damage multiple organ systems, causing them to fail.  If sepsis progresses to septic shock, blood pressure drops dramatically, which may lead to death 

#### Real Time Patient Monitoring for risk indicators of Sepsis  with Hortonworks Data Flow

###COMPONENTS USED:
* NIFI
* STORM
* SOLR
* KAFKA
* HBASE

## SETUP

##### requires maven
```
sh install_mvn.sh
exit
```
##### log back in and check maven works
```
mvn -v
```

####setup kafka
```
cd /usr/hdp/current/kafka-broker/bin/
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic healthcaredemo 
```
####cleanup kafka q. First enable property in ambari->kafka->configs : delete.topic.enable=true
```
cd /usr/hdp/current/kafka-broker/bin/
./kafka-topics.sh --zookeeper localhost:2181 --delete --topic healthcaredemo
```
####check the kafka q
```
/usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic healthcaredemo --from-beginning
```

####storm
Make sure Storm is started.  
From the /HealthCareDemoStorm directory...
```
mvn clean install
mkdir -p /opt/HealthCareDemo/
cp {GIT_DIRECTORY}/HealthCareDemoStorm/src/main/resources/healthcare_event_topology.properties /opt/HealthCareDemo/
storm jar target/HealthCareDemo-1.0-SNAPSHOT.jar com.hortonworks.se.healthcare.storm.PatientTopology /opt/HealthCareDemo/healthcare_event_topology.properties
```
####Kill Healthcare Storm Topology
```
storm kill healthcare-event-processor
```

####HBASE
```
[root@sandbox ~]$ su hbase
[hbase@sandbox root]$ hbase shell
hbase(main):001:0> create 'patient_events', 'events'    
hbase(main):002:0> create 'patient_danger_events', 'count'    
hbase(main):003:0> list    
hbase(main):004:0> exit 
```
####producer simulate
```
[root@sandbox HealthCareDemoStorm]# java -cp target/HealthCareDemo-1.0-SNAPSHOT.jar com.hortonworks.se.healthcare.producer.PatientDataProducer sandbox.hortonworks.com:6667 sandbox.hortonworks.com:2181
```
##### hbase bolt requires core-site.xml, hdfs-site.xml and hbase-site.xml on storm's classpath or in src/main/resources 

#### solr
```
wget http://apache.mirror1.spango.com/lucene/solr/5.2.1/solr-5.2.1.tg

root@sandbox ~]# sh /root/solr-5.2.1/bin/solr start

Started Solr server on port 8983 (pid=21554). Happy searching!

[root@sandbox ~]# sh /root/solr-5.2.1/bin/solr create -c healthcare -d /root/solr-5.2.1/server/solr/configsets/data_driven_schema_configs/conf -n healthcare

Setup new core instance directory:
/root/solr-5.2.1/server/solr/healthcare

Creating new core 'healthcare' using command:
http://localhost:8983/solr/admin/cores?action=CREATE&name=healthcare&instanceDir=healthcare
```
####BANANA
Install Banana 1.6 and add the Sepsis Monitoring Dashboard as the default dashboard

```
unzip banana-release.zip
mv banana-release banana
cp -R banana /root/solr-5.2.1/server/solr-webapp/
mv /root/solr-5.2.1/server/solr-webapp/webapp/banana/src/app/dashboards/default.json /root/solr-5.2.1/server/solr-webapp/webapp/banana/src/app/dashboards/default_bkp.json
cp {GIT_DIRECTORY}/ui/Active_Sepsis_Monitoring.json /root/solr-5.2.1/server/solr-webapp/webapp/banana/src/app/dashboards/default.json
```

#### TO DELETE SOLR INDEX
```
curl "http://localhost:8983/solr/admin/cores?action=UNLOAD&core=healthcare&deleteIndex=true"
```
  
