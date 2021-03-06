# The SWITCH Monitoring System

Instructions for the implementation of the SWITCH Monitoring System:

## Step 1- Initiating the Monitoring Server on a machine such as “194.249.1.175”.

```
docker run -e MONITORING_SERVER="194.249.1.175" -p 8080:8080 -p 4242:4242 -p 4245:4245 -p 7199:7199 -p 7000:7000 -p 7001:7001 -p 9160:9160 -p 9042:9042 -p 8012:8012 -p 61621:61621 salmant/ul_monitoring_server_container_image:latest
```

It takes almost one minute to run the Monitoring Server. The Monitoring Server should be running on a machine with enough memory, disk and CPU resources. This machine should address the Cassandra hardware requirements explained in the following page:

https://wiki.apache.org/cassandra/CassandraHardware

Note 1: The environment variable named `MONITORING_SERVER` should be the IP address of the machine where the Monitoring Server is running.

Note 2: The Dockerfile to make the Monitoring Server container image is as follows:

https://github.com/salmant/ASAP/blob/master/SWITCH-Monitoring-System/Dockerfile.centos

Note 3: The Monitoring Server container image is publically released on Docker Hub: 

https://hub.docker.com/r/salmant/ul_monitoring_server_container_image/

## Step 2- Initiating the Monitoring Adapter on a machine such as “194.249.1.46”.

```
docker run -e MONITORING_SERVER="194.249.1.175" -e MONITORING_PREFIX="eu.switch.beia" -p 4242:4242 -p 4245:4245 -p 8125:8125/udp beia/monitoring_adapter
```

## Step 3- Implementing the Monitoring Agent based on StatsD.
Each container running in the service cluster includes two parts: “application instance” and “Monitoring Agent”. To make the Monitoring Agent, the StatsD protocol is used. Therefore, Monitoring Agents in the SWITCH project have been implemented through StatsD protocol available for many programming languages such as C/C++ and Python. There are too many examples on the Web to make a StatsD client: 

https://github.com/etsy/statsd/wiki#client-implementations

For instance, the following project is a Java-based StatsD client which can be downloaded: 

https://github.com/tim-group/java-statsd-client

The following code is a very simple example for the implementation of a Monitoring Agent:

https://github.com/salmant/ASAP/blob/master/SWITCH-Monitoring-System/MonitoringAgent.java

Note 1: Implementing a StatsD client is very simple. After a quick skim through the above example (MonitoringAgent.java), you would easily find out how to write your own code based on whatever programming language.

Note 2:  Different parameters have been defined in this code (MonitoringAgent.java). Therefore, you should define these parameters as needed.
* `MONITORING_PREFIX`
* Monitoring Adapter's IP address (`MONITORING_ADAPTER`)
* Container's IP address (`DockerHost_IP`) which is called agentip in the Monitoring System
* Monitoring interval (`MonitoringInterval_ms`)
* <metric_group_name>
* <metric_name>
* <units> which represents the scale of metric. 

Note 3: As shown in the above example (MonitoringAgent.java), UUID has been used for the unique identification of containers which is called agentid in the Monitoring System. In the SWITCH platform, agentid that represents the "CONTAINER_ID" can be defined by Docker. 

Note 4:  The measured value, e.g. value1 or value2 presented in the example (MonitoringAgent.java), is a “long” number. Therefore, if the raw value is a “flout” number, you should multiply that number by 10^X in order to make it “Long”. 

## Step 4- Manipulating data stored in the Cassandra TSDB.
Then, you can check if the values sent by the StatsD client (the Monitoring Agent) are stored in the TSDB. To this end, on the machine where the Monitoring Server is running (e.g. “194.249.1.175”), the following commands should be executed step-by-step:

- To get the "CONTAINER_ID" of the containerized Monitoring Server:
`docker ps`

- To log in the running container:
`docker exec -it "CONTAINER_ID" bash`
such as: `docker exec -it ff3ab1400aa0 bash`

- To start the CQL interactive terminal for Apache Cassandra:
`cqlsh "Monitoring_Server_IP" -u catascopia_user -p catascopia_pass`
such as: `cqlsh 194.249.1.175 -u catascopia_user -p catascopia_pass`

- To see the measured values, you can execute the following comment: 
`select * from jcatascopiadb.metric_value_table LIMIT 50;`

- To see the registered containers (agents in the TSDB), you can execute the following comment: 
`select * from jcatascopiadb.agent_table LIMIT 50;`

- To see the monitoring metrics, you can execute the following comment:
`select * from jcatascopiadb.metric_table LIMIT 50;`

