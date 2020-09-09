# Installing Netezza on IBM Cloud
##Installation Node
### Read the instructions
1. Review the [installation instructions](https://www.ibm.com/support/knowledgecenter/SSTNZ3/com.ibm.ips.doc/postgresql/admin/adm_nps_cloud_ibm.html).  I will document how I did it and any gotchas.
### Get the installer
1. Get the installer for ***nzcloud***.  This is in [PartnerWorld Software Access Catalog](https://www.ibm.com/partnerworld/program/benefits/software-access-catalog).
 - Log in using your IBM ID that is linked to your PartnerWorld Account.    
 - Enter ***CC72JEN*** into the **Find by part number** edit box, then download this ***nzcloud-linux-v11.1.0.0.tar.gz*** file.
### Create a bastion node (Linux VM)
1. I created a 2x4GB virtual server with CentOs and 100GB boot disk. This is used as an installation or Bastion Node.   I selected a place in the same Data Center as the OpenShift nodes.

### Add prereqs to the bastion node and test permissions on IBM Cloud account
1. I install IBM Cloud CLI to run a few sanity checks on my account prior to trying to install ***NZ CLoud***
 - Run `curl -sL https://ibm.biz/idt-installer | bash`
1. Log in through the CLI, using your IBM Cloud ID.
  - `ibmcloud login`
  - Select the account you want to use.
  - Select the region - in my case ***us-east***
  - Send usage statistics (up to you on the answer)
1. Let's check our Kubernetes infrastructure permissions.
 - Run `ibmcloud oc infra-permissions get`
 - In my case below, I should have proper permissions to create an OpenShift cluster.
  ~~~
  [root@nz-install ~]#  ibmcloud oc infra-permissions get
  Analyzing the infrastructure permissions for resource group ...
  Retrieved suggested and required infrastructure permissions that are missing for account ID 21#####6 with infrastructure access set up by linked account API key.
  OK
  Missing Virtual Worker Permissions
  No changes are suggested or required.
  Missing Physical Worker Permissions
  No changes are suggested or required.
  Missing Network Permissions
  No changes are suggested or required.
  Missing Storage Permissions
  No changes are suggested or required.
  To set infrastructure permissions, see 'https://ibm.biz/infra-permission'
  ~~~
1. Install the **Development Tools** to install ***unzip***, ***Python 3***, ***which*** and ***gettext***
 - `sudo yum groupinstall -y "Development Tools"`
1. Install ***jq***
  - `yum install epel-release -y`
  - `yum install jq -y`
  - `jq --version`

### Move the installer to bastion node
1. Log into the newly provisioned VM
  - `ssh root@169.60.72.67`
  - Create a directory called to put the install media. I created ***/root/nz***.
1. From your laptop in a different terminal, move the installer to the bastion node.
~~~
Toms-MBP:nzcloud tjm$ scp nzcloud-linux-v11.1.0.0.tar.gz root@169.60.72.67:/root/nz
root@169.60.72.67's password:
nzcloud-linux-v11.1.0.0.tar.gz      100%   45MB  29.4MB/s   00:01    
~~~
1. Back in the terminal that is logged into the newly minted VM, I move to ***/root/nz*** to gunzip the installer.
1. Add execute permissions to the file `chmod +x nzcloud-linux-v11.1.0.0.tar.gz `
1. Unpack the archive `tar -xzf nzcloud-linux-v11.1.0.0.tar.gz`
1. Move to the ***nz-cloud*** directory
 - `cd nz-cloud`
1. Copy the ***ips_ibmcloud_infra.properties.template*** to ***ibm_infra.properties***
 - `cp ips_ibmcloud_infra.properties.template ibm_infra.properties`
1. Review the new files
~~~
[root@nz-install nz-cloud]# cat ibm_infra.properties
#######################################
#   USER REQUIRED INPUT PROPERTIES    #
#######################################
CLOUD_PROVIDER IBM
CLUSTER_NAME ${CLUSTER_NAME}
ZONE ${ZONE}
APIKEY ${IBM_CLOUD_API_KEY}
#######################################
#   USER OPTIONAL INPUT PROPERTIES    #
#######################################
PRIVATE_VLAN ${PRIVATE_VLAN}
PUBLIC_VLAN ${PUBLIC_VLAN}
PREFERRED_RESOURCE_GROUP ${PREFERRED_RESOURCE_GROUP}
~~~  

### Collect information for the ibm_infra.properties files
1. Open up the ***ibm_infra.properties*** file.  
1. Replace `${CLUSTER_NAME}` with a unique name.  I will use ***NZCluster***
1. Replace `${ZONE}` with your preferred zone.  I'll use ***wdc06***
  - find the available zone using `ibmcloud oc zone ls --provider classic`   
  - If you are not sure what these symbols stand for, look up **locations**.  Anything with a ***†*** is a **multi-zone** `ibmcloud oc locations --provider classic`   
    **Note:** The listing of the locations is random and not consistent in order.   This list is subject to change as new data centers on line all the time.
  ~~~
  [root@nz-install nz-cloud]# ibmcloud oc locations --provider classic
  Classic Infrastructure Zones
  Zone    Metro                   Country               Geography   
  hkg02   Hong Kong (hkg-mtr)     Hong Kong (hkg)       Asia Pacific (ap)   
  syd05   Sydney (syd)†           Australia (au)        Asia Pacific (ap)   
  tok02   Tokyo (tok)†            Japan (jp)            Asia Pacific (ap)   
  dal12   Dallas (dal)†           United States (us)    North America (na)   
  mon01   Montreal (mon)          Canada (ca)           North America (na)   
  fra02   Frankfurt (fra)†        Germany (de)          Europe (eu)   
  fra05   Frankfurt (fra)†        Germany (de)          Europe (eu)   
  lon02   London (lon)†           United Kingdom (uk)   Europe (eu)   
  seo01   Seoul (seo)             Korea (kr)            Asia Pacific (ap)   
  wdc06   Washington DC (wdc)†    United States (us)    North America (na)   
  wdc07   Washington DC (wdc)†    United States (us)    North America (na)   
  tok04   Tokyo (tok)†            Japan (jp)            Asia Pacific (ap)   
  mex01   Mexico City (mex-cty)   Mexico (mex)          North America (na)   
  lon04   London (lon)†           United Kingdom (uk)   Europe (eu)   
  tok05   Tokyo (tok)†            Japan (jp)            Asia Pacific (ap)   
  lon06   London (lon)†           United Kingdom (uk)   Europe (eu)   
  tor01   Toronto (tor)           Canada (ca)           North America (na)   
  che01   Chennai (che)           India (in)            Asia Pacific (ap)   
  hou02   Houston (hou)           United States (us)    North America (na)   
  osl01   Oslo (osl)              Norway (no)           Europe (eu)   
  syd04   Sydney (syd)†           Australia (au)        Asia Pacific (ap)   
  fra04   Frankfurt (fra)†        Germany (de)          Europe (eu)   
  ams03   Amsterdam (ams)         Netherlands (nl)      Europe (eu)   
  syd01   Sydney (syd)†           Australia (au)        Asia Pacific (ap)   
  dal13   Dallas (dal)†           United States (us)    North America (na)   
  mil01   Milan (mil)             Italy (it)            Europe (eu)   
  dal10   Dallas (dal)†           United States (us)    North America (na)   
  mel01   Melbourne (mel)         Australia (au)        Asia Pacific (ap)   
  sjc03   San Jose (sjc)          United States (us)    North America (na)   
  sjc04   San Jose (sjc)          United States (us)    North America (na)   
  sng01   Singapore (sng-mtr)     Singapore (sng)       Asia Pacific (ap)   
  lon05   London (lon)†           United Kingdom (uk)   Europe (eu)   
  par01   Paris (par)             France (fr)           Europe (eu)   
  sao01   Sao Paulo (sao)         Brazil (br)           South America (sa)   
  wdc04   Washington DC (wdc)†    United States (us)    North America (na)   
  ~~~
1. Next you will need to get your API Key.
1. [Click here](https://cloud.ibm.com/iam/apikeys) to create your apikey to used in the file. You man need to log into IBM Cloud account to retrieve this.
1. Click ***Create an IBM Cloud API key +***
  - Provide a **name** and **description**.
  - **Click** ***Create***
  - For your sanity sake, download the file as you can use it to log in and it saves the apikey value to use another time.   You will use this value to replace `${IBM_CLOUD_API_KEY}`
1. Get the ***Public and Private VLAN*** `ibmcloud oc vlan ls --zone  wdc06`
  ~~~
  [root@nz-install nz-cloud]# ibmcloud oc vlan ls --zone  wdc06
  OK
  ID        Name   Number   Type      Router         Supports Virtual Workers   
  2942828          1250     private   bcr01a.wdc06   true   
  2942826          1315     public    fcr01a.wdc06   true   
  ~~~
1. Find out the resource group you want to assign NZ Cloud.  It is easier to control if you use Resource groups.  In my case, I will use ***default***
  ~~~
  [root@nz-install nz-cloud]# ibmcloud resource groups
  Retrieving all resource groups under account 10c85c1177aa4ae2959a936b46334bc1 as mactom@us.ibm.com...
  OK
  Name      ID                                 Default Group   State   
  Default   a1083ff7fbd547f390d2b0f4c8735231   true            ACTIVE   
  ~~~
1. Edit the `ibm_infra.properties` file and add your selected values.   
  ~~~
  #######################################
  #   USER REQUIRED INPUT PROPERTIES    #
  #######################################
  CLOUD_PROVIDER IBM
  CLUSTER_NAME nzcluster
  ZONE wdc06
  APIKEY EwZheUTyL7kyzP4tuQDgD39sSqLDkHn900nrTZC2b
  #######################################
  #   USER OPTIONAL INPUT PROPERTIES    #
  #######################################
  PRIVATE_VLAN 2942828
  PUBLIC_VLAN 2942826
  PREFERRED_RESOURCE_GROUP Default
  ~~~

### Provision the Bare Metal Servers
Run the installer to create the Openshift cluster on bare metal nodes of ROKS (IBM CLOUD). Note: The script will exit with a message to wait for the bare metal nodes to be in a normal state. This process can take up to a day, which is why the install is separated into different parts..

```
./nz-cloud -p ibm_infra.properties -i ocp -v
```
or
```
./nz-cloud -p ibm_infra.properties -i all -v
```

### Provision ocp

### Provision CPD

### Provision NPS
