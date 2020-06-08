# Install instructions for Cloud Pak for Data

## Provision the Control plane or lite assembly
1. **Click** *Catalog*
1. **Click** *Software*
1. **Filter** on *Analytics* or In the search box **enter** *Cloud Pak* and press **enter**.  This should list the available Cloud Paks that can be provisioned. In these instructions, you are going to install ***Cloud Pak for Data***.
1. **Click** on the *Cloud Pak for Data* tile.
1. **Click** on the *Readme* tab to review what you can provision using ***Schematics*** (Terraform).

## Building the configuration
1. On the *create* tab, this is where we will build the configuration of the Cloud Pak for Data clusters
1. First section describes the bare minimum configuration.  For Cloud Pak for Data it is as follows.  `Each cluster must meet a set of minimum requirements: 3 nodes with 16 cores, 64GB memory, and 25GB disk per node.`  As I know I want to add Db2 Warehouse and provision a Data Virtualization instance, I will start with 4 node OpenShift cluster.
1. Select the cluster that was previously created.  Where you created project `zen`.
1. Select the project.   While you can pick `default`, you would be better served picking the previously create project like `zen`.
1. Rename the workspace to something you will remember like `CPDDemo-HealthCare` for a health care demo.
1. Pick your resource group, if you did not create one, just use `Default`.
1. You can tag your environment for easy search.  This is not required.
1. Scroll down and select ***Run Script*** if you do not have permissions, you can share a link and have an administrator execute this.
1. Scroll down and select the services that you desire to add to Cloud Pak for Data. **Note:** ***This is a first time set up action only. Currently, Once provisioned, toggling true then false will lead to no further action.  You will need to uninstall or install again, which will be shown later.***
1. On the right, you should see your entitlements and a yellow box, which will turn green after you run the script below.
1. **Check the box** and to *accept the license*.   
1.  This can take 4 hours to provision.

<iframe width="600" height="322" src="https://www.youtube.com/embed/Ic0xnlci47o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

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

## Set up Data Virtualization
1. **Click** on the *Services* icon to get to the services catalog.
1. Go to the Data Virtualization tile, you see to "Provision Instance".
1. **Click** on "Provision Instance".
1. Then **Click** *Configure*. **Note:** for this Cloud Pak for Data on ROKS, the semaphores are already configured, so make sure to uncheck the box to the left of `Set up semaphore parameters for Data Virtualization automatically`, otherwise the provision can fail.
1. **Click** *Configure*.
1. Select the number of nodes and cores to allocate.  For a demo system defaults are enough.  ***Note:*** Depending on how many node you have in your worker pool, this could fail provisioning with a not enough resource message.  If you configure a default 3 16/64 workers and install all of the services, this message can appear.  Resizing your worker pool to 4 will get you around this.
1. Make sure your change the storage classes from `default` to `ibmc-file-gold-gid`.  Choosing default could cause a failure.
1. Once complete, you will see Data Virtualization under the Collect area on the left table of contents.  From here you can add data sources or pointers to file folders.
**Note:** I have not yet tested this with Remote Data Connections aka  local file folders.

## Set up Watson Studio or Watson Machine Learning
1.  If you pick Watson Studio or Watson Machine Learning tiles, there will be no actions to take. You can proceed and create a project.

## Set up Watson OpenScale
1. For Watson OpenScale Service, there is an `Open` button which launch a UI to help configure the initial information.
1. Let's provision OpenScale.
1. You will be met with an auto configure.  
1. First step is to enter connection details for a Db2 Database, however one has not yet been Deployed.  You need to have Db2/Db2 Warehouse running somewhere to get OpenScale to work.  If you have one handy, you can use this otherwise we will configure it in the next section.
1. **Create** a *Watson Machine Learning* instance.

## adding additional services.
1. You have already set up the client environment.  If not go have to the [first page](README.md) and execute these step under `Installing the client environment`  You will have logged into the OpenShift cluster, set the project to `zen` already built the encrypted route to the internal container repository.  This is where all of your containers and helm charts will be stored.  If `zen` doesn't exist, then your probably went with `default`.

1. How do I get the installer? Good question.  You work for an ***IBM Business Partner*** with ***Software Access Catalog subscription***.  Your **IBM Cloud ID** is listed in your companies PartnerWorld Profile.  This provides access to [Software Access Catalog](https://www.ibm.com/partnerworld/program/benefits/software-access-catalog)  **Note:** There is not a Windows install and the `.bin` file in PartnerWorld Software Access Catalog doesn't download properly on Mac.  Make sure you have access to a Linux system to get the `.tar.gz` file.
1. Access the catalog
1. **Log in** using your *IBM ID* (linked to partnerWorld profile).
1. Scroll to the bottom and **Click** *I Agree*
1. **Click** in the edit box under "Find by Part Number".  Enter `CC3Y1ML`.  **Click** Search.
1. *Download Director* will be pre selected, change to *HTTP*.
1. **Click** the radio button for *IBM Cloud Pak for Data Enterprise Edition V2.5 - Installer Linux x86 Multilingual (CC3Y1ML)*
1. **Click** the *I agree* radio button and **Click** *Download Now*
1. Locate the *CP4D_EE_Installer_V2.5.bin* file. (probably in the downloads directory)
1. Move to a Linux system.  execute `chmod +x CP4D_EE_Installer_V2.5.bin`
1. Exceute the command `./CP4D_EE_Installer_V2.5.bin`  This will download the installer for linux and Mac (darwin) in a tar file *cloudpak4data-ee-v2.5.0.0*
1. Extract the tar file `tar -xvf cloudpak4data-ee-v2.5.0.0`
1. Find a file called `repo.yaml`  This is a simple file, but key. Here is where you will add the `apikey` to get access to the container registry for the additional Services.   
1. In a web browser, **Login** and access [the entitlement registry](https://myibm.ibm.com/products-services/containerlibrary).  **Click** *Get entitlement key* on the left side above Library.  Either *Generate a key* or *Copy the existing key*.  This will populate your scratchpad where you can paste it into the `repo.yaml` file.
1.  **Paste** the *apikey* into the Yaml file to the right of the `apikey:`  After you paste, it should something like this only longer.
~~~
registry:
  - url: cp.icr.
    username: cp
    apikey: eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJ
    name: base-registry
fileservers:
 - url: https://raw.
~~~~


## Install Db2 Warehouse
1. The first thing you will want to do is to pick one node that will house Db2 Warehouse and add a label.
   1. Rum: `oc get nodes`   This will produce a list of nodes.
   1. Run: `oc label node <node name or IP Address> icp4data=database-db2wh`
1. Run env to verify that the following variables are exported

`export NAMESPACE=zen`
`export STORAGE_CLASS=ibmc-file-gold-gid`
`export DOCKER_REGISTRY_PREFIX=$(oc get routes docker-registry -n default -o template={{.spec.host}})`

1. Set the security aspects for Db2 Warehouse to install properly
  `./cpd-linux adm --repo ../repo.yaml  --namespace ${NAMESPACE} --apply --accept-all-licenses --assembly db2wh`
1. Deploy Db2 Warehouse by running the following:
  `./cpd-darwin --repo ../repo.yaml --namespace ${NAMESPACE} --storageclass ${STORAGE_CLASS} --transfer-image-to=${DOCKER_REGISTRY_PREFIX}/${NAMESPACE} --target-registry-username=ocadmin  --target-registry-password=$(oc whoami -t) --cluster-pull-prefix docker-registry.default.svc:5000/${NAMESPACE} --insecure-skip-tls-verify --assembly db2wh`
1. This will take some time to download, push to the registry, request new storage from IBM Cloud and provision the services and pods.  



### Provision a Database instance
1. Once installed and all pods are up, you can go to the service catalog page with the square with petals icon in upper right.  
1. On the services page, **Click** the left side filter to go to *Datasources* to get to **Db2 Warehouse** tile.  
1. **Click** the 3 vertical dots on upper left of the tile.
1. **Click** *Provision Instance*.
1. On the *Configure* page keep defaults or adjust if you know you need more.  **Click** *Next*.
1. On the *Storage* page, Select *Create new storage*; Change **Storage Class** to *ibmc-file-gold-gid*; Adjust the size to reflect the amount needed.  *Default is 100GB*.
1. **Click** *Next*
1. Review the settings. Here you can change the **Display name** to something more memorable. **Click** *Create*.
1. Your instance is being created and will be accessible from the **Services > Db2 Warehouse tile.**  Also accessed from the Left menu *My Instance* then **Click** *Provisioned instances*
1. From *Provision Instances* on the left you will see 3 horizontal dots.  **Click** this and see the options.
  - *Open* will open the DB2 Warehouse instance for use.
  - *View details* will provide you the details including *user* ; *password* ; *jdbc url*
  - *Manage Access* let you add user ids and assign them *Admin* or *User* roles.
  - *Delete* This will delete this particular instance.
**Note:**  If you resize your cluster's worker pool to a lower number of nodes, it is possible that the node with the label for Db2 Warehouse may be deleted.   This would render any database instances unusable until you relabel a node and restart the pods so they start on the same node.

### Create a Table  
1. When you Open the interface, you will run as the user id that provisioned it, for example **user999** which is **admin**, so any tables that are created, by *admin*, via *SQL editor* by default will go under **Schema** *user999*.
  - You can use the UI and select and create tables under certain schemas.

### uninstalling DB2 Warehouse.
1. First you will want to release any storage and delete instances to make sure storage is released.   Use *Delete* to do this.
1. From the commands. 1) set namespace 2) do a dry run unintall to check what will be taken off. 3) run the uninstall.
  - `export NAMESPACE=zen`  My namespace is *zen* your may be different like *default*
  - `./cpd-darwin uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly db2wh --uninstall-dry-run`
  - `./cpd-darwin uninstall --namespace ${NAMESPACE} --repo docker-registry.default.svc:5000/${NAMESPACE}  --assembly db2wh`
1.  Go to the Services catalog and verify that Db2 Wareshouse is no longer enabled.   
