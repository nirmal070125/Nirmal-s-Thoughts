Stratos 4.1.0 - M1 - Developer Preview - User Guide
=========


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
    * Run ``` up.sh ``` script file - ``` {SETUP_HOME}$ ./up.sh ``` (You might have to use ```sudo``` in some cases.
    * Above command will start-up 4 VMs, namely discovery, master, minion-1 and minion-2.
    * Pull Stratos PHP Docker Image from **TODO** into master node. **TODO add steps**

- Download and extract [Apache ActiveMQ 5.10.0 or later](http://activemq.apache.org/) and start ActiveMQ - ``` {ACTIVEMQ_HOME}$ ./bin/activemq start ```

- (**Run setup.sh???** ) Build Stratos 4.1.0 - M1 code from this **FIXME** [tag](), copy (from {**STRATOS_SOURCE}/products/stratos/modules/distribution/target/**) and extract the binary **apache-stratos-4.1.0-SNAPSHOT.zip** to a preferred directory (**STRATOS_HOME**).

Testing M1
----------

**1. Register Kubernetes-CoreOS Host Cluster in Stratos**

- Curl Command

``` sh 
curl -X POST -H "Content-Type: application/json" -d @"kub-register.json" -k  -u admin:admin "https://127.0.0.1:9443/stratos/admin/kubernetes/deploy/group"
```

**kub-register.json**
```javascript
{
      "groupId": "KubGrp1",
      "description": "Kubernetes CoreOS cluster on EC2 ",
      "kubernetesMaster": {
                  "hostId" : "KubHostMaster1",
                  "hostname" : "master.dev.kubernetes.example.org",
                  "hostIpAddress" : "127.0.0.1",
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
           "lower": "4000"
        },

        "kubernetesHost": [
      	      {
                     "hostId" : "KubHostSlave1",
                     "hostname" : "slave1.dev.kubernetes.example.org",
                     "hostIpAddress" : "127.0.0.1",
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
         },
         {
            "protocol": "https",
            "port": "443",
            "proxyPort": "8243"
         }
       ],
       "container": [
        {
          "imageName": "apachestratos/stratos-php",
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
```javascript
{
    "cartridgeType": "php",
    "alias": "myPhp",
    "commitsEnabled": "false",
    "property": [
            {
             "name": "KUBERNETES_CLUSTER_ID",
             "value": "KubGrp1"
            },
	    {
             "name": "KUBERNETES_REPLICAS_MIN",
             "value": "2"
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
curl -X POST -H "Content-Type: application/json" -d 'myPhp' -k -v -u admin:admin "https://localhost:9443/stratos/admin/cartridge/unsubscribe"
```

**5. Accessing PHP service**

**TODO**


Troubleshooting Guide
-------------


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
-output
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
-output
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
-output
```sh
core@master ~ $ docker inspect a3f787d0a7ae | grep IPAddress
IPAddress": "10.100.56.3",
```

**10. How to ssh to a docker container?**

- ssh to the node
```sh
ssh root@CONTAINER-IPAddress
```
enter ```g``` for password

**11. How to kill a docker container?**

- ssh to the node
```sh
docker kill CONTAINER-ID
```
-output
```sh
core@minion-1 ~ $ docker kill 6f5ba525f9ab
6f5ba525f9ab
```
-Another conatiner will be created within few seconds

