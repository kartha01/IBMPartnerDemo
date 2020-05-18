# Partner Demo Environments on IBM Cloud

## Ephemeral Product demo (Free) Good if limited customization needed
1. Product demos where you are just showing the product with some canned assets, you can use [Cloud Pak for Data Experiences](https://www.ibm.com/cloud/paks/experiences/cloud-pak-for-data)  
  Not every service is provisioned, but these are live clusters with multiple people sharing the environment.  Meaning while you can add live data (at this moment), but others can see it.  You can spin this up in 10 minutes and have most of what you need there.  There is internal discussions of locking this down tighter as mainly a sales tool.

## Persistent Demo Environment (Cost)
If you are looking for something you can personalized, isolated and persistent, you can go down the path of a cluster of your own on IBM Cloud. Following the steps below, you will be responsible for infrastructure costs, but we do have some offsets that can easy this monetary pain.
### Prerequisite steps to get entitlement to Cloud Pak software:
1. Already an IBM PartnerWorld Partner and have purchased the Value Package subscription.  If not please find the correct [Value Pack subscription here](https://www.ibm.com/partnerworld/program/benefits/partner-package).
1. Add employees to your IBM PartnerWorld Profile.  This will provide them with authorization to entitled software on premises or Cloud Container registry.
1. Verify that you have access to [the entitlement registry](https://myibm.ibm.com/products-services/containerlibrary).  Click Library, it should say "IBM SOFTWARE ACCESS 1 YEAR"

### Creating an IBM Cloud account
1. Decide on what [type of account](https://cloud.ibm.com/docs/account?topic=account-accounts) you desire for your Organization.
1.[Create an IBM Cloud account](https://cloud.ibm.com/registration?cm_sp=Cloud-Home-_-LeadspaceReg-IBMCloud_CloudHome-_-LSReg) for your Organization. You will want to add people to this and use business email addresses. These business email addresses should also be listed in your IBM PartnerWorld Profile. This gains you entitlement to Not For Resale IBM Software and Container registry for Cloud Paks.
1. Verify you can log into your companies account.
1. Request your [PartnerWorld IBM Cloud Credits](https://www-356.ibm.com/partnerworld/wps/servlet/mem/ContentHandler/pw_frm_bam_mrb_ibm-cloud-credits).  You will need your cloud account #, so the credits are allocated appropriately.  This is is an annual action.
1. Best practices on [setting up your account](https://cloud.ibm.com/docs/account?topic=account-account_setup)
1. Add additional people to the organization's cloud account.

### Creating an Open Shift cluster
1. Log into [IBM Cloud](https://cloud.ibm.com/)
1. Click on Catalog > Services
1. Filter on Containers by checking the box on the left.
1. Select Red Hat OpenShift on IBM cloud
1. On the right is a pane for estimated monthly cost, since the cost is based on hourly tiered Kubernetes Compute plus the 30 day OpenShift license.  If you are authenticated and authorized for PartnerWorld Software Access Catalog per above instructions, you are only responsible the hourly compute used.  This suggests that you can quiesce the virtual servers to pause the hourly charges.
1. On the left is where you will alter the version, license, type of compute and number of worker nodes. **Note:** You are only paying for Worker nodes no master or bastion nodes.  
1. Going down the page:
  1. First select the OpenShift version
  1. OCP Entitlement;  Select *Apply my Cloud Pak entitlement to this worker pool*
  1. Select Resource Group - *Default* unless you created your own which is easier to organize later.
  1. Pick your Geography
  1. Pick your Availability *Single zone*
  1. Pick worker zone
  1. Click Change Flavor to select the appropriate Flavor then click *Done* ***Example*** Cloud Pak for Data uses either 16 core/64GB RAM or 32 core/64GB RAM.   
  1. Pick number of worker nodes ***Example:*** Cloud Pak for Data start with 3. *suggestions in table below*
  1. Adjust your cluster name to your liking. ***Example*** CloudPakDataDemo-HealthCare
  1. Click *Create* on the right hand side blue button.  Just above this is your estimate.  Make sure that there is a line item for Cloud Pak entitlement with a negative value.

|Cloud Pak|Nodes|Size: Cores/RAM|
|:---|:---:|:---:|
|Apps|1|4/16|
|Automation|3|8/32|
|Data|3|16/64|
|Integration|3|8/32|
|Multi-Cloud Management|1|1/8|
|Security|3|8/32|

<iframe width="600" height="322" src="https://www.youtube.com/embed/dDC9eqIMO8U" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Checking OpenShift was created and working properly
1. Open the [Dashboard](https://cloud.ibm.com/) on IBM Cloud.
1. Click the link for *Clusters*  This give you a high level view of the cluster.  Take note of the version up until the underbar.  You will need to match this version with your OpenShift Client. ***Example:***
1. Click on the cluster name ***Example:*** CloudPakDataDemo-HealthCare
1. From here you can see all vital information at one glance.
  1. Click on *Worker nodes*  This lists out the specifics for each virtual server.
  1. Click *Worker pool*. This shows you how many in the pool, flavor, etc  The three dots to the right allow you to delete or resize.   If you started with three and need more, you can *resize* the worker pool.  up or down.
  1. Click *Access*.  This is all the tools you will want to access the underlying OpenShift cluster from the command line.   You can do things from the console, but many folks want to use command line. You will need these to provision additional Cloud Pak services. Go ahead and download these to your system. Make sure that the OpenShift Client is the same version as your cluster.

  <iframe width="600" height="322" src="https://www.youtube.com/embed/RexZGfz_D04" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


### Installing the client environment
  1. Download the Client
  1. unzip and copy oc to your `/usr/local/bin` or include in PATH.
  1. Execute oc version to check that everything is working.  
  1. Go back to the dashboard and in the upper right click the blue button/link for OpenShift Web Console.
  1. in the upper right hand corner, there should be a person icon, click the arrow and click "Copy Login Command".  This will out the login with your token in to your copy buffer.  **This token is renewed daily.**
  1. Paste into your terminal window.  `oc login https://c106-e.us-south.containers.cloud.ibm.com:30783 --token=EAVMH6YNi0BA88H3VO90v_WidUoNsG7reNPOtF3u4Tg`
  1. Test out command line connectivity to underlying OpenShift infrastructure:  `oc version`  or `oc get pods` You can also do much of this through the OpenShift Console
  1. While you can install the Cloud Paks into the default project, it's a better idea to put it in its own project or namespace.  These terms are linked.  Lets create a new project.  I'll call mine `zen` from the terminal while logged in issue `oc new-project zen`  This will create a zen project and you will use this project name when creating the Cloud Pak.  
  1.  If you are going to customize the Cloud Pak cluster with other services your will need to create a Route to the internal container registry.  These are your two commands
  - `oc create route reencrypt --service=docker-registry -n default`
  - `oc annotate route docker-registry --overwrite haproxy.router.openshift.io/balance=source -n default`
  - `oc get routes -n default`


### Install your Cloud Pak of Choice
  - [Cloud Oak for Apps](apps.md)
  - [Cloud pak for Automation](autmoation.md)
  - [Cloud Pak for Data](Data.md)
  - [Cloud Pak for Intergration](integration.md)
  - [Cloud Pak for Mult-Cloud Manager](mcm.md)
