# Partner Demo Environments on IBM Cloud
- [Product demo (free) Good if limited customization needed](#product-demo-free-good-if-limited-customization-needed)
- [Other options to think about](#other-options-to-think-about)
- [Persistent Demo Environment (Cost)](#persistent-demo-environment-cost)
  * [Prerequisite steps to get entitlement to Cloud Pak software:](#prerequisite-steps-to-get-entitlement-to-cloud-pak-software)
  * [PartnerWorld Cloud Credits IBM Cloud account](#partnerworld-cloud-credits-ibm-cloud-account)
  * [Proof of Technology (POC) account](#proof-of-technology-poc-account)
  * [Creating an IBM Cloud account without credits.](#creating-an-ibm-cloud-account-without-credits)
- [Add IBM Cloud Security instructions and best practices](#add-ibm-cloud-security-instructions-and-best-practices)
- [Creating an Open Shift cluster](#creating-an-open-shift-cluster)
  * [Via the UI](#via-the-ui)
  * [Via the Command line](#via-the-command-line)
- [Checking OpenShift was created and working properly](#checking-openshift-was-created-and-working-properly)
- [Using OpenShift console](#using-openshift-console)
- [Installing the CLI environment](#installing-the-cli-environment)
  * [Retrieving the token to log into OpenShift on IBM Cloud](#retrieving-the-token-to-log-into-openshift-on-ibm-cloud)
  * [Quick run through of some OpenShift CLI commands](#quick-run-through-of-some-openshift-cli-commands)
- [Install your Cloud Pak of Choice (currently only Data)](#install-your-cloud-pak-of-choice)


## Product demo (free) Good if limited customization needed
1. Product demos where you are just showing the product with some canned assets, you can use [Cloud Pak for Data Experiences](https://www.ibm.com/cloud/paks/experiences/cloud-pak-for-data)  
**Limitations:** Not every service is provisioned, but these are live clusters with multiple people sharing the environment.  Meaning while you can add live data (at this moment), but others can see it.  You can spin this up in 10 minutes and have most of what you need there.  There is internal discussions of locking this down tighter as mainly a sales tool.

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

## Other options to think about
<!---  - Blue Demos (need competency to view certain assets) - Currently no CPD partner facing VMs. --->
<!---  - Quick ready-for environments with canned set of services.  --->
  - IBM Cloud to demonstrate product functionality
  - Build on your own hardware

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

## Persistent Demo Environment (Cost)
If you are looking for something you can personalized, isolated and persistent, you can go down the path of a cluster of your own on IBM Cloud. Following the steps below, you will be responsible for infrastructure costs, but we do have some offsets that can easy this monetary pain.

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

### Prerequisite steps to get entitlement to Cloud Pak software:
1. Subscribe to an IBM Partner Package.  Every new package has the Software Access Catalog plus access to PartnerWorld Software developer to developer support, so choose the right [Partner Package for your business](https://www.ibm.com/partnerworld/program/benefits/partner-package) **Note:** Must be logged in to PartnerWorld to see details
1. Add employees to your IBM PartnerWorld Profile.  This will provide them with authorization to entitled software on premises or Cloud Container registry.  **Note:**  This could take up to 24 hours to take hold.
1. Verify that you have access to [the entitlement registry](https://myibm.ibm.com/products-services/containerlibrary).  Click ***Library***, it should say `IBM SOFTWARE ACCESS 1 YEAR`

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)


### PartnerWorld Cloud Credits IBM Cloud account
1. You may be entitled to ***IBM Cloud credits free of charge*** based on your **PartnerWorld Program Status**, or may choose to ***purchase*** an [IBM Partner Package](https://www.ibm.com/partnerworld/program/benefits/partner-package) (with or without a [Booster Pack](https://www.ibm.com/partnerworld/program/benefits/partner-package-booster) to get more.  [Click here](https://www.ibm.com/partnerworld/program/benefits/cloud-credit) to learn about how many credits you can obtain and [request your credits to be activated via this form](https://www.ibm.com/partnerworld/wps/servlet/mem/ContentHandler/pw_frm_bam_mrb_ibm-cloud-credits-for-partner-package).   Once requested, the IBM PartnerWorld team will help set up an account with the IBM Cloud credits loaded for you to use.
1. Verify you can log into your companies account.
1. If you need more ***IBM Cloud credits*** than your base **Partner Package** offers, you can also choose to [purchase a Booster Pack](https://www.ibm.com/partnerworld/program/benefits/partner-package-booster) to add more ***IBM Cloud credits*** to your account.
1. Best practices on [setting up your account](https://cloud.ibm.com/docs/account?topic=account-account_setup)
1. Add additional people to the organization's IBM Cloud account.  Make sure these email ids are also listed in your companies PartnerWorld Profile.

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

### Proof of Technology (POC) account
How do I request an account?  What qualifies? Can my VAD help here?
***Coming soon***

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

###  Creating an IBM Cloud account without credits.
1. Sign up for [IBM Cloud](https://cloud.ibm.com/).
1. Add credit card information or contact [IBM Sales for a subscription](https://www.ibm.com/cloud/pricing).  
1. Best practices on [setting up your account](https://cloud.ibm.com/docs/account?topic=account-account_setup)
1. Add additional people to the organization's IBM Cloud account.  Make sure these email ids are also listed in your companies PartnerWorld Profile.

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

## Add IBM Cloud Security instructions and best practices
1. Once you have access to the account, it is best to create a Resource group and Access Group in IAM.
1. Read [Best practices for organizing resources and assigning access](https://cloud.ibm.com/docs/account?topic=account-account_setup)
1. Go to the [Access (IAM) dashboard](https://cloud.ibm.com/iam/overview), **Click** ***Resource Groups***.  This will make it easier to contain access.
1. **Click** the ***Create***  then **enter** a value for your resource group.  I will pick ***CPD***.
1. From the Console go to [Manage>Access(IAM)>Groups](https://cloud.ibm.com/iam/groups)
1. **Click** ***Create***  add you group name here.
1. **Click** the ***Users*** tab then **Add Users**. This will produce a list of users that can be added. Always good to add yourself
1. **Click** the ***Access Policies*** tab and select the following services and appropriate permissions.
  - Cloud Object Storage  (Administrator or Editor)
  - Infrastructure Services (Administrator or Editor)
  - Cloud Foundry Enterprise Environment (Administrator or Writer)
  - Container Registry (Manager or Writer)
  - Kubernetes Service (Manager or Writer)
  - Certificate Manager (Manager or Writer)
  - User Management (Editor)
  - IAM Access Groups Service (Editor)
  - Internet Services (Administrator)
  - DNS Services (Administrator)
  - Add appropriate controls for your resource group

**Note:** For further reading on [Access Management on IBM Cloud](https://cloud.ibm.com/docs/account?topic=account-cloudaccess)  


[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

## Creating an Open Shift cluster
### Via the UI
1. Log into [IBM Cloud](https://cloud.ibm.com/)
1. Click on **Catalog** > **Services**
1. Filter on ***Containers*** by checking the box on the left.
1. Select **Red Hat OpenShift on IBM Cloud**
1. On the right is a pane for estimated monthly cost, since the cost is based on hourly tiered Kubernetes Compute plus the 30 day OpenShift license.  If you are authenticated and authorized for **PartnerWorld Software Access Catalog** per above instructions, you are only responsible the hourly compute used, however you will be billed even if Compute resources are down.  **This service does not have suspendible billing.**
1. On the left is where you will alter the version, license, type of compute and number of worker nodes. **Note:** You are only paying for Worker nodes no master or bastion nodes.  
1. Going down the page:
  1. First select the ***OpenShift version***
  1. **OpenShift Entitlement**;  Select ***Apply my Cloud Pak entitlement to this worker pool***
  1. Select **Resource Group** - ***Default*** unless you created your own which is easier to organize later.
  1. Pick your **Geography**
  1. Pick your **Availability** ***Single zone***
  1. Pick **Worker Zone**
  1. Click **Change Flavor** to select the appropriate Flavor then click ***Done***
    **Example** Cloud Pak for Data uses either 16 core/64GB RAM or 32 core/64GB RAM.   
  1. Pick number of **worker nodes** **Example:** Cloud Pak for Data start with ***3***. ***suggestions on bare minimum in table below*** I am running **Cloud Pak for Data** plus all favorite ***Services*** all that is installed on 7 node - 16 core 64GB cluster.  I tried 32 core and 64GB Ram, but It didn't seem to buy me much around deployment.  With ROKS, it is easy to add or subtract from your worker pool.
  1. Adjust your **cluster name** to your liking. **Example** ***CloudPakDataDemo-HealthCare***
  1. Click **Create** on the right hand side blue button.  Just above this is your estimate.  Make sure that there is a line item for **Cloud Pak entitlement** with a ***negative*** value.
  1. This can take ***30 minutes*** to provision.

  |Cloud Pak|Nodes|Size: Cores/RAM|
  |:---|:---:|:---:|
  |Apps|1|4/16|
  |Automation|3|8/32|
  |Data|4|16/64|
  |Integration|3|8/32|
  |Multi-Cloud Management|1|1/8|
  |Security|3|8/32|

**Note:** Creating a 3.11 or 4.3 cluster on IBM Cloud is the same steps, except selecting the version.

<iframe width="600" height="322" src="https://www.youtube.com/embed/dDC9eqIMO8U" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

### Via the Command line
1. Log into your account using `ibmcloud login`
1. List out all the locations which provide infrastructure for **OpenShift** using ***classic*** public cloud. There are other options which are a little more complicated to create, but you can get isolation and latest networking and infrastructure.   ***vpc-classic***  and ***vpc-gen2***
  - `ibmcloud oc locations --provider classic`
  - If you want to narrow it to a Geography: Americas (na), Europe (eu) or Asia Pacific (ap)
   ~~~
   Toms-MBP:bin tjm$ ibmcloud oc locations --provider classic | grep \(na\)
   dal13   Dallas (dal)†           United States (us)    North America (na)   
   wdc06   Washington DC (wdc)†    United States (us)    North America (na)   
   sjc04   San Jose (sjc)          United States (us)    North America (na)   
   dal12   Dallas (dal)†           United States (us)    North America (na)   
   wdc07   Washington DC (wdc)†    United States (us)    North America (na)   
   tor01   Toronto (tor)           Canada (ca)           North America (na)   
   sjc03   San Jose (sjc)          United States (us)    North America (na)   
   wdc04   Washington DC (wdc)†    United States (us)    North America (na)   
   hou02   Houston (hou)           United States (us)    North America (na)   
   dal10   Dallas (dal)†           United States (us)    North America (na)   
   mex01   Mexico City (mex-cty)   Mexico (mex)          North America (na)   
   mon01   Montreal (mon)          Canada (ca)           North America (na)  
   ~~~
1. In this case, I have picked ***Washington DC*** datacenter ***06*** and I want to know what worker node configurations are offered. Generally I want either ***32x64GB*** or ***16x64GB*** servers.  To get the list execute: `ibmcloud oc flavors --zone wdc06 --provider classic`   Since I know that I only want virtual servers and servers with 64GB of RAM, I will run the following `ibmcloud oc flavors --zone wdc06 --provider classic | grep x64 | grep -v physical`
~~~
Toms-MBP:bin tjm$ ibmcloud oc flavors --zone wdc06 --provider classic | grep x64 | grep -v physical
b2c.16x64                 16      64GB     1000Mbps        UBUNTU_16_64   virtual       25GB         100GB               classic   
b3c.16x64                 16      64GB     1000Mbps        UBUNTU_18_64   virtual       25GB         100GB               classic   
c2c.32x64                 32      64GB     1000Mbps        UBUNTU_16_64   virtual       25GB         100GB               classic   
c3c.32x64                 32      64GB     1000Mbps        UBUNTU_18_64   virtual       25GB         100GB               classic   
m2c.8x64                  8       64GB     1000Mbps        UBUNTU_16_64   virtual       25GB         100GB               classic   
m3c.8x64                  8       64GB     1000Mbps        UBUNTU_18_64   virtual       25GB         100GB               classic   
~~~
1. You need to have a VLAN ID.  `ibmcloud oc vlan ls --zone wdc06`
~~~
Toms-MBP:bin tjm$ ibmcloud oc vlan ls --zone wdc06
OK
ID        Name   Number   Type      Router         Supports Virtual Workers   
2942828          1250     private   bcr01a.wdc06   true   
2942826          1315     public    fcr01a.wdc06   true   
~~~
1. You will need the version of OpenShift for the command line.   Use `ibmcloud oc versions`  You can get more specific using the --show-version.  These options are ***openshift*** or ***kubernetes***.  Below I am listing out only the OpenShift versions
~~~
Toms-MBP:bin tjm$ ibmcloud oc versions --show-version openshift
OK
OpenShift Versions   
3.11.248_openshift   
4.3.31_openshift (default)   
4.4.17_openshift   
~~~
1. Now that you have all the information need to build an OpenShift Cluster.  Here is the command line to build out an OpenShift Cluster in the Wahington DC Data Center, using virtual servers with 16 cores and 64GB RAM.  The cluster will be using shared hardware using 3 worker pool of 3.   I also want to use the entitlement from the Cloud Pak. Without ***--entitlement cloud_pak*** your account will be charged for an OpenShift license.   Run the following `ibmcloud oc cluster create classic --zone wdc06 --flavor b3c.16x64 --hardware shared --public-vlan 2942826  --private-vlan 2942828 --workers 3 --name commandline --version 4.3.31_openshift  --public-service-endpoint --entitlement cloud_pak`
~~~
Toms-MBP:bin tjm$  ibmcloud oc cluster create classic --zone wdc06 --flavor b3c.16x64 --hardware shared --public-vlan 2942826  --private-vlan 2942828 --workers 3 --name commandline --version 4.3.31_openshift  --public-service-endpoint --entitlement cloud_pak
Creating cluster...
OK
Cluster created with ID bt932qpw05g1h1gagch0
~~~
1. Let's list the clusters Use `ibmcloud oc cluster ls`
~~~
Toms-MBP:bin tjm$ ibmcloud oc cluster ls
OK
Name                        ID                     State           Created          Workers   Location          Version                 Resource Group Name   Provider   
commandline                 bt932qpw05g1h1gagch0   normal          26 minutes ago   3         Washington D.C.   4.3.31_1536_openshift   Default               classic   
~~~


## Checking OpenShift was created and working properly
1. Open the [Dashboard](https://cloud.ibm.com/) on IBM Cloud.
1. Click the link for ***Clusters***  This give you a high level view of the cluster.  Take note of the version up until the underbar.  You will need to match this version with your OpenShift Client. **Example:**
1. Click on the cluster name **Example:** CloudPakDataDemo-HealthCare
1. From here you can see all vital information at one glance.
  1. Click on ***Worker nodes***  This lists out the specifics for each virtual server.
  1. Click ***Worker pool***. This shows you how many in the pool, flavor, etc  The three dots to the right allow you to delete or resize.   If you started with three and need more, you can ***resize*** the worker pool.  up or down.
  1. Click ***Access***.  This is all the tools you will want to access the underlying OpenShift cluster from the command line.   You can do things from the console, but many folks want to use command line. You will need these to provision additional Cloud Pak services. Go ahead and download these to your system. Make sure that the OpenShift Client is the same version as your cluster.

  <iframe width="600" height="322" src="https://www.youtube.com/embed/RexZGfz_D04" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

## Using OpenShift console
  1. Since master nodes are shared in the ROKS environment and you will use a token to gain access, you can get there via your IBM Cloud account.  
  1. From the Main IBM Cloud page, **click** on the ***Navigation menu*** in thee upper left.   
  1. **Scroll down** to ***OpenShift***
  1. **Click** ***Clusters***
  1. **Click** on your cluster. Mine is ***CloudPakData***.
  1. If you are not in ***Overview*** then **Click** ***Overview***
  1. **Click** ***OpenShift web console***
  1. Since we want to install to a project other than default, **Click** the ***+ Create Project*** button int the upper right.
    - Add ***zen*** to the **Name**
    - Add ***zen*** or ***Cloud Pak for Data*** to **Display Name**
    - Add something for you knowledge to description. I'll leave mine blank.
  1. This is where you will install the Cloud Pak for Data Services.
  1. You can work with Deployments, jobs, volume claims and Pods later by accessing this space.

- **OpenShift 3.11 UI Demo video**

  <iframe width="600" height="322" src="https://www.youtube.com/embed/TPgUJkIyQoY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

  [Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

## Installing the CLI environment
  1. Download the Client.  **Note** This should match your server side.
  - [3.x Client download](https://mirror.openshift.com/pub/openshift-v3/clients/)
  - [4.x Client download](https://mirror.openshift.com/pub/openshift-v4/clients/oc/)
  1. unzip and copy oc to your `/usr/local/bin` or include in PATH.
  1. Execute oc version to check that everything is working.  
  1. [Retrieving the token to log into OpenShift on IBM Cloud](#retrieving-the-token-to-log-into-openshift-on-ibm-cloud)
  1. While you can install the Cloud Paks into the default project, it's a better idea to put it in its own project or namespace.  These terms are linked.  Let's create a new project.  I'll call mine `zen` from the terminal while logged in issue `oc new-project zen`  This will create a zen project and you will use this project name when creating the Cloud Pak.  
  1.  If you are going to customize the Cloud Pak cluster with other services your will need to create a Route to the internal container registry.  These are your two commands

  - ***OCP 3.11***
    - `oc create route reencrypt --service=docker-registry -n default`
    - `oc annotate route docker-registry --overwrite haproxy.router.openshift.io/balance=source -n default`
    - `oc get routes -n default`
  - ***OCP 4.x***
    - `oc get routes -n openshift-image-registry`

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

### Retrieving the token to log into OpenShift on IBM Cloud
 1. Go to the dashboard and in the upper right click the blue button/link for OpenShift Web Console.
 1. In the upper right hand corner, there should be a person icon, **click** the arrow and **click** ***Copy Login Command***.  
 - **OCP 3.11:** This will provide the login with your token in to your copy buffer.  **This token is renewed daily.**
 - **OCP 4.x:** This will launch a new page with a single URL ***Display Token***.  **Click** this URL.  **Copy** the contents of the ***Log in with this token*** gray area. **This token is renewed daily.**
 1. Paste into your terminal window.  `oc login https://c106-e.us-south.containers.cloud.ibm.com:30783 --token=EAVMH6YNi0BA88H3VO90v_WidUoNNPOtF3u4Tg`
 1. Test out command line connectivity to underlying OpenShift infrastructure:  `oc version`  or `oc get pods` You can also do much of this through the OpenShift Console

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

### Quick run through of some OpenShift CLI commands
In the section installing, you ran a few commands to validate the installation.  Let go a little deeper to showcase what you might want to be able to do from the command line.

1. **List** all the **pods** in the **cluster**.  You should see on the left the **namespaces** or **projects**. next moving right is the **pod name**, how many are **ready**, their **state**, **restarts** and days running or **age**.
`oc get pods --all-namespaces`
~~~
Toms-MBP:bin tjm$ oc get pods --all-namespaces
NAMESPACE                           NAME                                                              READY     STATUS      RESTARTS   AGE
default                             docker-registry-6d6cdd559d-ll5rz                                  1/1       Running     0          3d
default                             docker-registry-6d6cdd559d-w7pb6                                  1/1       Running     0          3d
...
ibm-system                          ibm-cloud-provider-ip-169-60-227-155-6b8c7f49cf-nm9v5             1/1       Running     0          3d
...
kube-proxy-and-dns                  proxy-and-dns-rgkzw                                               1/1       Running     0          18d
kube-proxy-and-dns                  proxy-and-dns-tt6rz                                               1/1       NodeLost    0          7d
...
kube-service-catalog                apiserver-64d8986ddd-22f2z                                        1/1       Running     0          3d
...
kube-system                         vpn-5848948c84-c9frz                                              1/1       Running     0          3d
...
openshift-ansible-service-broker    asb-58f44b5c6c-6472d                                              1/1       Unknown     0          3d
openshift-ansible-service-broker    asb-58f44b5c6c-jz6zn                                              1/1       Running     0          18m
...
openshift-monitoring                prometheus-operator-68ccbf549-dwxld                               1/1       Running     0          3d
...
openshift-template-service-broker   apiserver-7df5d95cfb-9pzlz                                        1/1       Unknown     0          3d
openshift-template-service-broker   apiserver-7df5d95cfb-knzdc                                        1/1       Running     0          3d
~~~
1. **List** all the **pods** in a **namespace**.  Compare this list to the last.  Notice anything?  The ***Unknown*** is gone and replaced with a different pod.  OpenShift realized it was not responsive and spun up a different pod, then shut down the old one.  
~~~
Toms-MBP:bin tjm$ oc get pods -n openshift-template-service-broker
NAME                         READY     STATUS    RESTARTS   AGE
apiserver-7df5d95cfb-knzdc   1/1       Running   0          3d
apiserver-7df5d95cfb-p9cxm   1/1       Running   0          26m
~~~
Try another, notice ***NodeLost*** is gone. **NodeLost** in this case the pod name is the same, so the updated state kept OpenShift from restarting the pod or deleting.  
~~~
Toms-MBP:bin tjm$ oc get pods -n kube-proxy-and-dns  
NAME                  READY     STATUS    RESTARTS   AGE
proxy-and-dns-2866f   1/1       Running   0          18d
proxy-and-dns-5m5tr   1/1       Running   0          18d
proxy-and-dns-b997g   1/1       Running   0          18d
proxy-and-dns-d8jrr   1/1       Running   0          18d
proxy-and-dns-fk74t   1/1       Running   0          18d
proxy-and-dns-rgkzw   1/1       Running   0          18d
proxy-and-dns-tt6rz   1/1       Running   1          7d
~~~
1. If you have a pod that is **Evicted** or **Terminating**, you may want to delete or reboot these manually. You can gracefully terminate and restart a pod or force it down. We are going to **delete** a **pod** in the ***openshift-template-service-broker*** namespace.  **Note:** my second command, I forgot to add a namespace.
~~~
Toms-MBP:bin tjm$ oc get pods -n openshift-template-service-broker
NAME                         READY     STATUS    RESTARTS   AGE
apiserver-7df5d95cfb-knzdc   1/1       Running   0          3d
apiserver-7df5d95cfb-p9cxm   1/1       Running   0          35m
Toms-MBP:bin tjm$ oc delete pod apiserver-7df5d95cfb-knzdc
Error from server (NotFound): pods "apiserver-7df5d95cfb-knzdc" not found
Toms-MBP:bin tjm$ oc delete pod apiserver-7df5d95cfb-knzdc -n openshift-template-service-broker
pod "apiserver-7df5d95cfb-knzdc" deleted
Toms-MBP:bin tjm$ oc get pods -n openshift-template-service-broker
NAME                         READY     STATUS    RESTARTS   AGE
apiserver-7df5d95cfb-55dcd   0/1       Running   0          13s
apiserver-7df5d95cfb-p9cxm   1/1       Running   0          36m
Toms-MBP:bin tjm$ oc get pods -n openshift-template-service-broker
NAME                         READY     STATUS    RESTARTS   AGE
apiserver-7df5d95cfb-55dcd   1/1       Running   0          35s
apiserver-7df5d95cfb-p9cxm   1/1       Running   0          36m
~~~
1. For an **Evicted** pod, it might not go gracefully, so you can add `--grace-period=0 --force` to your delete command.
~~~
Toms-MBP:~ tjm$ oc delete pods apiserver-7df5d95cfb-p9cxm -n openshift-template-service-broker --grace-period=0 --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "apiserver-7df5d95cfb-p9cxm" force deleted
Toms-MBP:~ tjm$ oc get pods -n openshift-template-service-broker
NAME                         READY     STATUS    RESTARTS   AGE
apiserver-7df5d95cfb-55dcd   1/1       Running   0          11m
apiserver-7df5d95cfb-m67nk   1/1       Running   0          35s
~~~
1. Always adding `-n openshift-template-service-broker` or another ***namespace*** can be annoying.  If you think you will be in a specific namespace more often than not, like ***zen*** for **Cloud Pak for Data** then execute `oc project zen`  this will allow you to drop the `-n <namespace>` for anything that is in this namespace.
1. If you want to look into the logs of a pod, you can run `oc logs <pod name> -n <namespace>`  Sometimes there is a lot of information and other times not so much.   You can watch or follow the logs by adding `-f` to the end of the command.
~~~
Toms-MBP:~ tjm$ oc logs apiserver-7df5d95cfb-m67nk -n openshift-template-service-broker
I0616 17:12:52.000294       1 serve.go:116] Serving securely on [::]:8443
I0616 17:12:52.000593       1 controller_utils.go:1025] Waiting for caches to sync for tsb controller
I0616 17:12:52.101071       1 controller_utils.go:1032] Caches are synced for tsb controller
~~~
1. If this doesn't yield much information, then you can use `oc describe <resource>`.
~~~
Toms-MBP:~ tjm$ oc describe pod  apiserver-7df5d95cfb-m67nk -n openshift-template-service-broker
Name:               apiserver-7df5d95cfb-m67nk
Namespace:          openshift-template-service-broker
Priority:           0
PriorityClassName:  <none>
Node:               10.95.7.50/10.95.7.50
Start Time:         Tue, 16 Jun 2020 13:12:49 -0400
...
...
Events:
  Type    Reason     Age   From                 Message
  ----    ------     ----  ----                 -------
  Normal  Scheduled  14m   default-scheduler    Successfully assigned openshift-template-service-broker/apiserver-7df5d95cfb-m67nk to 10.95.7.50
  Normal  Pulled     14m   kubelet, 10.95.7.50  Container image "registry.ng.bluemix.net/armada-master/iksorigin-ose-template-service-broker:v3.11.216" already present on machine
  Normal  Created    14m   kubelet, 10.95.7.50  Created container
  Normal  Started    14m   kubelet, 10.95.7.50  Started container
~~~  


**MORE TO COME**

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)

## Install your Cloud Pak of Choice
<!---  [Cloud Pak for Apps](apps.md) [Cloud pak for Automation](automation.md) --->
  - [Cloud Pak for Data 2.5](Data25.md)
  - [Cloud Pak for Data 3.0](Data30.md)
<!---  [Cloud Pak for Integration](integration.md)[Cloud Pak for Mult-Cloud Manager](mcm.md) --->

[Back to Table of Contents](https://tjmcmanus.github.io/IBMPartnerDemo)
