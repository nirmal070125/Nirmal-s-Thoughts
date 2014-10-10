Stratos 4.1.0 - M1 - Developer Preview - Getting Started Guide
=========

Table of Content
----------

- [Main Features](#main-features)
- [Pre-requisite](#pre-requisite)
- [Testing M1](#testing-m1)
- [Jira List](#jira-list)
- [Troubleshooting Guide](#troubleshooting-guide)


Main Features
-------------

- [Docker](https://www.docker.com/) support using [Google Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes) and [CoreOS](https://coreos.com/)
- [MQTT](http://mqtt.org/) support (removal of JNDI)

Pre-requisite
-------------

- Setting up the Kubernetes-CoreOS cluster in your local machine
    * Install [Vagrant 1.6.5](https://www.vagrantup.com/) and [Oracle VM VirtualBox Manager 4.3.14](https://www.virtualbox.org/) in your local machine.
    * Get a GIT clone of [Vagrant Kubernetes Setup](https://github.com/nirmal070125/vagrant-kubernetes-setup) onto your machine.
    * Navigate to the cloned repository directory (**SETUP_HOME**).
    * Run ``` up.sh ``` script file - ``` {SETUP_HOME}$ ./up.sh ``` (You might have to use ```sudo``` in some cases.)
    * Above command will start-up 4 VMs, namely discovery, master, minion-1 and minion-2.
    * SSH to master node ; ``` {SETUP_HOME}$ vagrant ssh master ```
    * Pull Stratos PHP Docker Image from DockerHub into master node or into the local machine.
    ``` sh 
    docker pull apachestratos/php-4.1.0-m1
    ```
    * Import downloaded Stratos PHP Docker image as a tarball.
    ```sh
    docker save -o stratos-php-latest.tar  apachestratos/php-4.1.0-m1
    ```     
    * SCP the Stratos PHP Docker Image tarball to minion-1 and minion-2. You can find the private key file which you can use to SCP, in the **{SETUP_HOME}/ssh.config** file, against **IdentityFile** attribute. 
    ``` sh
    scp -i ~/.vagrant.d/insecure_private_key stratos-php-latest.tar core@172.17.8.101:.
    ```
    * SSH to minion-1 (``` {SETUP_HOME}$ vagrant ssh minion-1 ```) and minion-2 (``` {SETUP_HOME}$ vagrant ssh minion-2```)
    * Load Stratos PHP Docker Image from tarball to minion-1 and minion-2
    ```sh
    docker load -i stratos-php-latest.tar
    ```   
    * Verify that the image is properly loaded by issuing ```docker images``` in each node;
    ```sh
    core@master ~ $ docker images
    REPOSITORY                   TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    apachestratos/php-4.1.0-m1   latest              0fff8e5ac572        3 hours ago         771.7 MB
    ```

- Download and extract [Apache ActiveMQ 5.10.0 or later](http://activemq.apache.org/) and start ActiveMQ - ``` {ACTIVEMQ_HOME}$ ./bin/activemq start ```
  Please make sure mqtt transport connector is enabled in the ActiveMQ configuration file; **{ACTIVEMQ_HOME}/conf/activemq.xml**.

- Build Stratos 4.1.0 - M1 code from this **4.1.0-m1** tag, copy (from {**STRATOS_SOURCE}/products/stratos/modules/distribution/target/**) and extract the binary **apache-stratos-4.1.0-SNAPSHOT.zip** to a preferred directory (**STRATOS_HOME**). 

- Change the **MgtHostName** and **HostName** elements' values in **{STRATOS_HOME}/repository/conf/carbon.xml** such that they point to the private IP address of your local machine.

- Change the **dataBridgeConfiguration.thriftDataReceiver.hostName** element's value in **{STRATOS_HOME}/repository/conf/data-bridge/data-bridge-config.xml** to the private IP address of your machine.

- Change the property named **"java.naming.provider.url"** value to **tcp://{MB_IP}:61616** in **{STRATOS_HOME}/repository/deployment/server/outputeventadaptors/JMSOutputAdaptor.xml** file.


- Start Stratos using ``` {STRATOS_HOME}$ ./bin/stratos.sh start ``` command.


Testing M1
----------

**1. Register Kubernetes-CoreOS Host Cluster in Stratos**

- Curl Command

``` sh 
curl -X POST -H "Content-Type: application/json" -d @"kub-register.json" -k  -u admin:admin "https://localhost:9443/stratos/admin/kubernetes/deploy/group"
```

**kub-register.json**
```javascript
{
      "groupId": "KubGrp1",
      "description": "Kubernetes CoreOS cluster on EC2 ",
      "kubernetesMaster": {
                  "hostId" : "KubHostMaster1",
                  "hostname" : "master.dev.kubernetes.example.org",
                  "hostIpAddress" : "172.17.8.100",
                  "property" : [
                      {
                         "name": "prop1",
                         "value": "val1"
                      },
                      {
                         "name": "prop2",
                         "value": "val2"
                      }
                  ]
        },

        "portRange" : {
           "upper": "5000",
           "lower": "4500"
        },

        "kubernetesHosts": [
              {
                     "hostId" : "KubHostSlave1",
                     "hostname" : "slave1.dev.kubernetes.example.org",
                     "hostIpAddress" : "172.17.8.101",
                     "property" : [
                         {
                             "name": "prop1",
                             "value": "val1"
                         },
                         {
                             "name": "prop2",
                             "value": "val2"
                         }
                     ]
                },
                {
                     "hostId" : "KubHostSlave2",
                     "hostname" : "slave2.dev.kubernetes.example.org",
                     "hostIpAddress" : "172.17.8.102",
                     "property" : [
                         {
                             "name": "prop1",
                             "value": "val1"
                         },
                         {
                             "name": "prop2",
                             "value": "val2"
                         }
                     ]
                }
    ],

    "property": [
           {
                  "name": "prop1",
                  "value": "val1"
           },
           {
                  "name": "prop2",
                  "value": "val2"
           }
    ]
}
```
* Verify Kubernetes-CoreOS Host Cluster Registration
```sh
 curl  -k  -u admin:admin "https://localhost:9443/stratos/admin/kubernetes/group/KubGrp1"
```
Response:
```javascript
{"kubernetesGroup":{"description":"Kubernetes CoreOS cluster on EC2 ","groupId":"KubGrp1","kubernetesHosts":[{"hostId":"KubHostSlave1","hostIpAddress":"172.17.8.101","hostname":"slave1.dev.kubernetes.example.org","property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]},{"hostId":"KubHostSlave2","hostIpAddress":"172.17.8.102","hostname":"slave2.dev.kubernetes.example.org","property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]}],"kubernetesMaster":{"hostId":"KubHostMaster1","hostIpAddress":"172.17.8.100","hostname":"master.dev.kubernetes.example.org","property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]},"portRange":{"lower":4000,"upper":5000},"property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]}}
```

**2. Deploy a Docker Cartridge**

- Curl Command

``` sh 
curl -X POST -H "Content-Type: application/json" -d @'php-docker-cartridge.json' -k -v -u admin:admin "https://localhost:9443/stratos/admin/cartridge/definition"
```

**php-docker-cartridge.json**
```javascript
{
      "type": "php",
      "provider": "apache",
      "host": "apachestratos.org",
      "displayName": "PHP",
      "description": "PHP Cartridge",
      "version": "5.0",
      "multiTenant": "false",
      "deployerType": "kubernetes",
      "portMapping": [
         {
            "protocol": "http",
            "port": "80",
            "proxyPort": "8280"
         }
       ],
       "container": [
        {
          "imageName": "apachestratos/php-4.1.0-m1",
          "property": [
            {
             "name": "prop-name",
             "value": "prop-value"
            }
          ]
        }
      ]
 }
```

**3. Subscribe to a Docker Cartridge**	
- Curl Command

``` sh 
curl -X POST -H "Content-Type: application/json" -d @php-subscription.json -k -v -u admin:admin "https://localhost:9443/stratos/admin/cartridge/subscribe"
```
**php-subscription.json**

- Replace **payload_parameter.MB_IP** and **payload_parameter.CEP_IP** by your local machine IP in the following json;

```javascript
{
    "cartridgeType": "php",
    "alias": "myphp",
    "commitsEnabled": "false",
    "property": [
            {
             "name": "KUBERNETES_CLUSTER_ID",
             "value": "KubGrp1"
            },
	        {
             "name": "KUBERNETES_REPLICAS_MIN",
             "value": "3"
            },     
            {
             "name": "KUBERNETES_REPLICAS_MAX",
             "value": "20"
            }, 
            {
             "name": "payload_parameter.MB_IP",
             "value": "10.100.5.84"
            },       
            {
             "name": "payload_parameter.MB_PORT",
             "value": "1883"
            },       
            {
             "name": "payload_parameter.CEP_IP",
             "value": "10.100.5.84"
            },       
            {
             "name": "payload_parameter.CEP_PORT",
             "value": "7611"
            }
          ]    
}


```
**4. Unsubscribe from a Cartridge**
```sh
curl -X POST -H "Content-Type: application/json" -d 'myphp' -k -v -u admin:admin "https://localhost:9443/stratos/admin/cartridge/unsubscribe"
```

**5. Accessing PHP service**

Currently accessing via Load Balancer is not supported. You could access the service via **http://{HOST_IP}:{SERVICE_PORT}**.

In order to find the **SERVICE_PORT** issue following command in the Kubernetes Master Node.
```sh
kubecfg list services
```

Then access from one of the following URLs:
- http://172.17.8.100:{SERVICE_PORT}
- http://172.17.8.101:{SERVICE_PORT}
- http://172.17.8.102:{SERVICE_PORT}

Jira List
----------

**Sub-task**

- [STRATOS-730](https://issues.apache.org/jira/browse/STRATOS-730) - Puppet in docker image
- [STRATOS-731](https://issues.apache.org/jira/browse/STRATOS-731) - Implement tagging of docker images with Stratos version numbers
- [STRATOS-736](https://issues.apache.org/jira/browse/STRATOS-736) - create an updateable dns docker image
- [STRATOS-737](https://issues.apache.org/jira/browse/STRATOS-737) - minimise size of stratos in docker images

**Bug**
- [STRATOS-598](https://issues.apache.org/jira/browse/STRATOS-598) - IaaS provider properties not included by automated product configuration script
- [STRATOS-640](https://issues.apache.org/jira/browse/STRATOS-640) - Load balancer updates its Cluster Map every minute
- [STRATOS-649](https://issues.apache.org/jira/browse/STRATOS-649) - CLI inconsistent handling of STRATOS_URL validation
- [STRATOS-650](https://issues.apache.org/jira/browse/STRATOS-650) - command line mode does not accept options
- [STRATOS-668](https://issues.apache.org/jira/browse/STRATOS-668) - Java.naming.provider.url is incorrect in HAProxy Extension jndi.properties file
- [STRATOS-676](https://issues.apache.org/jira/browse/STRATOS-676) - LB shouldn't be re-writing http location header if Location is a hostname
- [STRATOS-677](https://issues.apache.org/jira/browse/STRATOS-677) - Instances are getting spawn when unsubscribing
- [STRATOS-682](https://issues.apache.org/jira/browse/STRATOS-682) - typo in class name
- [STRATOS-685](https://issues.apache.org/jira/browse/STRATOS-685) - Resources got loaded from Registry when publishing events to BAM
- [STRATOS-701](https://issues.apache.org/jira/browse/STRATOS-701) - References to 'incubator' in the code base
- [STRATOS-702](https://issues.apache.org/jira/browse/STRATOS-702) - HAProxy Extension won't update it's member list
- [STRATOS-706](https://issues.apache.org/jira/browse/STRATOS-706) - member terminate event should log reason
- [STRATOS-707](https://issues.apache.org/jira/browse/STRATOS-707) - Remove wso2 slf4j from cartridge agent
- [STRATOS-748](https://issues.apache.org/jira/browse/STRATOS-748) - Fails to deploy policies in UI.
- [STRATOS-779](https://issues.apache.org/jira/browse/STRATOS-779) - Stratos is creating more instances than the max limit
- [STRATOS-793](https://issues.apache.org/jira/browse/STRATOS-793) - Instructions to deploy a cartridge using the wizard is incorrect
- [STRATOS-795](https://issues.apache.org/jira/browse/STRATOS-795) - Stratos forgets about cartridges if they disappear while Stratos isn't running
- [STRATOS-798](https://issues.apache.org/jira/browse/STRATOS-798) - Error while login to Stratos in docker-integration branch
- [STRATOS-802](https://issues.apache.org/jira/browse/STRATOS-802) - Partition deployment fails in EC2
- [STRATOS-815](https://issues.apache.org/jira/browse/STRATOS-815) - Support HAProxy extension to default and service aware load balancer
- [STRATOS-820](https://issues.apache.org/jira/browse/STRATOS-820) - Error when publishing tenant subscribed event
- [STRATOS-823](https://issues.apache.org/jira/browse/STRATOS-823) - Incorporate isPublic and description properties at command line tool
- [STRATOS-846](https://issues.apache.org/jira/browse/STRATOS-846) - Failed to process instance clean up using available message processors
- [STRATOS-847](https://issues.apache.org/jira/browse/STRATOS-847) - Stratos LB mix up members of different multi-tenant clusters in 10x10 concurrency
- [STRATOS-848](https://issues.apache.org/jira/browse/STRATOS-848) - NPE thrown when deploying cartridge definition
- [STRATOS-849](https://issues.apache.org/jira/browse/STRATOS-849) - Stratos does not create specified min instance count in deployment policy

**Improvements**

- [STRATOS-651](https://issues.apache.org/jira/browse/STRATOS-651) - Add a CLI integration test suite
- [STRATOS-686](https://issues.apache.org/jira/browse/STRATOS-686) - Windows NullPointerException for LoadBalancerConfigurationTest
- [STRATOS-697](https://issues.apache.org/jira/browse/STRATOS-697) - Improvements for Clustering/simple grouping support in Stratos
- [STRATOS-699](https://issues.apache.org/jira/browse/STRATOS-699) - HAProxy Puppet Configurations
- [STRATOS-745](https://issues.apache.org/jira/browse/STRATOS-745) - Wiki - Add a section to explain Stratos configurations
- [STRATOS-756](https://issues.apache.org/jira/browse/STRATOS-756) - Ability to pass any property via Partition definition
- [STRATOS-763](https://issues.apache.org/jira/browse/STRATOS-763) - Re-organizing puppet modules structure
- [STRATOS-768](https://issues.apache.org/jira/browse/STRATOS-768) - [Wiki] Reorganize Stratos wiki structure
- [STRATOS-770](https://issues.apache.org/jira/browse/STRATOS-770) - Adding in a "Description" field to all definition types
- [STRATOS-782](https://issues.apache.org/jira/browse/STRATOS-782) - Kubernetes based cartridge deployment
- [STRATOS-790](https://issues.apache.org/jira/browse/STRATOS-790) - Messaging module refactoring to remove header based message distingushment
- [STRATOS-791](https://issues.apache.org/jira/browse/STRATOS-791) - MQTT protocol support for the messaging module
- [STRATOS-800](https://issues.apache.org/jira/browse/STRATOS-800) - Update REST endpoint with new authorization actions
- [STRATOS-807](https://issues.apache.org/jira/browse/STRATOS-807) - Re-designing cluster monitor hierarchy to support any 'entity' monitors to be plugged in
- [STRATOS-824](https://issues.apache.org/jira/browse/STRATOS-824) - [Wiki] Add troubleshooting steps - Newly created instance is not working

**New Feature**

- [STRATOS-777](https://issues.apache.org/jira/browse/STRATOS-777) - Introduce subscription filters to intercept a new subscription
- [STRATOS-781](https://issues.apache.org/jira/browse/STRATOS-781) - Stratos Kubernetes Integration
- [STRATOS-786](https://issues.apache.org/jira/browse/STRATOS-786) - Kubernetes Cluster Monitor to maintain the minimum number of replicas
- [STRATOS-787](https://issues.apache.org/jira/browse/STRATOS-787) - Kubernetes Host Cluster Registration
- [STRATOS-788](https://issues.apache.org/jira/browse/STRATOS-788) - Container API for Cloud Controller
- [STRATOS-789](https://issues.apache.org/jira/browse/STRATOS-789) - Dynamic Host Port allocation
- [STRATOS-799](https://issues.apache.org/jira/browse/STRATOS-799) - Stratos User Management and Permissions model
- [STRATOS-801](https://issues.apache.org/jira/browse/STRATOS-801) - Introduce new API methods to create/update/delete users

**Tasks**

- [STRATOS-672](https://issues.apache.org/jira/browse/STRATOS-672) - Define convention wrt tabs/spaces
- [STRATOS-719](https://issues.apache.org/jira/browse/STRATOS-719) - [Wiki] Describe all the configuration parameters in autoscaler.xml
- [STRATOS-772](https://issues.apache.org/jira/browse/STRATOS-772) - Modify mock REST endpoint for 4.1.0 changes - tenant isolation and user mgmt
- [STRATOS-822](https://issues.apache.org/jira/browse/STRATOS-822) - [Wiki] Document the debug logs that can be used to debug Stratos
- [STRATOS-850](https://issues.apache.org/jira/browse/STRATOS-850) - Update puppet scripts to support MQTT configuration


Troubleshooting Guide
---------------------


**1. How to ssh to master node?**

- Navigate to the cloned repository directory (**SETUP_HOME**)
```sh
vagrant ssh master
```

**2. How to ssh to minion-1 node?**

- Navigate to the cloned repository directory (**SETUP_HOME**)
```sh
vagrant ssh minion-1
```

**3. How to ssh to minion-2 node?**

- Navigate to the cloned repository directory (**SETUP_HOME**)
```sh
vagrant ssh minion-2
```

**4. How to list all the machines in CoreOS cluster?**

- ssh to master node
```sh
fleetctl list-machines
```

- output
```sh
core@master ~ $ fleetctl list-machines
MACHINE		IP		METADATA
07215782...	172.17.8.100	-
4b56425a...	172.17.8.102	-
bf39a4c4...	172.17.8.101	-
```


**5. How to list the available replication controllers?**

- ssh to master node
```sh
kubecfg list /replicationControllers
```
- output
```sh
core@master ~ $ kubecfg list /replicationControllers
ID                  Image(s)                           Selector         Replicas
test2.php.domain    54.254.64.141:5000/stratos-php     name=php         2
```


**6. How to list the available pods?**

- ssh to master node
```sh
kubecfg list /pods
```
- output
```sh
core@master ~ $ kubecfg list /pods
ID                                     Image(s)     Host                Labels                              Status
115bbe15-49ff-11e4-91b7-08002794b041   stratos-php  172.17.8.100/      name=php,replicationController=php   Waiting
115cc7e9-49ff-11e4-91b7-08002794b041   stratos-php  172.17.8.101/      name=php,replicationController=php   Waiting
```

**7. How to tail the kubernetes log?**

- ssh to master node
```sh
journalctl -f
```

**8. How to restart the kubernetes scheduler?**

- ssh to master node
```sh
systemctl restart scheduler
```

**9. How to list all the docker containers in a node?**

- ssh to the node
```sh
docker ps
```
- output
```sh
core@master ~ $ docker ps
CONTAINER ID        IMAGE                        COMMAND                CREATED             STATUS   PORTS
37e3303eb337        stratos-php:latest           "/bin/sh -c '/usr/lo   2 minutes ago       Up                
a3f787d0a7ae        kubernetes/pause:latest      "/pause"               2 minutes ago       Up       0.0.0.0:80->80/tcp
```

**10. How to get the IPAddress of a docker container?**

- ssh to the node
```sh
docker inspect CONTAINER-ID | grep IPAddress
```
- output
```sh
core@master ~ $ docker inspect a3f787d0a7ae | grep IPAddress
IPAddress": "10.100.56.3",
```

**10. How to ssh to a docker container?**

- ssh to the node
```sh
ssh root@CONTAINER-IPAddress
```
- enter ```g``` for password

**11. How to kill a docker container?**

- ssh to the node
```sh
docker kill CONTAINER-ID
```
- output
```sh
core@minion-1 ~ $ docker kill 6f5ba525f9ab
6f5ba525f9ab
```
- another conatiner will be created within few seconds

**12. How to get info of a specific replicationController as a json?**

- ssh to the master node
```sh
 kubecfg -json get /replicationControllers/REPLICATION-CONTROLLER-ID
```
- output
```"
core@master ~ $  kubecfg -json get /replicationControllers/test2.php.domain
{"kind":"ReplicationController","id":"test2.php.domain","creationTimestamp":"2014-10-02T07:44:58Z","resourceVersion":6701,"apiVersion":"v1beta1","desiredState":{"replicas":2,"replicaSelector":{"name":"test2.php.domain"},"podTemplate":{"desiredState":{"manifest":{"version":"v1beta1","id":"","volumes":null,"containers":[{"name":"test2-apachestratos-org","image":"54.254.64.141:5000/stratos-php","ports":[{"name":"tcp80","hostPort":80,"containerPort":80,"protocol":"tcp"}],"env":[{"name":"SERVICE_NAME","key":"SERVICE_NAME","value":"php"},{"name":"HOST_NAME","key":"HOST_NAME","value":"test2.apachestratos.org"},{"name":"MULTITENANT","key":"MULTITENANT","value":"false"},{"name":"TENANT_ID","key":"TENANT_ID","value":"-1234"},{"name":"TENANT_RANGE","key":"TENANT_RANGE","value":"-1234"},{"name":"CARTRIDGE_ALIAS","key":"CARTRIDGE_ALIAS","value":"test2"},{"name":"CLUSTER_ID","key":"CLUSTER_ID","value":"test2.php.domain"},{"name":"CARTRIDGE_KEY","key":"CARTRIDGE_KEY","value":"uLBSXhS3Kzos5xVe"},{"name":"REPO_URL","key":"REPO_URL","value":"null"},{"name":"PORTS","key":"PORTS","value":"80"},{"name":"PROVIDER","key":"PROVIDER","value":"apache"},{"name":"PUPPET_IP","key":"PUPPET_IP","value":"127.0.0.1"},{"name":"PUPPET_HOSTNAME","key":"PUPPET_HOSTNAME","value":"puppet.raj.org"},{"name":"PUPPET_DNS_AVAILABLE","key":"PUPPET_DNS_AVAILABLE","value":"null"},{"name":"PUPPET_ENV","key":"PUPPET_ENV","value":"stratos"},{"name":"DEPLOYMENT","key":"DEPLOYMENT","value":"default"},{"name":"CEP_PORT","key":"CEP_PORT","value":"7611"},{"name":"COMMIT_ENABLED","key":"COMMIT_ENABLED","value":"false"},{"name":"MB_PORT","key":"MB_PORT","value":"1883"},{"name":"MB_IP","key":"MB_IP","value":"172.17.42.1"},{"name":"CEP_IP","key":"CEP_IP","value":"172.17.42.1"},{"name":"MEMBER_ID","key":"MEMBER_ID","value":"test2.php.domaindb8e4a24-17e8-40a2-ad85-e0d0ca127887"},{"name":"LB_CLUSTER_ID","key":"LB_CLUSTER_ID"},{"name":"NETWORK_PARTITION_ID","key":"NETWORK_PARTITION_ID"},{"name":"KUBERNETES_CLUSTER_ID","key":"KUBERNETES_CLUSTER_ID","value":"KubGrp1"},{"name":"KUBERNETES_MASTER_IP","key":"KUBERNETES_MASTER_IP","value":"127.0.0.1"},{"name":"KUBERNETES_PORT_RANGE","key":"KUBERNETES_PORT_RANGE","value":"4000-5000"}]}],"restartPolicy":{"always":{}}}},"labels":{"name":"test2.php.domain"}}},"currentState":{"replicas":2,"podTemplate":{"desiredState":{"manifest":{"version":"","id":"","volumes":null,"containers":null,"restartPolicy":{}}}}},"labels":{"name":"test2.php.domain"}}
```

**12. How to get update a specific replicationController?**

- ssh to the master node
- get the replication controller info as a json as above and save it to a file
```sh
 kubecfg -json get /replicationControllers/REPLICATION-CONTROLLER-ID >> rep-controller.json
```
- edit the rep-controller.json (for example, set replicas to 0)
- update the replicationController
```sh
 kubecfg -c rep-controller.json update /replicationControllers/REPLICATION-CONTROLLER-ID
```
- output
```sh
core@master ~ $ kubecfg -c raj.json update /replicationControllers/test2.php.domain
ID                  Image(s)                         Selector                Replicas
test2.php.domain    54.254.64.141:5000/stratos-php   name=test2.php.domain   0
```
- it will delete all the pods imediately


**13. How to delete a pod?**

- ssh to the master node
- get the pod ID using ```kubecfg list /pods```
```sh
kubecfg delete pods/POD-ID
```
- output
```sh
core@master ~ $ kubecfg delete pods/3ed5c5a6-4a0a-11e4-91b7-08002794b041
I1002 08:01:25.351501 02112 request.go:292] Waiting for completion of /operations/67
Status
success
```

**14. How to delete a replicationController?**

- ssh to the master node
- get the replicationController ID using ```kubecfg list /replicationControllers```
```sh
kubecfg delete replicationControllers/REPLICATION-CONTROLLER-ID
```
- output
```sh
core@master ~ $ kubecfg delete /replicationControllers/test2.php.domain
I1002 08:32:47.187198 02150 request.go:292] Waiting for completion of /operations/86
Status
success
```
- if you unsubscribe to the cartridge, replicationControllers, pods and containers for that service cluster will be wiped out
