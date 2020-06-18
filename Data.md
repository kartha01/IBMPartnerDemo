# Install instructions for Cloud Pak for Data
- [Provision the Control plane or lite assembly](#provision-the-control-plane-or-lite-assembly)
- [Building the configuration](#building-the-configuration)
- [Review Services installed](#review-services-installed)
- [Set up Data Virtualization](#set-up-data-virtualization)
- [Set up Watson Studio or Watson Machine Learning](#set-up-watson-studio-or-watson-machine-learning)
  * [Increase the capacity or scale up your service](#increase-the-capacity-or-scale-up-your-service)
- [Set up Watson OpenScale](#set-up-watson-openscale)
- [Adding additional services](#adding-additional-services)
  * [To enable or disable the default admin users](#to-enable-or-disable-the-default-admin-users)
  * [How can I patch a service or control plane](#how-can-i-patch-a-service-or-control-plane)
- [Install Db2 Warehouse (SMP)](#install-db2-warehouse-smp)
  * [Provision a Database instance](#provision-a-database-instance)
  * [Create a Table](#create-a-table)
  * [Uninstalling DB2 Warehouse.](#uninstalling-db2-warehouse)
- [Installing DataStage service](#installing-datastage-service)
  * [Create a Transformation Project](#create-a-transformation-project)
  * [Uninstalling DataStage.](#uninstalling-datastage)
- [Installing Analytics Dashboards](#installing-analytics-dashboards)
  * [Uninstalling Analytics Dashboards](#uninstalling-analytics-dashboards)
- [Installing Analytics Engine (Spark Clusters)](#installing-analytics-engine-spark-clusters)
  * [Provision Analytics Engine instance](#provision-analytics-engine-instance)
- [Installing Cognos Analytics](#installing-cognos-analytics)
  * [Provision Cognos Analytics instance](#provision-cognos-analytics-instance)

## Provision the Control plane or lite assembly
1. **Click** ***Catalog***
1. **Click** ***Software***
1. **Filter** on ***Analytics*** or In the search box **enter** ***Cloud Pak*** and press **enter**.  This should list the available Cloud Paks that can be provisioned. In these instructions, you are going to install ***Cloud Pak for Data***.
1. **Click** on the ***Cloud Pak for Data*** tile.
1. **Click** on the ***Readme*** tab to review what you can provision using ***Schematics*** (Terraform).

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Building the configuration
1. On the ***create*** tab, this is where we will build the configuration of the Cloud Pak for Data clusters
1. First section describes the bare minimum configuration.  For Cloud Pak for Data it is as follows.  `Each cluster must meet a set of minimum requirements: 3 nodes with 16 cores, 64GB memory, and 25GB disk per node.`  As I know I want to add Db2 Warehouse and provision a Data Virtualization instance, I will start with 4 node OpenShift cluster.  As I add more features, I am needing more capacity when I provision the instance.
1. Select the cluster that was previously created.  Where you created project `zen`.
1. Select the project.   While you can pick `default`, you would be better served picking the previously create project like `zen`.
1. Rename the workspace to something you will remember like `CPDDemo-HealthCare` for a health care demo.
1. Pick your resource group, if you did not create one, just use `Default`.
1. You can tag your environment for easy search.  This is not required.
1. Scroll down and select ***Run Script*** if you do not have permissions, you can share a link and have an administrator execute this.
1. Scroll down and select the services that you desire to add to Cloud Pak for Data. **Note:** This is a first time set up action only. Currently, Once provisioned, toggling true then false will lead to no further action.  You will need to uninstall or install again, which will be shown later.
1. On the right, you should see your entitlements and a yellow box, which will turn green after you run the script below.
1. **Check the box** and to ***accept the license***.   

**Note:** This can take 4 hours to provision.
<iframe width="600" height="322" src="https://www.youtube.com/embed/Ic0xnlci47o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Review Services installed
1. From the Schematics workspace, **Click** `Offering dashboard`
1. Depending on your browser, you will need to accept the certificate.  General path is Advanced then accept risk.  This will bring you to the login page. Bookmark this for future usage.
1. **Login** using default credentials as seen in video.
1. Along the black menu bar at the top the icon to the right on the magnifying glass is `Services`.  **Click** on this link.  
1. Take a look at the services that are `enabled`.   These have been installed or deployed.   You should see 5 services.  
  - **AI** (Watson Studio, Watson Machine Learning, Watson OpenScale)
  - **Data Governance** (Watson Knowledge Catalog)
  - **Data Sources** (Data Virtualization).  
1. Some are set up automatically and some you need to provision.
<iframe width="560" height="315" src="https://www.youtube.com/embed/vYS7xwk0fn8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>  

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Set up Data Virtualization
1. **Click** on the ***Services*** icon to get to the services catalog.
1. Go to the Data Virtualization tile, you see to "Provision Instance".
1. **Click** on ***Provision Instance***.
1. Then **Click** ***Configure***. **Note:** for this Cloud Pak for Data on ROKS, the semaphores are already configured, so make sure to ***uncheck*** the box to the left of `Set up semaphore parameters for Data Virtualization automatically`, otherwise the provision can fail.
1. **Click** ***Configure***.
1. Select the number of nodes and cores to allocate.  For a demo system defaults are enough.  **Note:** Depending on how many node you have in your worker pool, this could fail provisioning with a not enough resource message.  If you configure a default 3 16/64 workers and install all of the services, this message can appear.  Resizing your worker pool to 4 will get you around this.
1. Make sure your change the storage classes from `default` to `ibmc-file-gold-gid`.  Choosing default could cause a failure.
1. Once provisioned, set some users to have ***Data Engineer*** role.
   - Go to the Navigator on the left and scroll down to ***Administer***  **Click** on ***Manage Users***.
   - On the right of the list of users, you will see a pencil, which indicates ***edit***.  **Click** the pencil for the user you desire to edit. If you pick ***admin***, you will see this is a new role added based on the Data Virtualization service.  
1. Go to ***My Instance***   
1. Once complete, you will see **Data Virtualization** under the **Collect** area on the left table of contents.  From here you can add data sources or pointers to file folders.
**Note:** I have not yet tested this with ***Remote Data Connections*** aka  local file folders.

**Note:**  The total provision and assigning roles was 21 minutes.
<iframe width="600" height="322" src="https://www.youtube.com/embed/ll40JJx5xyc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Set up Watson Studio or Watson Machine Learning
1. If you pick Watson Studio or Watson Machine Learning tiles, there will be no actions to take. You can proceed and create a project.

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Increase the capacity or scale up your service
1. To scale services, you will need to have the command line installed.  (Instruction under the ***Adding additional services*** section)
1. You can scale **up** the services by executing these commands for either **wsl** or **wml**.  **Note:** There is currently no scale down function.
1. Run env to verify that the following variables are exported
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template=\{\{.spec.host\}\})
   ~~~
1. Scale up **Watson Studio** or **Watson Machine Learning** by running the following (config sizes are small, medium and large):
   ~~~
   ./cpd-linux scale -a wml -n zen --config medium  --load-from ./cpd-linux-workspace
   ~~~
1. Scale down **Watson Studio** or **Watson Machine Learning** by running the following (config sizes are small, medium and large):
   ~~~
   ./cpd-linux scale -a wml -n zen --config small  --load-from ./cpd-linux-workspace
   ~~~

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Set up Watson OpenScale
1. For Watson OpenScale Service, there is an `Open` button which launch a UI to help configure the initial information.
1. Let's provision OpenScale.
1. You will be met with an ***auto setup*** to configure.  
1. First step is accept the local WML instance.
1. Enter connection details for a Db2 Database.  I am using a DB2 Warehouse that I installed later on.  You need to have Db2/Db2 Warehouse running somewhere to get OpenScale to work.  If you have one handy, you can use this otherwise we will configure it in the next section.

**Note:** Actual time taken is 18 minutes which included reviewing the sample.
<iframe width="560" height="315" src="https://www.youtube.com/embed/dJV_ZqK-pvc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Adding additional services
1. You have already set up the client environment.  If not go have to the [first page](README.md) and execute these step under **Installing the client environment**  You will have logged into the OpenShift cluster, set the project to ***zen*** already built the encrypted route to the internal container repository.  This is where all of your containers and helm charts will be stored.  If ***zen*** doesn't exist, then your probably went with ***default***.
1. How do I get the installer? Good question.  You work for an ***IBM Business Partner*** with ***Software Access Catalog subscription***.  Your **IBM Cloud ID** is listed in your companies PartnerWorld Profile.  This provides access to [Software Access Catalog](https://www.ibm.com/partnerworld/program/benefits/software-access-catalog)  
  **Note:** There is not a Windows install and the `.bin` file in PartnerWorld Software Access Catalog doesn't download properly on Mac.  Make sure you have access to a Linux system to get the `.tar.gz` file.
1. Access the catalog
1. **Log in** using your ***IBM ID*** (linked to partnerWorld profile).
1. Scroll to the bottom and **Click** ***I Agree***
1. **Click** in the edit box under **Find by Part Number**.  Enter `CC3Y1ML`.  **Click** ***Search***.
1. ***Download Director*** will be pre selected, change to ***HTTP***.
1. **Click** the radio button for ***IBM Cloud Pak for Data Enterprise Edition V2.5 - Installer Linux x86 Multilingual (CC3Y1ML)***
1. **Click** the ***I agree*** radio button and **Click** ***Download Now***
1. Locate the `CP4D_EE_Installer_V2.5.bin` file. (probably in the downloads directory)
1. Move to a Linux system and execute:
~~~
chmod +x CP4D_EE_Installer_V2.5.bin
~~~
1. Exceute the command, which will download the installer for linux and Mac (darwin) in a tar file `cloudpak4data-ee-v2.5.0.0.tar.gz`
~~~
./CP4D_EE_Installer_V2.5.bin
~~~
1. Extract the tar file.
~~~
tar -xvf cloudpak4data-ee-v2.5.0.0
~~~
1. Find a file called `repo.yaml`  This is a simple file, but key. Here is where you will add the `apikey` to get access to the container registry for the additional Services.   
1. In a web browser, **Login** and access [the entitlement registry](https://myibm.ibm.com/products-services/containerlibrary).  **Click** ***Get entitlement key*** on the left side above Library.  Either ***Generate a key*** or ***Copy the existing key***.  This will populate your scratchpad where you can paste it into the `repo.yaml` file.
1.  **Paste** the ***apikey*** into the Yaml file to the right of the `apikey:`  After you paste, it should something like this only longer.
~~~
registry:
  - url: cp.icr.
    username: cp
    apikey: eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJ
    name: base-registry
fileservers:
 - url: https://raw.
~~~~
**Note:**  While the documentation denotes Linux, using **cpd-linux**, the video uses **cdp-darwin** for Mac.  Currently, there is no Windows interface.

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

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

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### How can I patch a service or control plane
From time to time any software needs a patch for security reasons, new feature or a bug fix.  How do you know that there is a patch for a specifica service, common services or the control plane.   Take a [look here for v2.5]( https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/patch/avail-patches.html).  This document has a link to each of the service patches and any extra work that might be needed.   Most patches need a prerequisite patch for the common services.
1. These commands will start to look familiar.  Set up the environment variables that you will use in the commands, this will lead to less errors later on.
~~~
export NAMESPACE=zen
export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template=\{\{.spec.host\}\})
~~~
1. Run the command to patch the common services. Notice that the command is `patch`, `patch-name` is ***cpd-2.5.0.0-ccs-patch-6***.  This name will change after this writing. Note the assembly name can be `wkc` or `wsl`, why not `lite` for the control plane, I am not sure as of this writing.  
~~~
./cpd-linux patch --repo ../repo.yaml  --namespace ${NAMESPACE}  --transfer-image-to ${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --cluster-pull-prefix docker-registry.default.svc:5000/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --insecure-skip-tls-verify  --assembly wkc  --patch-name cpd-2.5.0.0-ccs-patch-6
~~~
1. Verify that the patch has been applied.
~~~
Toms-MBP:~ tjm$ oc project zen
Already on project "zen" on server "https://c106-e.us-south.containers.cloud.ibm.com:31432".
Toms-MBP:~ tjm$ oc describe cpdinstall cr-cpdinstall | grep "Patch Name:" | sort | uniq | cut -d: -f2
       cpd-2.5.0.0-ccs-patch-6
~~~
1. you can repeat this pattern, replacing the values to the right of **assembly**  and **patch-name**

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Install Db2 Warehouse (SMP)
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
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template=\{\{.spec.host\}\})
   ~~~
1. Set the security aspects for Db2 Warehouse to install properly
  ~~~
  ./cpd-linux adm --repo ../repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly db2wh`
  ~~~
1. Deploy **Db2 Warehouse** by running the following:
  ~~~
  ./cpd-linux --repo ../repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix docker-registry.default.svc:5000/${NAMESPACE} --insecure-skip-tls-verify --assembly db2wh
  ~~~
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
 **Note:** Actual time taken is 16 minutes
 <iframe width="560" height="315" src="https://www.youtube.com/embed/943Gi4Z9vUo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

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

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Create a Table  
1. When you Open the interface, you will run as the user id that provisioned it, for example **user999** which is **admin**, so any tables that are created, by ***admin***, via **SQL editor** by default will go under **Schema** ***user999***.
  - You can use the UI and select and create tables under certain schemas.

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Uninstalling DB2 Warehouse.
1. First you will want to release any storage and delete instances to make sure storage is released.   Use ***Delete*** to do this.
1. From the command line:
  - Set namespace.  My namespace is ***zen*** your may be different like ***default***
  ~~~
  export NAMESPACE=zen
  ~~~  
  - Do a dry run uninstall to check what will be taken off.
  ~~~
  ./cpd-linux uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly db2wh --uninstall-dry-run
  ~~~
  - Run the uninstall
  ~~~
  ./cpd-linux uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly db2wh
  ~~~
1.  Go to the **Services** catalog and verify that **Db2 Warehouse** is no longer ***enabled***.   

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Installing DataStage service
1. Verify you have enough resource capcity to run DataStage.  You many need to increase your work pool by a node.
1. Create a [ds.yaml](ds.yaml) file to override and set certain storage classes. On ROKS we will be using the default storage classes, but defining them no the less.
1. Run `env` to verify that the following variables are exported
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template=\{\{.spec.host\}\})
   ~~~
1. Set the security aspects for DataStage to install properly
  ~~~
  ./cpd-linux adm --repo ../repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly ds
  ~~~
1. Deploy DataStage by running the following:
  ~~~
  ./cpd-linux --repo ../repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix docker-registry.default.svc:5000/${NAMESPACE} --insecure-skip-tls-verify --assembly ds --override=ds.yaml
  ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  
**Note:** Actual time taken is 11 minutes for the install.
<iframe width="600" height="322" src="https://www.youtube.com/embed/HhBzRVBBanQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Create a Transformation Project
1. From the navigator **click** ***Organize > transform data***.  This will bring you into the Data Flow Designer in a projects view.
  1. If you do not see ***Organize > transform data***, check your setting and profile in the upper right under the user icon. **Click** ***Profile and Setttings*** > ***Permissions*** then verify that you have following two permissions:
  - Integrate and transform data
  - View governance artifacts
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

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Uninstalling DataStage.
1. First you will want to release any storage and delete instances to make sure storage is released.   Use ***Delete*** to do this.
1. From the command line:
  - Set **namespace**.  My namespace is ***zen*** your may be different like ***default***
  ~~~
  export NAMESPACE=zen
  ~~~  
  - Do a dry run uninstall to check what will be taken off.
  ~~~
  ./cpd-linux uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly ds --uninstall-dry-run
  ~~~
  - Run the uninstall
  ~~~
  ./cpd-linux uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly ds
  ~~~
1. Go to the **Services** catalog and verify that **DataStage** is no longer ***enabled***.   

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Installing Analytics Dashboards
**Analytics Dashboards** gives you a quick way to visualize your data.  This is not full blown **Cognos Analytics for Cloud Pak for Data*** nor **on premises** version.  Understand the [current differences here](https://community.ibm.com/community/user/businessanalytics/blogs/david-cushing/2018/12/13/1)
1. Run `env` to verify that the following variables are exported
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template=\{\{.spec.host\}\})
   ~~~
1. Set the security aspects for Analytics Dashboards to install properly
  ~~~
  ./cpd-linux adm --repo ../repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly cde
  ~~~
1. Deploy Analytics Dashboards by running the following:
  ~~~
  ./cpd-linux --repo ../repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix docker-registry.default.svc:5000/${NAMESPACE} --insecure-skip-tls-verify --assembly cde
  ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.

**Note:** Actual time taken is 13 minutes.  
 <iframe width="560" height="315" src="https://www.youtube.com/embed/cs8-FbiYGM8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Uninstalling Analytics Dashboards
1. From the command line:
  - Set namespace.  My namespace is ***zen*** your may be different like ***default***
  ~~~
  export NAMESPACE=zen
  ~~~  
  - Do a dry run uninstall to check what will be taken off.
  ~~~
  ./cpd-linux uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly cde --uninstall-dry-run
  ~~~
  - Run the uninstall
  ~~~
  ./cpd-linux uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly cde
  ~~~
1.  Go to the **Services** catalog and verify that **Analytics Dashboards** is no longer ***enabled***.   

 [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)


## Installing Analytics Engine (Spark Clusters)
If you are using **Data Refinery** and you have to prep files larger than 100MB, then you will want to install the **Analytics Engine** to be able to select a different runtime than ***Data Refinery XS***.  This is the service to ***install*** if you want to use **Spark** with **Watson Studio**.
1. Run `env` to verify that the following variables are exported
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template=\{\{.spec.host\}\})
   ~~~
1. Set the security aspects for Analytics Engine to install properly
  ~~~
  ./cpd-linux adm --repo ../repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly spark
  ~~~
1. Deploy Analytics Engine by running the following:
  ~~~
  ./cpd-linux --repo ../repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix docker-registry.default.svc:5000/${NAMESPACE} --insecure-skip-tls-verify --assembly spark
  ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  

    [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Provision Analytics Engine instance
***coming soon***

    [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

## Installing Cognos Analytics
Understand the [current differences here](https://community.ibm.com/community/user/businessanalytics/blogs/david-cushing/2018/12/13/1)  
1. Run `env` to verify that the following variables are exported
   ~~~
   export NAMESPACE=zen
   export STORAGE_CLASS=ibmc-file-gold-gid
   export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template=\{\{.spec.host\}\})
   ~~~
1. Set the security aspects for Cognos to install properly
   ~~~
   ./cpd-linux adm --repo ../repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly ca
   ~~~
1. Deploy Cognos Analytics by running the following:
   ~~~
   ./cpd-linux --repo ../repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix docker-registry.default.svc:5000/${NAMESPACE} --insecure-skip-tls-verify --assembly ca
   ~~~
1. You will need to tab to accept the license.
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  

    [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)

### Provision Cognos Analytics instance

***coming soon***  

    [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo/Data.html)    
