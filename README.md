# Setup scripts for Maximo Application Suite on Redhat CodeReady Containers

```mas-standalone-install``` is installation scripts for Maximo Application Suite (MAS) 8.7 on Red Hat CodeReady Containers (CRC). This scripts enable to install all prereqs, MongoDB, UDS, SLS, cert-manager, service binding operator, and MAS core on CRC env. An entitlement key for IBM Container Registry is a only required parameter for MAS core installation. No need the other parametersand they are configured by default for CRC. If you want to setup prereqs on a small OpenShift environment without exaggerated configurations, you can use this scripts with custom variables for your environment. Please consider [Ansible scripts](https://ibm-mas.github.io/ansible-devops) or any other tools for large projects.

Detailed for the scripts here: https://www.linkedin.com/pulse/running-maximo-manage-stand-alone-openshift-server-red-nishimura/

Disclaimer: I (and also IBM) do NOT support running the MAS on CRC with this guide. You need to have skills to debug K8s/OpenShift to try out the installation.

## Usage

Install CRC, prereqs, and MAS Core.

```shell
$ sudo yum update && yum install jq git
$ crc setup
$ crc config set cpus 16
$ crc config set memory 48000
$ crc config set disk-size 300
$ crc config set disable-update-check true
$ crc start
$ eval $(crc oc-env)
$ oc login -u kubeadmin -p ************ https://api.crc.testing:6443
```

This is using the manual process and uses shell scripts 

```shell
$ git clone https://github.com/pvkvicky2000/mas-standalone-install
$ cd mas-standalone-install
$ export ENTITLEMENT_KEY=<Your Entitlement Key> # Get from https://myibm.ibm.com/products-services/containerlibrary
$ ./setup.sh
```

This script uses IBM containers to do one click install , assuming that the license.dat file also exists and is in the path.

```shell
###docker install
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

###Only for CRC
crc config set enable-cluster-monitoring true
crc stop && crc start

oc login -u kubeadmin -p ********** https://api.crc.testing:6443

export CRC_HOST_IP=192.168.130.1

/home/vijay/crc/mas-standalone-installation/managed-nfs-storage.sh

####

docker run -ti -v /path/to/license.dat:/opt/app-root/src/license.dat quay.io/ibmmas/ansible-devops:latest bash

export ENTITLEMENT_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXX

export IBM_ENTITLEMENT_KEY=$ENTITLEMENT_KEY
export MAS_ENTITLEMENT_KEY=$ENTITLEMENT_KEY

export storage_class=managed-nfs-storage   ## Chanage this for storage class

export MAS_INSTANCE_ID=local   ### Change this for instance
export MAS_WORKSPACE_ID=local  ### Change this for instance
export MAS_APP_ID=manage
export MAS_APPWS_COMPONENTS="base=latest,health=latest"
export MANAGE_AIO_FLAG=false

export MAS_CONFIG_DIR=/opt/app-root/src/masconfig

export SLS_LICENSE_ID=10005ad2dea7    ### get this from license.dat file
export SLS_LICENSE_FILE=/opt/app-root/src/license.dat   

export UDS_CONTACT_EMAIL=pvkvicky@gmail.com  
export UDS_CONTACT_FIRSTNAME=Vijay
export UDS_CONTACT_LASTNAME=Krishna

export PROMETHEUS_ALERTMGR_STORAGE_CLASS=$storage_class           
export PROMETHEUS_STORAGE_CLASS=$storage_class
export PROMETHEUS_USERWORKLOAD_STORAGE_CLASS=$storage_class
export GRAFANA_INSTANCE_STORAGE_CLASS=$storage_class
export MONGODB_STORAGE_CLASS=$storage_class
export UDS_STORAGE_CLASS=$storage_class
export DB2_META_STORAGE_CLASS=$storage_class
export DB2_DATA_STORAGE_CLASS=$storage_class
export DB2_BACKUP_STORAGE_CLASS=$storage_class
export DB2_LOGS_STORAGE_CLASS=$storage_class
export DB2_TEMP_STORAGE_CLASS=$storage_class

export DB2_INSTANCE_NAME=maxdb8               ## set the DB parameters
export DB2_DBNAME=MAXDB8
export DB2_WORKLOAD=ANALYTICS
export DB2_LDAP_USERNAME=maximo
export DB2_LDAP_PASSWORD=maximo_2012
export DB2_CPU_REQUESTS=2000m
export DB2_CPU_LIMITS=2000m
export DB2_MEMORY_REQUESTS=4Gi
export DB2_MEMORY_LIMITS=4Gi

export MONGODB_REPLICAS=1                  ## set the Mongo parameters
export MONGODB_NAMESPACE=mongo

export MAS_CHANNEL=8.9.x                   ## set the MAS Version

export SLS_DOMAIN=apps-crc.testing         ## set the Mas domain name 


oc login --token=xxxx --server=https://myocpserver
rm -rf /opt/app-root/src/masconfig
mkdir /opt/app-root/src/masconfig

ansible-playbook ibm.mas_devops.oneclick_core
ansible-playbook ibm.mas_devops.oneclick_add_manage

```


#This should be run only in case the onclick_add_manage is not used
The additional script, ```mas-ws_setup.sh```, enables to complete MAS workspace configuration except license file uploading to start deployment for MAS apps like Maximo Manage. To put your license file path to ```SLS_LICENSE_FILE```, all of the steps of Suite setup are completed without any manual interventions.

```shell
$ export UDS_EMAIL=<Your e-mail>
$ export UDS_LASTNAME=<Your last name>
$ export UDS_FIRSTNAME=<Your first name>
$ # export SLS_LICENSE_FILE=<Your license path>
$ ./mas-ws_setup.sh
```

The required information to complete Suite setup can be obtained from the following command.

```shell
$ ./get_setup_params.sh
```

For a small OCP enviroment, use ```-p``` option with required environment variables.

```shell
$ export ENTITLEMENT_KEY=<Your Entitlement Key> # Get from https://myibm.ibm.com/products-services/containerlibrary
$ ./setup.sh -p
```


## Environment varialbe list

```
ENTITLEMENT_KEY
```

A key for accessing IBM Container Registry. It can be obtained from here: https://myibm.ibm.com/products-services/containerlibrary

```
MAS_INSTANCE_ID (default: crc)
```

An instance ID for the deployment. https://www.ibm.com/docs/en/mas87/8.7.0?topic=installation-instance-requirements#instance_name

```
MAS_DOMAIN_NAME (default: mas.apps-crc.testing)
```

A base domain name for the deployment. https://www.ibm.com/docs/en/mas87/8.7.0?topic=installation-instance-requirements#dns

```
MAS_CHANNEL (default: 8.7.x)
```

A channel for the MAS subscription. This specifies which version to be used in the instance.

```
MAS_WORKSPACE_ID (default: dev)
```

A workspace ID for the instance. https://www.ibm.com/docs/en/mas87/8.7.0?topic=installation-instance-requirements#workspace

```
MAS_WORKSPACE_NAME (default: Maximo dev)
```

A description for the workspace ID. https://www.ibm.com/docs/en/mas87/8.7.0?topic=installation-instance-requirements#workspace

    
```
SLS_NAMESPACE (default: ibm-sls)
```

A namespace for Suite License Service.

```
SLS_STORAGE_CLASS (default: local-path)
```

A storage class (RWO) for persistent storage in the SLS operator. Use appropriate storage class provided by cloud provider or on-premise solutions in OCP env.

```
SLS_DOMAIN_NAME (default: apps-crc.testing)
```

A base domain name for SLS.

```
SLS_LICENSE_FILE
```

A license file path for uploading MAS AppPoints token license to SLS.

```
UDS_EMAIL
```
A contact email address to use for User Data Service communication. 

```
UDS_LASTNAME
```
The given name of the owner of the provided contact email address.

```
UDS_FIRSTNAME
```

The surname of the owner of the provided contact email address.

```
UDS_STORAGE_CLASS (default: local-path)
```

A storage class (RWO) for persistent storage in the UDS operator. Use appropriate storage class provided by cloud provider or on-premise solutions in OCP env.

```
MONGODB_NAMESPACE (defualt: mongodb)
```

A namespace for the MongoDB operator.

```
MONGODB_REPLICAS (default: 3)
```

A number of instances for MongoDB service.

```
MONGODB_CPU_REQUEST (default: 100m)
```

A request parameter for CPU in the MongoDB instance. See https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits

```
MONGODB_MEM_REQUEST (default: 256Mi)
```
A request parameter for memory in the MongoDB instance. See https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits

```
MONGODB_CPU_LIMIT (default: 1)
```

A limit parameter for CPU in the MongoDB instance. See https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits

```
MONGODB_MEM_LIMIT (default: 1Gi)
```

A limit parameter for memory in the MongoDB instance. See https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits

```
MONGODB_STORAGE_SIZE (default: 20Gi)
```

The storage claim size of the provided storage class for the MongoDB instances.

```
MONGODB_STORAGE_LOG_SIZE (default: 2Gi)
```

The storage claim size of the provided storage class for the MongoDB logs.

```
MONGODB_STORAGE_CLASS (default: local-path)
```

Storage class (RWO) for persistent storage in the UDS operator. Use appropriate storage class provided by cloud provider or on-premise solutions in OCP env.

```
MONGODB_PASSWORD (default: auto-generated)
```

A MongoDB password to access the MongoDB servcies. This parameter is automatically generated when it specified in the environment variable.
