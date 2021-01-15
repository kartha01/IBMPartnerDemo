# Install instructions for Cloud Pak for Data (work in progress)
- [Migrating from Cloud Pak for Data 3.0](#migrating-from-cloud-pak-for-data-3.0)
- [Provision OpenShift](https://tjmcmanus.github.io/IBMPartnerDemo/#creating-an-open-shift-cluster)
- [Provision the Control plane using IBM Cloud tile](#provision-the-control-plane-using-ibm-cloud-tile)
  * [Building the configuration](#building-the-configuration)
  * [Review Services installed](#review-services-installed)
  - [Troubleshooting and managing Cloud Pak for Data through the console](#troubleshooting-and-managing-cloud-pak-for-data-through-the-console)
    * [Manage Deployments](#manage-deployments)
    * [Manage Users](#manage-users)
    * [Viewing Logs](#viewing-logs)    
- [Setting up the Cloud Pak for Data Client](#setting-up-the-cloud-pak-for-data-client)
  * [Installing the Cloud Pak for Data Control Plane by command line](#installing-the-cloud-pak-for-data-control-plane-by-command-line)
  * [To enable or disable the default admin users](#to-enable-or-disable-the-default-admin-users)
  * [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)
- [Data Virtualization](#data-virtualization)
  * [Install Data Virtualization](#install-data-virtualization)
  * [Set up Data Virtualization](#set-up-data-virtualization)
  * [What can be done with Data Virtualization and Watson Knowledge Catalog](#what-can-be-done-with-data-virtualization-and-watson-knowledge-catalog)
- [Watson Studio](#watson-studio)
  * [Install](#install-watson-studio)
  * [Set up](#set-up-watson-studio)
  * [Increase the capacity or scale up](#increase-the-capacity-or-scale)
- [Watson Machine Learning](#watson-machine-learning)
  * [Install](#install-watson-machine-learning)
  * [Set up](#set-up-watson-machine-learning)
  * [Increase the capacity or scale up your service](#increase-the-capacity-or-scale-up-your-service)  
- [Watson OpenScale](#watson-openscale)
  * [Install Watson OpenScale](#install-watson-openscale)
  * [Set up Watson OpenScale](#set-up-watson-openscale)
- [Db2 Warehouse](#db2-warehouse)  
  * [Install Db2 Warehouse (SMP)](#install-db2-warehouse-smp)
  * [Provision a Database instance](#provision-a-database-instance)
  * [Create a Table](#create-a-table)
  * [Uninstalling DB2 Warehouse.](#uninstalling-db2-warehouse)
- [Db2 Advanced](#db2-advanced)  
  * [Install Db2 Advanced (SMP)](#install-db2-advanced)
  * [Provision a Database instance](#provision-an-advanced-database-instance)
  * [Create a Table](#create-a-table)
  * [Uninstalling DB2 Advanced](#uninstalling-db2-advanced)
- [Installing DataStage service](#installing-datastage-service)
  * [Create a Transformation Project](#create-a-transformation-project)
  * [Uninstalling DataStage.](#uninstalling-datastage)
- [Installing Analytics Dashboards](#installing-analytics-dashboards)
  * [Uninstalling Analytics Dashboards](#uninstalling-analytics-dashboards)
- [Installing Analytics Engine (Spark Clusters)](#installing-analytics-engine-spark-clusters)
  * [Provision Analytics Engine instance](#provision-analytics-engine-instance)
  * [Resource constraints](#resource-constraints)
  * [Uninstalling Analytics Engine.](#uninstalling-analytics-engine)
- [Installing Cognos Analytics](#installing-cognos-analytics)
  * [Provision Cognos Analytics instance](#provision-cognos-analytics-instance)
  * [Uninstalling Cognos Analytics](#uninstalling-cognos-analytics)  
- [Watson Knowledge Catalog](#watson-knowledge-catalog)
  * [Installing Watson Knowledge Catalog service](#installing-watson-knowledge-catalog-service)  
  * [Uninstalling Watson Knowledge Catalog](uninstalling-watson-knowledge-catalog)

## The AI Ladder and how Cloud Pak for Data
<iframe width="560" height="315" src="https://www.youtube.com/embed/pN_cZq-ov6Y" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Migrating from Cloud Pak for Data 3.0
1. [Set up the Cloud Pak for Data Client](#setting-up-the-cloud-pak-for-data-client)
1. Login to Openshift
1. Run `env` to verify that the following variables are exported
  - OpenShift 4.x
  ~~~
  export NAMESPACE=zen
  export STORAGE_CLASS=ibmc-file-gold-gid
  export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
  export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
  ~~~
1. Convert the control plane metadata to be compatible with the new version 3.5 model by running the following command:
  `./cpd-cli operator-upgrade -n ${NAMESPACE}`  
1. Update the Control plane or **lite** assembly.  This is a dry run, so you understand what will be updated.  This takes about 60 seconds.
  `./cpd-cli adm --repo ./repo.yaml --namespace ${NAMESPACE} --latest-dependency --assembly lite`
1. Apply the changes.  This takes about 90 seconds.
  `./cpd-cli adm --repo ./repo.yaml --namespace ${NAMESPACE} --latest-dependency --assembly lite --apply`
1. Upgrade the control plane, but start with a dry run first to validate that you have everything in order.
 `./cpd-cli upgrade --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly lite  --dry-run`
1. Do the actual upgrade. This took me 26 minutes.
  `./cpd-cli upgrade --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly lite`
1. Check the status of the upgrade.  This will give you a list of the other services to update and patch.
  `./cpd-cli status --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly lite`
1. Apply security changes for the services that you have installed.  I did Data Virtualization.  Removing the `--apply` flag will be a dry run.
  `./cpd-cli adm --repo ./repo.yaml --namespace ${NAMESPACE} --latest-dependency --assembly dv --apply`  
1. Upgrade the services that you have installed.  I did Data Virtualization and it took 90 minutes.  **Note** on a few upgrades of services, I have seen the upgrade fail, but when I ran it again it succeeded.  Most errors were waiting on an Operator Lock.  
  `./cpd-cli upgrade --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly service name>`    
1. Upgrade any running services instances.  Each service has its own prerequites, so please consult the knowledge center.   I have not needed to update any service instances.   DV is one that I deleted before I could update.


## Provision the Control plane using IBM Cloud tile
1. **Click** ***Catalog***
1. **Click** ***Software***
1. **Filter** on ***Analytics*** or In the search box **enter** ***Cloud Pak*** and press **enter**.  This should list the available Cloud Paks that can be provisioned. In these instructions, you are going to install ***Cloud Pak for Data***.
1. **Click** on the ***Cloud Pak for Data*** tile.
1. **Click** on the ***Readme*** tab to review what you can provision using ***Schematics*** (Terraform).

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Building the configuration
1. On the ***create*** tab, this is where we will build the configuration of the Cloud Pak for Data clusters
1. First section describes the bare minimum configuration.  For Cloud Pak for Data it is as follows.  `Each cluster must meet a set of minimum requirements: 3 nodes with 16 cores, 64GB memory, and 25GB disk per node.`  As I know I want to add Db2 Warehouse and provision a Data Virtualization instance, I will start with 4 node OpenShift cluster.  As I add more features, I am needing more capacity when I provision the instance.
1. Select the cluster that was previously created.  Where you created project `zen`.
1. Select the project.   While you can pick `default`, you would be better served picking the previously create project like `zen`.
1. Rename the workspace to something you will remember like `CPDDemo-HealthCare` for a health care demo.
1. Pick your resource group, if you did not create one, just use `Default`.
1. You can tag your environment for easy search.  This is not required.
1. Scroll down and select ***Run Script*** if you do not have permissions, you can share a link and have an administrator execute this.
1. Scroll down and select the services that you desire to add to Cloud Pak for Data. **Note:** This is a first time set up action only. Currently, Once provisioned, toggling true then false will lead to no further action.  You will need to uninstall or install again, which will be shown later.  To install all of these, you will need significantly more infrastructure.   The Master or controller OCP Nodes are part of a Service Account.  The Worker or Compute Nodes are added to your account.   VLAN Spanning is needed for this to work.  To get a better idea for sizing infrastructure, you can used this [Sales Config tool](https://salesconfig.ibmcloudpack.com/sales/#/).  It may require an IBM approval for access, which you will use your IBM ID.
- **AI** (Apache Spark, IBM Streams, RStudio Server, Watson Studio, Watson Machine Learning, Watson OpenScale)
- **Dashboarding** (Analytic Dashboards)
- **Data Governance** (Watson Knowledge Catalog, Open Source Management)
- **Data Sources** (Data Virtualization, Db2 Warehouse).  
1. On the right, you should see your entitlements and a yellow box, which will turn green after you run the script below.
1. **Check the box** and to ***accept the license***.   
1. Once install is complete, you may need to [check for and apply patches](#how-can-i-patch-a-service-or-control-plane).  Hopefully in the future you will always pull the latest from the container registry.


[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Review Services installed
1. From the Schematics workspace, **Click** `Offering dashboard`
1. Depending on your browser, you will need to accept the certificate.  General path is Advanced then accept risk.  This will bring you to the login page. Bookmark this for future usage.
1. **Login** using default credentials as seen in video.
1. Along the black menu bar at the top the icon to the right on the magnifying glass is `Services`.  **Click** on this link.  
1. Take a look at the services that are `enabled`.   These have been installed or deployed. Depending on what was selected any of these could be `available` or `enabled`.  
  - **AI** (Apache Spark, IBM Streams, RStudio Server, Watson Studio, Watson Machine Learning, Watson OpenScale)
  - **Dashboarding** (Analytic Dashboards)
  - **Data Governance** (Watson Knowledge Catalog, Open Source Management)
  - **Data Sources** (Data Virtualization, Db2 Warehouse).  
1. Some are set up automatically and some you need to provision.
<iframe width="560" height="315" src="https://www.youtube.com/embed/vYS7xwk0fn8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>  

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

## Troubleshooting and managing Cloud Pak for Data through the console

### Manage Deployments
 ***coming soon***  
  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)    
### Manage Users
 ***coming soon***  
 <iframe width="600" height="322" src="https://www.youtube.com/embed/8D-IOawofAo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)    
### Viewing logs
 ***coming soon***  
  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)  

## Setting up the Cloud Pak for Data Client
1. You have already set up the client environment.  If not go have to the [first page](README.md) and execute these step under **Installing the client environment**  You will have logged into the OpenShift cluster, set the project to ***zen*** already built the encrypted route to the internal container repository.  This is where all of your containers and helm charts will be stored.  If ***zen*** doesn't exist, then your probably went with ***default***.
1. Here is a link to the [knowledge center for Cloud Pak for Data](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/cpd/install/install.html) installation instructions.  The next steps are a sequential version for this environment.  
1. How do I get the installer? Click the [CPD Github](https://github.com/IBM/cpd-cli/releases) to download the **Enterprise Edition**.  Named ***cpd-cli-darwin-EE-3.5.1.tgz***.  File is about 128MB.
1. Unarchive the client bits. You should find the following.
  - cpd-cli
  - repo.yaml
  - plugins  (directory)
  - LICENSES (directory)
1. Find the file called `repo.yaml`  This is a simple file, but key. Here is where you will add the `apikey` to get access to the container registry for the additional Services.  There are services where you will need to access different [container repository namespaces](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/cpd/install/installation-files.html)  I will include a repo.product.yaml when I document the install.
1. In a web browser, **Login** and access [the entitlement registry](https://myibm.ibm.com/products-services/containerlibrary).  **Click** ***Get entitlement key*** on the left side above Library.  Either ***Generate a key*** or ***Copy the existing key***.  This will populate your scratchpad where you can paste it into the `repo.yaml` file.
1.  **Paste** the ***apikey*** into the Yaml file to the right of the `apikey:`  After you paste, it should something like this only longer.  ***Note*** Leave **namespace** as is as I have not yet found a use for it.
~~~
---
fileservers:
  -
    url: "https://raw.github.com/IBM/cloud-pak/master/repo/cpd/3.5"
registry:
  -
    apikey: eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE1NzYwMjA4NTAsImp0aSI6ImViMzliZTk4MDJjNzRjYjM4M2Q3YjYxYTYzNmUwY2VkIn0.5Czky1_pLPLIEzScJzQ_qVNxo3v3EXyVGK7a7Dafcrg
    name: base-registry
    namespace: ""
    url: cp.icr.io/cp/cpd
    username: cp
~~~~
  **Note:**  While the documentation denotes Linux, using **cpd-linux**, the video uses **cdp-darwin** for Mac, **cpd-windows.exe** for windows.  

1. Run the client command to verify that it works.   On Mac you will need to verify the command by launching it from finder.  
 ~~~
 ./cpd-cli
 ~~~
***NOTE:*** `cpd-cli` will run different operators to execute commands.  These operators or executors are under `lib/os name`  These are `config`, `cpd-base`, `cpdbr`, `cpdtool`, and `service-instance`.  On Mac, you may see a sercurity pop-up running something like `./cpd-cli status --repo ./repo.yaml --namespace zen --assembly lite --patches --available-updates`   You need to tell Mac that you approve to run these.  The simplest way is to open Finder and move to the executors and **open in Terminal** and accept **Open**.  This tell the system to allow you to run these. There may be something similar on Windows, but I have not tried this.

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

## Installing the Cloud Pak for Data Control Plane by command line
**Note:** This is not a recommended option on IBM Cloud. Since possible, I am documenting.
1. Follow the instructions in [Setting up the Cloud Pak for Data Client](#adding-additional-services)
1. Follow the instructions in [Retrieving the token to log into OpenShift on IBM Cloud](#retrieving-the-token-to-log-into-openshift-on-ibm-cloud)
1. Create your OpenShift namespace or project that you will install Cloud Pak for Data.
  - Example: `oc project zen`
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
  ~~~
  export NAMESPACE=zen
  export STORAGE_CLASS=ibmc-file-gold-gid
  export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
  export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
  ~~~
1. Make sure all the [node settings](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/node-settings.html) are set on the worker nodes. All this is done for you [Provision the Control plane using IBM Cloud tile](#provision-the-control-plane-using-ibm-cloud-tile)
1. Install the control plane or the ***lite*** assembly
  ~~~
  ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly lite
  ~~~
1. Verify the installation  
  ~~~
  ./cpd-cli status --namespace ${NAMESPACE} --assembly lite
  ~~~
1. Check for patches
  ~~~
  ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly lite
  ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
  - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)


### To enable or disable the default admin users
1. It is suggested to connect your user repository to an LDAP system.  
1. Once connected to LDAP, you should ***disable*** the default **admin** user.
1. To do this you will run a script in the user management pod.
 ~~~
 export NAMESPACE=zen
 oc exec -it -n $NAMESPACE  $(oc get pod -n $NAMESPACE  -l component=usermgmt | tail -1 | cut -f1 -d\ ) -- bash -c "/usr/src/server-src/scripts/manage-user.sh --disable-user admin"
 ~~~
1. To reenable **admin** user run the following, which will prompt you for a new password:
 ~~~
 export NAMESPACE=zen
 oc exec -it -n $NAMESPACE  $(oc get pod -n $NAMESPACE  -l component=usermgmt | tail -1 | cut -f1 -d\ ) -- bash -c "/usr/src/server-src/scripts/manage-user.sh --enable-user admin"
 ~~~
 **Note:** If you have lost or forgotten your CPD Admin password, you can log into OpenShift then execute the enable command which prompts you for a new password.

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### How can I patch a service or control plane
From time to time any software needs a patch for security reasons, new feature or a bug fix.  How do you know that there is a patch for a specifica service, common services or the control plane.   Take a [look here for v3.0.x](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/patch/avail-patches.html).  This document has a link to each of the service patches and any extra work that might be needed.   Most patches need a prerequisite patch for the common services.
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
   export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
   ~~~
1. Check for patches
  ~~~
  ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly lite
  ~~~
1. Run the command to patch the common services. Notice that the command is `patch`, `patch-name` is ***cpd-3.5.1-lite-patch-1***.  This name will change after this writing. Note the assembly name can be `lite`, `wkc` or `wsl`.  
 ~~~
 ./cpd-cli patch --repo ./repo.yaml  --namespace ${NAMESPACE} --cluster-pull-prefix image-${LOCAL_REGISTRY}/${NAMESPACE}  --insecure-skip-tls-verify --assembly lite  --patch-name cpd-3.5.1-lite-patch-1 --action online
 ~~~
1. Verify that the patch has been applied.
 ~~~
 Toms-MBP:~ tjm$ oc project zen
 Already on project "zen" on server "https://c106-e.us-south.containers.cloud.ibm.com:31432".
 Toms-MBP:~ tjm$ oc describe cpdinstall cr-cpdinstall | grep "Patch Name:" | sort | uniq | cut -d: -f2
       cpd-3.0.1-lite-patch-5
 ~~~
1. you can repeat this pattern, replacing the values to the right of **assembly**  and **patch-name**

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)


## Data Virtualization
### Install Data Virtualization
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
   export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
   ~~~
 1. Set the security aspects for Watson Studio to install properly
   ~~~
   ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly dv
   ~~~
 1. Deploy **Data Virtualization** by running the following:
   ~~~
   ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly dv
   ~~~  
1. Verify the installation  
   ~~~
   ./cpd-cli status --namespace ${NAMESPACE} --assembly dv
   ~~~
   1. Check for patches
   ~~~
   ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly dv
   ~~~
   1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
     - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Set up Data Virtualization
  1. **Click** on the ***Services*** icon to get to the services catalog.
  1. Go to the Data Virtualization tile, you see to "Provision Instance".
  1. **Click** on ***Provision Instance***.
  1. Then **Click** ***Configure***.  ***Check*** the box to the left of `Set up semaphore parameters for Data Virtualization automatically`.
  1. **Click** ***Configure***.
  1. Select the number of nodes and cores to allocate.  For a demo system defaults are enough.  **Note:** Depending on how many node you have in your worker pool, this could fail provisioning with a not enough resource message.  If you configure a default 3 16/64 workers and install all of the services, this message can appear.  Resizing your worker pool to 4 will get you around this.
  1. Make sure your change the storage classes from `default` to `ibmc-file-gold-gid`.  Choosing default could cause a failure.
  1. Once provisioned, set some users to have ***Data Engineer*** role.
    - Go to the Navigator on the left and scroll down to ***Administer***  **Click** on ***Manage Users***.
    - On the right of the list of users, you will see a pencil, which indicates ***edit***.  **Click** the pencil for the user you desire to edit. If you pick ***admin***, you will see this is a new role added based on the Data Virtualization service.  
  1. Go to ***My Instance***   
  1. Once complete, you will see **Data Virtualization** under the **Collect** area on the left table of contents.  From here you can add data sources or pointers to file folders.
  **Note:** I have not yet tested this with ***Remote Data Connections*** aka  local file folders.
### What can be done with Data Virtualization and Watson Knowledge Catalog
<iframe width="600" height="322" src="https://www.youtube.com/embed/SU0aRs8wpgc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

## Watson Studio

### Install Watson Studio
***Coming Soon Need to add override file***
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
   export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
   ~~~
 1. Set the security aspects for Watson Studio to install properly
   ~~~
   ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly wsl
   ~~~
 1. Deploy **Watson Studio** by running the following:
   ~~~
   ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly wsl
   ~~~  
1. Verify the installation  
   ~~~
   ./cpd-cli status --namespace ${NAMESPACE} --assembly wsl
   ~~~
   1. Check for patches
   ~~~
   ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly wsl
   ~~~
   1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
     - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Set up Watson Studio
1. If you pick Watson Studio or Watson Machine Learning tiles, there will be no actions to take. You can proceed and create a project.

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Increase the capacity or scale up
1. To scale services, you will need to have the command line installed.  (Instruction under the ***Adding additional services*** section)
1. You can scale **up** the services by executing these commands for either **wsl**.  **Note:** There is currently no scale down function.
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
   export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
   ~~~
1. Scale up **Watson Studio** by running the following (config sizes are small, medium and large):
   ~~~
   ./cpd-cli scale -n $NAMESPACE --config medium  --load-from ./cpd-cli-workspace  -a wsl
   ~~~
1. Scale down **Watson Studio** by running the following (config sizes are small, medium and large):
   ~~~
   ./cpd-cli scale -n zen --config small  --load-from ./cpd-cli-workspace -a wsl
   ~~~

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)
## Watson Machine Learning
### Install Watson Machine Learning
   ***Coming Soon Need to add override file***
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
      ~~~
      export NAMESPACE=zen
      export STORAGE_CLASS=ibmc-file-gold-gid
      export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
      export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
      ~~~
1. Set the security aspects for Watson Machine Learning to install properly
      ~~~
      ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly wml
      ~~~
1. Download the (override-wml.yaml)[override-wml.yaml]
1. Deploy **Watson Machine Learning** by running the following:
      ~~~
      ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly wml --override override-wml.yaml
      ~~~  
1. Verify the installation  
      ~~~
      ./cpd-cli status --namespace ${NAMESPACE} --assembly wml
      ~~~
1. Check for patches
      ~~~
      ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly wml
      ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
      - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Increase the capacity or scale up your Watson Machine Learning service
1. To scale services, you will need to have the command line installed.  (Instruction under the ***Adding additional services*** section)
1. You can scale **up** the services by executing these commands for either **wml**.  **Note:** There is currently no scale down function.
1. Run env to verify that the following variables are exported .

 - OpenShift 4.x
    ~~~
    export NAMESPACE=zen
    export STORAGE_CLASS=ibmc-file-gold-gid
    export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
    export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
    ~~~  
1. Scale up **Watson Machine Learning** by running the following (config sizes are small, medium and large):
    ~~~
    ./cpd-cli scale -n $NAMESPACE --config medium  --load-from ./cpd-cli-workspace  -a wml
    ~~~
1. Scale down **Watson Machine Learning** by running the following (config sizes are small, medium and large):
    ~~~
    ./cpd-cli scale -n zen --config small  --load-from ./cpd-cli-workspace -a wml
    ~~~

   [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

## Watson OpenScale
### Install Watson OpenScale
1. When setting up OpenScale, there are 2 pre-prerequisites.
   - Watson Machine Learning (Local or remote)
    - [Local then install](#watson-machine-learning)
    - Remote then document the following parameters:
   - Db2 Database (Db2 Advanced or DB2 Warehouse Local or remote)
    - If no remote Db2 instance available, then install either [Db2 Advanced](#db2-advanced) or [Db2 Warehouse](#db2-warehouse)
1. Run env to verify that the following variables are exported
 - OpenShift 4.x
  ~~~
  export NAMESPACE=zen
  export STORAGE_CLASS=ibmc-file-gold-gid
  export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
  export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
  ~~~
1. Set the security aspects for Watson Machine Learning to install properly
  ~~~
  ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly aiopenscale
  ~~~
1. Deploy **Watson OpenScale** by running the following:
  ~~~
  ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --assembly aiopenscale
  ~~~  
1. Verify the installation  
   ~~~
  ./cpd-cli status --namespace ${NAMESPACE} --assembly aiopenscale
  ~~~
1. Check for patches
    ~~~
    ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly aiopenscale
    ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
    - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

   [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)


### Set up Watson OpenScale
1. For Watson OpenScale Service, there is an `Open` button which launch a UI to help configure the initial information.
1. Let's provision OpenScale.
1. You will be met with an ***auto setup*** to configure.  
1. First step is accept the local WML instance.
1. Enter connection details for a Db2 Database.  I am using a DB2 Warehouse that I installed later on.  You need to have Db2/Db2 Warehouse running somewhere to get OpenScale to work.  If you have one handy, you can use this otherwise we will configure it in the next section.
***Coming soon***
  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)


## Db2 Warehouse
### Install Db2 Warehouse (SMP)
1. The first thing you will want to do is to pick one node that will house Db2 Warehouse and add a label.
1. Run: `oc get nodes`   This will produce a list of nodes.
1. Run `oc describe node |  grep -eHostname: -e 'cpu ' -e 'memory '` Review the output and find the node with the least amount of resources allocated   You will be setting a ***label*** to create **Node Affinity**.  **Db2 Warehouse** pods will be installed to this particular node.  Database will also be provisioned to this node.   Sometimes you may need to resize your OpenShift worker pool size to increase capacity. Base on review I will pick node `10.95.7.49` as it has the least amount of resources allocated right now.
   ~~~
   Toms-MBP:bin tjm$ oc describe node |  grep -eHostname: -e 'cpu ' -e 'memory '
    MemoryPressure   False   Wed, 10 Jun 2020 16:21:28 -0400   Mon, 08 Jun 2020 10:19:26 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
    Hostname:    10.95.7.21
    cpu       14941m (94%)      63645m (400%)
    memory    44715538Ki (74%)  150095560704 (244%)
    MemoryPressure   False   Wed, 10 Jun 2020 16:21:29 -0400   Mon, 08 Jun 2020 10:19:13 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
    Hostname:    10.95.7.23
    cpu       13960m (87%)      39825m (250%)
    memory    42889746Ki (71%)  106114088960 (172%)
    MemoryPressure   False   Wed, 10 Jun 2020 16:21:32 -0400   Mon, 08 Jun 2020 10:16:15 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
    Hostname:    10.95.7.49
    cpu       10718m (67%)      33863m (213%)
    memory    24435218Ki (40%)  85963040Ki (143%)
    MemoryPressure   False   Wed, 10 Jun 2020 16:21:35 -0400   Mon, 08 Jun 2020 10:21:40 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
    Hostname:    10.95.7.6
    cpu       11970m (75%)      49570m (312%)
    memory    30833170Ki (51%)  87313121280 (142%)
   ~~~
1. Bind Db2 Warehouse and provisioned instances to a specific node, by adding a **label** of `icp4data=database-db2wh` to a node. Select this node according to known resources available.  I picked node `10.95.7.49` from the last command.
   ~~~
   oc label node <node name or IP Address> icp4data=database-db2wh
   ~~~
   **Note:**  If you resize your OpenShift cluster's worker pool to a lower number of nodes, it is possible that the node with the label for Db2 Warehouse may be deleted.   This would render any database instances unusable until you label another node and restart the `db2wh` pods, so they start on the same node.  Do not try to label 2 nodes as you will get an ***anti-affinity*** error in the `db2u-0` and unified-console pods,  They will stay in a state of `pending`.  
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
    ~~~
    export NAMESPACE=zen
    export STORAGE_CLASS=ibmc-file-gold-gid
    export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
    export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
    ~~~
1. Set the security aspects for Db2 Warehouse to install properly
  ~~~
  ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly db2wh
  ~~~
1. Deploy **Db2 Warehouse** by running the following:
  ~~~
  ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly db2wh
  ~~~
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
1. Verify the installation  
   ~~~
   ./cpd-cli status --namespace ${NAMESPACE} --assembly db2wh
   ~~~
1. Check for patches
   ~~~
   ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly db2wh
   ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
    - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

 **Note:** Actual time taken is 16 minutes
 <iframe width="560" height="315" src="https://www.youtube.com/embed/943Gi4Z9vUo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Provision a Database instance
1. Once installed and all pods are up, you can go to the service catalog page with the square with petals icon in upper right.  
1. On the services page, **Click** the left side filter to go to ***Datasources*** to get to **Db2 Warehouse** tile.  
1. **Click** the 3 vertical dots on upper left of the tile or **Click** through the tile then **Click** ***Provision Instance***.
1. On the ***Configure*** page keep defaults or adjust if you know you need more.  **Click** ***Next***.
1. On the ***Storage*** page, Select ***Create new storage***; Change **Storage Class** to ***ibmc-file-gold-gid*** ; Adjust the size to reflect the amount needed.  ***Default is 100GB***.
1. **Click** ***Next***
1. Review the settings. Here you can change the **Display name** to something more memorable. **Click** ***Create***.
1. Your instance is being created and will be accessible from the **Services > Db2 Warehouse tile.**  Also accessed from the Left menu ***My Instance*** then **Click** ***Provisioned instances***
1. From ***Provision Instances*** on the left you will see 3 horizontal dots. From this view, you can watch the steps of the provision, just incase it fails based on insufficient resources.
**Note:** You can see a red triangle that states Failed. This is temporary and is most likely the Cloud provision service waiting on a storage volume to come online and available to mount as a persistent volume to map the persistent volume claim.
1. **Click** this and see the options.
  - ***Open*** will open the DB2 Warehouse instance for use.
  - ***View details*** will provide you the details including ***user*** ; ***password*** ; ***jdbc url***
  - ***Manage Access*** let you add ***user ids*** and assign them ***Admin*** or ***User*** roles.
  - ***Delete*** This will delete this particular instance.
**Note** Total time took about 12 minutes  
<iframe width="560" height="315" src="https://www.youtube.com/embed/VLxgR3CeOCg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Create a Table  
1. When you Open the interface, you will run as the user id that provisioned it, for example **user999** which is **admin**, so any tables that are created, by ***admin***, via **SQL editor** by default will go under **Schema** ***user999***.
  - You can use the UI and select and create tables under certain schemas.

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Uninstalling DB2 Warehouse.
1. First you will want to release any storage and delete instances to make sure storage is released.   Use ***Delete*** to do this.
1. From the command line:
1. Set namespace.  My namespace is ***zen*** your may be different like ***default***
1. Run env to verify that the following variables are exported
    - OpenShift 4.x
     ~~~
     export NAMESPACE=zen
     ~~~
1. Do a dry run uninstall to check what will be taken off.
  ~~~
  ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly db2wh --uninstall-dry-run
  ~~~
1. Run the uninstall
  ~~~
  ./cpd-cli uninstall --namespace ${NAMESPACE}  --assembly db2wh
  ~~~
1.  Go to the **Services** catalog and verify that **Db2 Warehouse** is no longer ***enabled***.   

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

## Db2 Advanced
### Install Db2 Advanced
1. The first thing you will want to do is to pick one node that will house Db2 Advanced and add a label.
1. Run: `oc get nodes`   This will produce a list of nodes.
1. Run `oc describe node |  grep -eHostname: -e 'cpu ' -e 'memory '` Review the output and find the node with the least amount of resources allocated   You will be setting a ***label*** to create **Node Affinity**.  **Db2 OLTP** pods will be installed to this particular node.  Database will also be provisioned to this node.   Sometimes you may need to resize your OpenShift worker pool size to increase capacity. Base on review I will pick node `10.95.7.49` as it has the least amount of resources allocated right now.
    ~~~
    Toms-MBP:bin tjm$ oc describe node |  grep -eHostname: -e 'cpu ' -e 'memory '
     MemoryPressure   False   Wed, 10 Jun 2020 16:21:28 -0400   Mon, 08 Jun 2020 10:19:26 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
     Hostname:    10.95.7.21
     cpu       14941m (94%)      63645m (400%)
     memory    44715538Ki (74%)  150095560704 (244%)
     MemoryPressure   False   Wed, 10 Jun 2020 16:21:29 -0400   Mon, 08 Jun 2020 10:19:13 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
     Hostname:    10.95.7.23
     cpu       13960m (87%)      39825m (250%)
     memory    42889746Ki (71%)  106114088960 (172%)
     MemoryPressure   False   Wed, 10 Jun 2020 16:21:32 -0400   Mon, 08 Jun 2020 10:16:15 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
     Hostname:    10.95.7.49
     cpu       10718m (67%)      33863m (213%)
     memory    24435218Ki (40%)  85963040Ki (143%)
     MemoryPressure   False   Wed, 10 Jun 2020 16:21:35 -0400   Mon, 08 Jun 2020 10:21:40 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
     Hostname:    10.95.7.6
     cpu       11970m (75%)      49570m (312%)
     memory    30833170Ki (51%)  87313121280 (142%)
    ~~~
1. Bind Db2 Advanced and provisioned instances to a specific node, by adding a **label** of `icp4data=database-db2-oltp` to a node. Select this node according to known resources available.  I picked node `10.95.7.49` from the last command.
    ~~~
    oc label node <node name or IP Address> icp4data=database-oltp
    ~~~
1. You can dedicate one or more [worker nodes to your Db2® database service](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/svc/dbs/aese-dednodes.html#aese-dednodes).  Do this before installing the service.
1. Run env to verify that the following variables are exported
    - OpenShift 4.x
     ~~~
     export NAMESPACE=zen
     export STORAGE_CLASS=ibmc-file-gold-gid
     export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
     export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
     ~~~
1. Set the security aspects for Db2 Advanced to install properly
   ~~~
   ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly db2oltp
   ~~~
1. Deploy **Db2 Advanced** by running the following:
   ~~~
   ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly db2oltp
   ~~~
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
1. The deployed service will look like this from `oc get pods`.  The completed pod is a load job which can be deleted.
   ~~~
   db2oltp-catalog-11530-6c488d895f-jddzn   1/1       Running     0          5m11s
   db2oltp-catalog-11530-uploadjob-hcsqr    0/1       Completed   0          5m11s
   ~~~
1. Verify the installation  
   ~~~
   ./cpd-cli status --namespace ${NAMESPACE} --assembly db2oltp
   ~~~
1. Check for patches
   ~~~
   ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly db2oltp
   ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
    - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

   [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Provision an OLTP Database instance
1. Once installed and the `db2oltp-catalog-11530` pod is running or the service tile is marked as available , you can go to the service catalog page with the square with petals icon in upper right.  
1. On the services page, **Click** the left side filter to go to ***Datasources*** to get to **Db2 Advanced** tile.  
1. **Click** the 3 vertical dots on upper left of the tile or **Click** through the tile then **Click** ***Provision Instance***.
1. On the ***Configure*** page keep defaults or adjust if you know you need more.  **Click** ***Next***.  **NOTE** Make sure the ***Value for node label*** matches what was set above.
1. On the ***System Storage*** page, Change **Storage Class** to ***ibmc-file-gold-gid*** ; Adjust the size to reflect the amount needed.  ***Default is 100GB***.
1. **Click** ***Next***
1. On the ***User Storage*** page, Change **Storage Class** to ***ibmc-file-gold-gid*** ; Adjust the size to reflect the amount needed.  ***Default is 100GB***.
1. **Click** ***Next***
1. On the ***Backup Storage*** page, Change **Storage Class** to ***ibmc-file-gold-gid*** ; Adjust the size to reflect the amount needed.  ***Default is 100GB***.
1. **Click** ***Next***
1. Review the settings. Here you can change the **Display name** to something more memorable. **Click** ***Create***.
1. Your instance is being created and will be accessible from the **Services > Db2 Advanced tile.**  Also accessed from the Left menu ***My Instance*** then **Click** ***Provisioned instances***
1. From ***Provision Instances*** , you will see a list of instances.   If you left the defaults, the name will be something like ***Db2 Advanced Edition-1***.   To get details **Click** on the instance name.  In my case ***Db2 Advanced Edition-1***.
 **Note:** You can see a red triangle that states Failed. This is temporary and is most likely the Cloud provision service waiting on a storage volume to come online and available to mount as a persistent volume to map the persistent volume claim.
1. **Click** the horizontal 3 "." on the right and see the options.
   - ***Open*** This will open the UI to allow you to work on the database
   - ***Manage Access*** This lets you control high level ACLs to the database.  
   - ***Delete*** This will delete this particular instance.


   [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Create a Table  
1. When you Open the interface, you will run as the user id that provisioned it, for example **user999** which is **admin**, so any tables that are created, by ***admin***, via **SQL editor** by default will go under **Schema** ***user999***.
   - You can use the UI and select and create tables under certain schemas.

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Uninstalling DB2 Warehouse.
1. First you will want to release any storage and delete instances to make sure storage is released.   Use ***Delete*** to do this.
1. From the command line:
1. Set namespace.  My namespace is ***zen*** your may be different like ***default***
1. Run env to verify that the following variables are exported
     - OpenShift 4.x
      ~~~
      export NAMESPACE=zen
      ~~~
1. Do a dry run uninstall to check what will be taken off.
    ~~~
    ./cpd-cli uninstall --namespace ${NAMESPACE}  --assembly db2wh --uninstall-dry-run
    ~~~
1. Run the uninstall
    ~~~
    ./cpd-cli uninstall --namespace ${NAMESPACE}  --assembly db2wh
    ~~~
1.  Go to the **Services** catalog and verify that **Db2 Warehouse** is no longer ***enabled***.   

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

## Installing DataStage service
1. Verify you have enough resource capcity to run DataStage.  You many need to increase your work pool by a node.
1. Create a [ds.yaml](ds.yaml) file to override and set certain storage classes. On ROKS we will be using the default storage classes, but defining them no the less.
1. Run env to verify that the following variables are exported
   - OpenShift 4.x
    ~~~
    export NAMESPACE=zen
    export STORAGE_CLASS=ibmc-file-gold-gid
    export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
    export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
    ~~~
1. Set the security aspects for DataStage to install properly
  ~~~
  ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly ds
  ~~~
1. Deploy DataStage by running the following:
  ~~~
  ./cpd-linux --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly ds --override=ds.yaml
  ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
1. Verify the installation  
  ~~~
  ./cpd-cli status --namespace ${NAMESPACE} --assembly ds
  ~~~
1. Check for patches
  ~~~
  ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly ds
  ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
  - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Create a Transformation Project
1. From the navigator **click** ***Organize > transform data***.  This will bring you into the Data Flow Designer in a projects view.
  1. If you do not see ***Organize > transform data***, check your setting and profile in the upper right under the user icon. **Click** ***Profile and Setttings*** > ***Permissions*** then verify that you have following two permissions:
  1. Integrate and transform data
  1. View governance artifacts
    If not the users will need to have ***Data Engineer*** role.
     - Go to the Navigator on the left and scroll down to ***Administer***  **Click** on ***Manage Users***.
     - On the right of the list of users, you will see a pencil, which indicates ***edit***.  
     - **Click** the pencil for the user you desire to edit.
     - Choose ***Data Engineer*** to get the appropriate permissions.
1. From here you can create a new project.  
  - **Click** ***+ Create*** then provide a Project name.
  - **Click** ***Create*** button.  This will take a minute or two.
1. **Click** the ***Clog*** and there will be 4 options.
  - **Adjust Compute Nodes** //This allows you to scale nodes for a parallel job.
  - **Configuration** //Allows you to tweak service configurations
  - **Multi-Cloud Kafka brokers** (might be grayed out)
  - **Server** //Connections to Github and whether to use Machine Learning (On by default)
1. **Click** on ***dstage1*** to see how this works.
  1. **Click** on ***+ Create***, you should see a ***Parallel job*** and ***Sequence job***
  1. **Click** ether job and a palette of nodes are available to you to build your job.  These nodes are specific to the type of job.
  1. Across the top, you should see another cog.  This allows you to add ***Smart Palatte*** to the options.  This will start prompting you with potential options.
  1.  Across the very top, is where you will add ***Connections***, ***Table Definitions***, ***Parameter Sets***, ***Jobs*** and the current ***Job_1*** that you were just creating.  

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Uninstalling DataStage.
1. First you will want to release any storage and delete instances to make sure storage is released.   Use ***Delete*** to do this.
1. From the command line:
1. Set namespace.  My namespace is ***zen*** your may be different like ***default***
1. Run env to verify that the following variables are exported
    - OpenShift 4.x
     ~~~
     export NAMESPACE=zen
     ~~~
1.  Do a dry run uninstall to check what will be taken off.
   ~~~
   ./cpd-cli uninstall --namespace ${NAMESPACE}  --assembly ds --uninstall-dry-run
   ~~~
1. Run the uninstall
   ~~~
   ./cpd-cli uninstall --namespace ${NAMESPACE}  --assembly ds
   ~~~
1. Go to the **Services** catalog and verify that **DataStage** is no longer ***enabled***.   

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

## Installing Analytics Dashboards
**Analytics Dashboards** gives you a quick way to visualize your data.  This is not full blown **Cognos Analytics for Cloud Pak for Data*** nor **on premises** version.  Understand the [current differences here](https://community.ibm.com/community/user/businessanalytics/blogs/david-cushing/2018/12/13/1)
1. Run env to verify that the following variables are exported
   - OpenShift 4.x
    ~~~
    export NAMESPACE=zen
    export STORAGE_CLASS=ibmc-file-gold-gid
    export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
    export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
    ~~~
1. Set the security aspects for Analytics Dashboards to install properly
  ~~~
  ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly cde
  ~~~
1. Deploy Analytics Dashboards by running the following:
  ~~~
  ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly cde
  ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.
1. Verify the installation  
  ~~~
  ./cpd-cli status --namespace ${NAMESPACE} --assembly cde
  ~~~
1. Check for patches
  ~~~
  ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly cde
  ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
  - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Uninstalling Analytics Dashboards
1. From the command line:
1. Set namespace.  My namespace is ***zen*** your may be different like ***default***
1. Run env to verify that the following variables are exported
    - OpenShift 4.x
     ~~~
     export NAMESPACE=zen
     ~~~
1. Do a dry run uninstall to check what will be taken off.
   ~~~
   ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly cde --uninstall-dry-run
   ~~~
1. Run the uninstall
   ~~~
   ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly cde
   ~~~
1.  Go to the **Services** catalog and verify that **Analytics Dashboards** is no longer ***enabled***.   

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)


## Installing Analytics Engine (Spark Clusters)
If you are using **Data Refinery** and you have to prep files larger than 100MB, then you will want to install the **Analytics Engine** to be able to select a different runtime than ***Data Refinery XS***.  This is the service to ***install*** if you want to use **Spark** with **Watson Studio**.
1. Run env to verify that the following variables are exported
   - OpenShift 4.x
    ~~~
    export NAMESPACE=zen
    export STORAGE_CLASS=ibmc-file-gold-gid
    export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
    export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
    ~~~
1. Set the security aspects for Analytics Engine to install properly
  ~~~
  ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly spark
  ~~~
1. You will need an when installing the Spark assembly on a OCP 4.3 cluster. This is not needed for 3.11.  Use this [spark-ocp43-override.yaml](spark-ocp43-override.yaml)
1. Deploy Analytics Engine by running the following:
  ~~~
  ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly spark --override spark-ocp43-override.yaml
  ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
1. Verify the installation  
  ~~~
   ./cpd-cli status --namespace ${NAMESPACE} --assembly spark
  ~~~
1. Check for patches
  ~~~
  ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly spark
  ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:    
  - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Provision Analytics Engine instance
***coming soon***
  [Provision the instance](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/wsj/spark/spark-provision.html)

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Resource constraints  
***coming soon***
  [Governing the about of resources in an instance](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/wsj/spark/spark-resource-quota.html)

### Uninstalling Analytics Engine
1. From the command line:
1. Set namespace.  My namespace is ***zen*** your may be different like ***default***
1. Run env to verify that the following variables are exported
     - OpenShift 4.x
      ~~~
      export NAMESPACE=zen
      ~~~
1. Do a dry run uninstall to check what will be taken off.
    ~~~
    ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly spark --uninstall-dry-run
    ~~~
1. Run the uninstall
    ~~~
    ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly spark
    ~~~
1.  Go to the **Services** catalog and verify that **Analytics Engine** is no longer ***enabled***.   

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)  

##Cognos Analytics  
### Installing Cognos Analytics
Understand the [current differences here](https://community.ibm.com/community/user/businessanalytics/blogs/david-cushing/2018/12/13/1)  
1. Run env to verify that the following variables are exported.
   - OpenShift 4.x
    ~~~
    export NAMESPACE=zen
    export STORAGE_CLASS=ibmc-file-gold-gid
    export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
    export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
    ~~~
1. Set the security aspects for Cognos to install properly
   ~~~
   ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly ca
   ~~~
1. Deploy Cognos Analytics by running the following:
   ~~~
   ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --latest-dependency --assembly ca
   ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
1. Verify the installation  
   ~~~
  ./cpd-cli status --namespace ${NAMESPACE} --assembly ca
   ~~~
1. Check for patches
  ~~~
  ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly ca
  ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
   - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)
 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)

### Provision Cognos Analytics instance
1. Preparing Db2 Advanced Database to make a connection that **Cognos Analytics** can used
1. Collect the details of the provisioned database. This will be used for the environment variables and when creating the global connection.
  ~~~
  Deployment id db2oltp-xxxxxxxxxxxxx
  Username userxxx
  Password xxxxxxxxxxxxxxxxxxxxxx
  JDBC URLjdbc:db2://HOST:PORT/BLUDB
  ~~~
**NOTE:** If you do not run through the following steps to prepare the Database, your provision will fail.   
1. From a terminal window. Run the following to set up the environment to execute the Database updates.
  ~~~
  export NAMESPACE=zen
  export DB2USERNAME=<USERNAME>
  export DB2PASSWORD=<PASSWORD>
  export DB2DEPLOYMENTID=<DEPLOYMENTID>
  ~~~
1. Connect to the DB2 pods
  ~~~
  export CMD="env DB2USERNAME=$DB2USERNAME DB2PASSWORD=$DB2PASSWORD  bash"
  oc  exec -ti $DB2DEPLOYMENTID-db2u-0 -n $NAMESPACE -- $CMD
  ~~~   
1.  Run the following command to update the data base values from within the pod.
  ~~~
  cd /mnt/blumeta0/home/db2inst1/sqllib
  . ./db2profile   
  ~~~  
  ~~~
  db2 CONNECT to BLUDB user $DB2USERNAME using  $DB2PASSWORD;
  db2 UPDATE DATABASE CONFIGURATION USING APPLHEAPSZ 1024 DEFERRED;
  db2 UPDATE DATABASE CONFIGURATION USING LOCKTIMEOUT 240 DEFERRED;
  db2 CREATE BUFFERPOOL CMDB_08KBP IMMEDIATE SIZE 1000 PAGESIZE 8K;
  db2 CREATE BUFFERPOOL CMDB_32KBP IMMEDIATE SIZE 1000 PAGESIZE 32K;
  db2 CREATE SYSTEM TEMPORARY TABLESPACE TSN_SYS_CMDB IN DATABASE PARTITION GROUP IBMTEMPGROUP PAGESIZE 32K BUFFERPOOL CMDB_32KBP;
  db2 CREATE USER TEMPORARY TABLESPACE TSN_USR_CMDB IN DATABASE PARTITION GROUP IBMDEFAULTGROUP PAGESIZE 8K BUFFERPOOL CMDB_08KBP;
  db2 CREATE REGULAR TABLESPACE TSN_REG_CMDB IN DATABASE PARTITION GROUP IBMDEFAULTGROUP PAGESIZE 8K BUFFERPOOL CMDB_08KBP;
  db2 DROP TABLESPACE USERSPACE1;
  db2 CREATE SCHEMA db2COGNOS AUTHORIZATION $DB2USERNAME;
  db2 ALTER BUFFERPOOL ibmdefaultbp size 49800;
  ~~~
1. Create a connection that will be used for Cognos Analytics.   
  1. From the main menu with 3 horizontal bars, click on ***Connections***.
  1. Provide a **Name**, I used ***Cognosdb***
  1. Provide a **Description** (optional)
  1. Select a **Connection Type**, Select ***Db2***.  This will expand to give more options.
  1. You will focus on the following.  You collected this at the top.
    - **Host** - This comes from the JDBC URL.  It is the **IP address**
    - **Port** - This comes from the JDBC URL.  It is the **number** after the ***IP:***
    - **Database** - This comes from the JDBC URL.  It is the **number** after the ***IP:PORT/***  It is generally ***BLUDB***
    - **Username** - Probably ***user999***
    - **Password** - Same as collected above.
  1. Click the ***Test connection*** button to verify that it works properly.
  1. Click ***Create***  
1. From the **Services** page, scroll to ***Cognos Analytic*** and **click** the tile  
1. in the upper right there is a button to **Provision Instance**



 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)   

### Uninstalling Cognos Analytics
1. From the command line:
1. Set namespace.  My namespace is ***zen*** your may be different like ***default***
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
  ~~~
  export NAMESPACE=zen
  ~~~
1. Do a dry run uninstall to check what will be taken off.
  ~~~
  ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly ca --uninstall-dry-run
  ~~~
1. Run the uninstall
  ~~~
  ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly ca
  ~~~
1.  Go to the **Services** catalog and verify that **Cognos Analytics** is no longer ***enabled***.   

   [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)


## Watson Knowledge Catalog
### Installing Watson Knowledge Catalog service
1. Because of all the Kernel changes, I generally suggest installing Watson Knowledge Catalog using the template during the control plane installation.
1. Verify you have enough resource capacity to run Watson Knowledge Catalog.  You many need to increase your work pool by a node.
1. Create a secret in the kube-system project or namespace for the norootsquash.yaml will execute properly.  **Note: **All norootsquash pods in Kube-system namespace will restart using the new parameters.
  - `oc project kube-system`
  -  Get the apikey from the repo.yaml file and use your accounts email address.
  - `oc create secret docker-registry cpregistrysecret --docker-username=cp --docker-password= API KEY used in repo.yaml  --docker-server=cp.icr.io/cp/cpd --docker-email= your email address`
1. Adjust the kernel parameters.  Save the files to the directory with `cpd-<os>` command prior to executing or running the command.
  - Run ***oc create -f [wkc-tune-43.yaml](wkc-tune-43.yaml)*** command to tune the kernel parameters.  
  - Run ***oc apply -f [setkernelparameters.yaml](setkernelparameters.yaml) -n kube-system***
  - Run ***oc apply -f [norootsquash.yaml](norootsquash.yaml) -n kube-system***
1. Run env to verify that the following variables are exported
 - OpenShift 4.x
  ~~~
  export NAMESPACE=zen
  export STORAGE_CLASS=ibmc-file-gold-gid
  export DOCKER_REGISTRY_PREFIX=$(oc get routes image-registry -n openshift-image-registry -o template=\{\{.spec.host\}\})
  export LOCAL_REGISTRY=image-registry.openshift-image-registry.svc:5000
  ~~~
1. Set the security aspects for Watson Knowledge Catalog to install properly
   ~~~
   ./cpd-cli adm --repo ./repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly wkc
   ~~~
1. Deploy Watson Knowledge Catalog by running the following: Download the [override-nginx.yaml](override-nginx.yaml) file for use in the install command.
   ~~~
   ./cpd-cli install --repo ./repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix ${LOCAL_REGISTRY}/${NAMESPACE} --insecure-skip-tls-verify --override override-nginx.yaml --assembly wkc
   ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
1. Verify the installation  
    ~~~
    ./cpd-cli status --namespace ${NAMESPACE} --assembly wkc
    ~~~
1. Check for patches
    ~~~
    ./cpd-cli status  --repo ./repo.yaml --namespace ${NAMESPACE} --patches --available-updates --assembly wkc
    ~~~
1. If there are patches  apply the highest number as it will be cumulative.  Some patches have prerequisite patches because they have dependencies on another service or on a set of shared, common services. If the patch details list one or more prerequisite patches, you must install the prerequisite patches before you install the service patch. You can run the following command to determine whether any of the prerequisite patches are already installed on the cluster:
      - [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)


### Post install tasks  (To Do's for Tom to validate)
1. [Setting up your default catalog](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/wsj/catalog/set-up-first.html)
1. [Optional tasks](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/wsj/install/postinstall-wkc.html)

### Uninstalling Watson Knowledge Catalog
1. From the command line:
1. Set namespace.  My namespace is ***zen*** your may be different like ***default***
1. Run env to verify that the following variables are exported
  - OpenShift 4.x
   ~~~
   export NAMESPACE=zen    
  ~~~  
1. Do a dry run uninstall to check what will be taken off.
   ~~~
   ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly wkc --uninstall-dry-run
   ~~~
1. Run the uninstall
   ~~~
   ./cpd-cli uninstall --namespace ${NAMESPACE} --assembly wkc
   ~~~

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data35.html)
