Stratos 4.1.0 - M1 - Developer Preview - User Guide
=========

Table of Content
----------

- Main Features
- Pre-requisite
- Testing M1
- Jira List
- Troubleshooting Guide


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
