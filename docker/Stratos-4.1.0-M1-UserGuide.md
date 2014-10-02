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
