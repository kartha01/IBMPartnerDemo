# Partner Demo Environments on IBM Cloud
- [Ephemeral Product demo (Free) Good if limited customization needed](#ephemeral-product-demo--free--good-if-limited-customization-needed)
- [Other options  to think about](#other-options--to-think-about)
- [Persistent Demo Environment (Cost)](#persistent-demo-environment--cost-)
  * [Prerequisite steps to get entitlement to Cloud Pak software:](#prerequisite-steps-to-get-entitlement-to-cloud-pak-software-)
  * [PartnerWorld Cloud Credits IBM Cloud account](#partnerworld-cloud-credits-ibm-cloud-account)
  * [POC account](#poc-account)
  * [Creating an IBM Cloud account without credits.](#creating-an-ibm-cloud-account-without-credits)
  * [Add IBM Cloud Security instructions and best practices](#add-ibm-cloud-security-instructions-and-best-practices)
  * [Creating an Open Shift cluster](#creating-an-open-shift-cluster)
  * [Checking OpenShift was created and working properly](#checking-openshift-was-created-and-working-properly)
  * [Using Openshift console](#using-openshift-console)
  * [Installing the CLI environment](#installing-the-cli-environment)
  * [Quick run through of some OpenShift CLI commands  (Coming)](#quick-run-through-of-some-openshift-cli-commands---coming-)
  * [Install your Cloud Pak of Choice](#install-your-cloud-pak-of-choice)

## Ephemeral Product demo (Free) Good if limited customization needed
1. Product demos where you are just showing the product with some canned assets, you can use [Cloud Pak for Data Experiences](https://www.ibm.com/cloud/paks/experiences/cloud-pak-for-data)  
**Limitations:** Not every service is provisioned, but these are live clusters with multiple people sharing the environment.  Meaning while you can add live data (at this moment), but others can see it.  You can spin this up in 10 minutes and have most of what you need there.  There is internal discussions of locking this down tighter as mainly a sales tool.

##  Other options  to think about
  - Blue Demos (need competency to view certain assets) - Currently no CPD partner facing VMs.
  - Quick ready-for environments with canned set of services.


## Persistent Demo Environment (Cost)
If you are looking for something you can personalized, isolated and persistent, you can go down the path of a cluster of your own on IBM Cloud. Following the steps below, you will be responsible for infrastructure costs, but we do have some offsets that can easy this monetary pain.

### Prerequisite steps to get entitlement to Cloud Pak software:
1. Already an IBM PartnerWorld member and have purchased an IBM Partner Package.  Every package has the Software Access Catalog, so choose the right [Partner Package for your business](https://www.ibm.com/partnerworld/program/benefits/partner-package) **Note:** Must be logged in to PartnerWorld to see details
1. Add employees to your IBM PartnerWorld Profile.  This will provide them with authorization to entitled software on premises or Cloud Container registry.
1. Verify that you have access to [the entitlement registry](https://myibm.ibm.com/products-services/containerlibrary).  Click ***Library***, it should say `IBM SOFTWARE ACCESS 1 YEAR`

### PartnerWorld Cloud Credits IBM Cloud account
1. When you picked your **PartnerWorld Package**, an IBM Cloud account will be created for your company.   When this is available perform the following steps.
1. Verify you can log into your companies account.
1. If you need more cloud credits than your base Partner Package offers, you can also choose to purchase a Booster Pack to add more cloud credits to your account.  [Learn more and choose a Booster Pack here.](https://www.ibm.com/partnerworld/program/benefits/partner-package-booster)
1. Best practices on [setting up your account](https://cloud.ibm.com/docs/account?topic=account-account_setup)
1. Add additional people to the organization's cloud account.

### POC account
***Coming soon***

###  Creating an IBM Cloud account without credits.
1. Sign up for IBM CloudPakData
1. Add credit card information or contact IBM Sales for a subscription.  
1. Best practices on [setting up your account](https://cloud.ibm.com/docs/account?topic=account-account_setup)
1. Add additional people to the organization's cloud account.

### Add IBM Cloud Security instructions and best practices
***Coming soon***

### Creating an Open Shift cluster
1. Log into [IBM Cloud](https://cloud.ibm.com/)
1. Click on Catalog > Services
1. Filter on Containers by checking the box on the left.
1. Select Red Hat OpenShift on IBM cloud
1. On the right is a pane for estimated monthly cost, since the cost is based on hourly tiered Kubernetes Compute plus the 30 day OpenShift license.  If you are authenticated and authorized for PartnerWorld Software Access Catalog per above instructions, you are only responsible the hourly compute used.  This suggests that you can quiesce the virtual servers to pause the hourly charges, but not true.  You will be billed even if Compute resources are down.
1. On the left is where you will alter the version, license, type of compute and number of worker nodes. **Note:** You are only paying for Worker nodes no master or bastion nodes.  
1. Going down the page:
  1. First select the OpenShift version
  1. OCP Entitlement;  Select ***Apply my Cloud Pak entitlement to this worker pool***
  1. Select Resource Group - ***Default*** unless you created your own which is easier to organize later.
  1. Pick your Geography
  1. Pick your Availability ***Single zone***
  1. Pick worker zone
  1. Click Change Flavor to select the appropriate Flavor then click ***Done*** **Example** Cloud Pak for Data uses either 16 core/64GB RAM or 32 core/64GB RAM.   
  1. Pick number of worker nodes **Example:** Cloud Pak for Data start with 3. ***suggestions on bare minimum in table below*** I am running all that is installed on 5 node - 16 core 64GB cluster.
  1. Adjust your cluster name to your liking. **Example** CloudPakDataDemo-HealthCare
  1. Click **Create** on the right hand side blue button.  Just above this is your estimate.  Make sure that there is a line item for Cloud Pak entitlement with a negative value.
  1. This can take 30 mimutes to provision.
<br>
|Cloud Pak|Nodes|Size: Cores/RAM|
|:---|:---:|:---:|
|Apps|1|4/16|
|Automation|3|8/32|
|Data|3|16/64|
|Integration|3|8/32|
|Multi-Cloud Management|1|1/8|
|Security|3|8/32|
<br>
<iframe width="600" height="322" src="https://www.youtube.com/embed/dDC9eqIMO8U" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<br>
### Checking OpenShift was created and working properly
1. Open the [Dashboard](https://cloud.ibm.com/) on IBM Cloud.
1. Click the link for ***Clusters***  This give you a high level view of the cluster.  Take note of the version up until the underbar.  You will need to match this version with your OpenShift Client. **Example:**
1. Click on the cluster name **Example:** CloudPakDataDemo-HealthCare
1. From here you can see all vital information at one glance.
  1. Click on ***Worker nodes***  This lists out the specifics for each virtual server.
  1. Click ***Worker pool***. This shows you how many in the pool, flavor, etc  The three dots to the right allow you to delete or resize.   If you started with three and need more, you can ***resize*** the worker pool.  up or down.
  1. Click ***Access***.  This is all the tools you will want to access the underlying OpenShift cluster from the command line.   You can do things from the console, but many folks want to use command line. You will need these to provision additional Cloud Pak services. Go ahead and download these to your system. Make sure that the OpenShift Client is the same version as your cluster.

  <iframe width="600" height="322" src="https://www.youtube.com/embed/RexZGfz_D04" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Using Openshift console
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

  <iframe width="600" height="322" src="https://www.youtube.com/embed/TPgUJkIyQoY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Installing the CLI environment
  1. Download the Client
  1. unzip and copy oc to your `/usr/local/bin` or include in PATH.
  1. Execute oc version to check that everything is working.  
  1. Go back to the dashboard and in the upper right click the blue button/link for OpenShift Web Console.
  1. in the upper right hand corner, there should be a person icon, **click** the arrow and **click** ***Copy Login Command***.  This will out the login with your token in to your copy buffer.  **This token is renewed daily.**
  1. Paste into your terminal window.  `oc login https://c106-e.us-south.containers.cloud.ibm.com:30783 --token=EAVMH6YNi0BA88H3VO90v_WidUoNNPOtF3u4Tg`
  1. Test out command line connectivity to underlying OpenShift infrastructure:  `oc version`  or `oc get pods` You can also do much of this through the OpenShift Console
  1. While you can install the Cloud Paks into the default project, it's a better idea to put it in its own project or namespace.  These terms are linked.  Let's create a new project.  I'll call mine `zen` from the terminal while logged in issue `oc new-project zen`  This will create a zen project and you will use this project name when creating the Cloud Pak.  
  1.  If you are going to customize the Cloud Pak cluster with other services your will need to create a Route to the internal container registry.  These are your two commands
  - `oc create route reencrypt --service=docker-registry -n default`
  - `oc annotate route docker-registry --overwrite haproxy.router.openshift.io/balance=source -n default`
  - `oc get routes -n default`

### Quick run through of some OpenShift CLI commands  (Coming)
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

### Install your Cloud Pak of Choice
<!---  [Cloud Pak for Apps](apps.md) [Cloud pak for Automation](automation.md) --->
  - [Cloud Pak for Data](Data.md)
<!---  [Cloud Pak for Intergration](integration.md)[Cloud Pak for Mult-Cloud Manager](mcm.md) --->
