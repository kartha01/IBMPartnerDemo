# Installing Netezza on IBM Cloud
##Installation Node
### Read the instructions
1. Review the [installation instructions](https://www.ibm.com/support/knowledgecenter/SSTNZ3/com.ibm.ips.doc/postgresql/admin/adm_nps_cloud_ibm.html).  I will document how I did it and any gotchas.
### Create a bastion node (Linux VM)
1. I created a 2x4GB virtual server with CentOs and 100GB boot disk. This is used as an installation or Bastion Node.   I selected a place in the same Data Center as the OpenShift nodes.
1. `ibmcloud login`
~~~
Toms-MBP:~ tjm$ ibmcloud login
API endpoint: https://cloud.ibm.com
Region: us-east
~~~
1. Select the account you want to use from the account list.
~~~
Authenticating...
OK
Select an account:
1. Thomas McManus's Account (eeee7d19b6701916e21bf02f116813a9) <-> 1670487
2. IBM PoC - mactom Nz Cloud/CPD Cloud (f1af5bc92ad34287bbce250dfbe068a2) <-> 2134520
~~~
1. Run to verify you can execute a Virtual Server create using `--test`.  I am creating this as a Transient server which has hourly suspended billing.
  - **hostname**  `-H` ***bastion***,
  - **FQDN** `-D` is by default the account name with dashes for spaces or `\` and a suffix of `.cloud`.  `IBM PoC - mactom Nz Cloud/CPD Cloud` converts to `IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud`
  - **flavor** `--flavor` ***B1_2x4x100***   This is a Balanced 2 CPU, 4 GB RAM  100 GB Disk VM
  - **data center** `-d` ***dal13***   Chose the data center of your choice here  I am picking ***Dallas 013***
  - **Operating System** `-o` ***CentOS_8_64***  This is formed using the ***CentOS Version 8 64 bit architecture.***
  - **suspended billing** `--transient`
`ibmcloud sl vs create -H bastion -D IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud  --flavor  B1_2X4X100 -d dal13 -o CentOS_8_64 --transient -test`
~~~
Toms-MBP:~ tjm$ ibmcloud sl vs create -H bastion -D IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud  --flavor  B1_2X4X100 -d dal13 -o CentOS_8_64 --transient --test
OK
The order is correct.
~~~
1. Now remove the `--test` parameter to actually create the virtual server. You will have to enter ***y*** to approve the charges.
~~~
Toms-MBP:~ tjm$ ibmcloud sl vs create -H bastion -D IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud  --flavor  B1_2X4X100 -d dal13 -o CentOS_8_64 --transient
This action will incur charges on your account. Continue?> y
name                 value   
ID                   109019840   
FQDN                 bastion.IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud   
Created              2020-09-14T16:04:08Z   
GUID                 2370515c-83e2-4035-b4ff-30055e004bf1   
Placement Group ID   -   
~~~

### Log into the bastion node
1. Assuming you are logged into IBM Cloud, run a list of the all the virtual server to get the one you wish to log into.
~~~
Toms-MBP:~ tjm$ ibmcloud sl vs list
id         hostname   domain                                 cpu   memory  public_ip     private_ip    datacenter   action   
109019840  bastion   IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud  2     4096   52.116.5.56   10.208.89.253   dal13           
~~~
1. Get the details of this server.
~~~
Toms-MBP:~ tjm$ ibmcloud sl vs detail 109019840 --passwords
Name              Value   
ID                109019840   
guid              2370515c-83e2-4035-b4ff-30055e004bf1   
hostname          bastion   
domain            IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud   
fqdn              bastion.IBM-PoC-mactom-Nz-Cloud-CPD-Cloud.cloud   
status            Active   
state             Running   
datacenter        dal13   
os                CentOS   
os version        8.0-64 Minimal for VSI   
cpu cores         2   
memory            4096   
public ip         52.116.5.59   
private ip        10.208.89.253   
private network   false   
private cpu       false   
created           2020-09-14T16:04:08Z   
updated           2020-09-14T16:06:14Z   
owner             IBM2134520   
vlans             type      number   id      
                  PUBLIC    1677     2947048      
                  PRIVATE   857      2947050      
                  .
users             software   username   password      
                  CentOS     root       lSwz289yyPJN      
~~~
1.  Using ssh log into the bastion node.
~~~
Toms-MBP:~ tjm$ ssh root@52.116.5.59
The authenticity of host '52.116.5.56 (52.116.5.59)' can't be established.
ECDSA key fingerprint is SHA256:dcDOAYnDhHX+9fInZjBDcnHhPyNX69yRI0SKL3/6aZE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '52.116.5.59' (ECDSA) to the list of known hosts.
root@52.116.5.59's password:
~~~
1. Create `mkdir /root/nz`  This is where the client media will be executed.

### Add prereqs to the bastion node and test permissions on IBM Cloud account
1. Install the **Development Tools** to install ***unzip***, ***Python 3***, ***which*** and ***gettext***
 - `sudo yum groupinstall -y "Development Tools"`
1. Install ***jq***
  - `yum install epel-release -y`
  - `yum install jq -y`
  - `jq --version`
1. Install ***Python*** on RHEL 8.
  - `yum install python3`
  - `python3 --version`
1. I install IBM Cloud CLI to run a few sanity checks on my account prior to trying to install ***NZ CLoud***
  - Run `curl -sL https://ibm.biz/idt-installer | bash`
1. Log in through the CLI, using your IBM Cloud ID.
  - `ibmcloud login`
  - Select the account you want to use.
  - Select the region - in my case ***us-south***
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

### Get the installer
1. While you can go to Software Access catalog to get the installer, it may make more sense to .  These are only the client side or installer.
1. You can download this to your [laptop then move it to the bastion node](#move-the-installer-to-bastion-node).
1. You can get the [sftp details and proceed to pull down the nz-cloud client](#use-sftp-from-fix-central) directly to the bastion node.

### Move the installer to bastion node
1. Log into the newly provisioned VM
  - `ssh root@52.116.5.59`
  - Create a directory called to put the install media. I created ***/root/nz***.
1. From your laptop in a different terminal, move the installer to the bastion node.
~~~
Toms-MBP:nzcloud tjm$ scp nzcloud-linux-v11.1.0.0.tar.gz root@52.116.5.59:/root/nz
root@52.116.5.59's password:
nzcloud-linux-v11.1.0.0.tar.gz      100%   45MB  29.4MB/s   00:01    
~~~
### Use sftp from fix central
1. Go to [Fix Central](https://www.ibm.com/support/fixcentral/swg/selectFixes?product=ibm%2FInformation+Management%2FIBM+Netezza+for+Cloud+Pak+for+Data) to pull the latest binaries.
1. Check the most recent version, at this writing, I am using `11.1.1.0-IM-INZCPD-fp200831`
1. Click **continue** then click **I Agree**
1. Use the values from the ***Fix package location*** section.  These values change every time you go to fix central, so ignore the ones you see inline.
~~~
Fix package location
.
The following location information can be used to download the fix files.
.
FTPS/SFTP server:	delivery04-bld.dhe.ibm.com
User ID:	plieRnJu
Password:	ZAAHyZ8sCncjB6n
~~~
1. Move to the `/root/nz` and execute the sftp id@hostname from above.  
~~~
[root@bastion nz]# sftp LpctfUwD@delivery04-bld.dhe.ibm.com
The authenticity of host 'delivery04-bld.dhe.ibm.com (170.225.15.104)' can't be established.
RSA key fingerprint is SHA256:QRJpOHdTFuPmP2NLOQHTpB+IrDSNrque7RadzKcFyFc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'delivery04-bld.dhe.ibm.com,170.225.15.104' (RSA) to the list of known hosts.
plieRnJu@delivery04-bld.dhe.ibm.com's password:
Connected to plieRnJu@delivery04-bld.dhe.ibm.com.
sftp> mget *
Fetching /nzcloud-linux-v11.1.1.0.tar.gz to nzcloud-linux-v11.1.1.0.tar.gz
/nzcloud-linux v11.1.1.0.tar.gz     100%   45MB  20.3MB/s   00:02    
sftp> exit
[root@bastion nz]# ls
nzcloud-linux-v11.1.1.0.tar.gz
[root@bastion nz]#
~~~

### Installing the nz-cloud CLI
1. Back in the terminal that is logged into the newly minted VM, I move to ***/root/nz*** to gunzip the installer.
1. Add execute permissions to the file `chmod +x nzcloud-linux-v11.1.1.0.tar.gz `
1. Unpack the archive `tar -xzf nzcloud-linux-v11.1.1.0.tar.gz`
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
# Either zone or region can be specified
ZONE ${ZONE}
#MULTIZONE_REGION ${MULTIZONE_REGION}
APIKEY ${IBM_CLOUD_API_KEY}
#######################################
#   CONFIGURABLE PROPERTIES           #
#######################################
# OPTIONAL: custom subnet CIDR for pod private IP addresses
#IBM_POD_SUBNET ${IBM_POD_SUBNET}
# OPTIONAL: custom subnet CIDR for service private IP addresses
#IBM_SERVICE_SUBNET ${IBM_SERVICE_SUBNET}
# OPTIONAL: VLANs may be specified for single zone deployment
#PRIVATE_VLAN ${PRIVATE_VLAN}
#PUBLIC_VLAN ${PUBLIC_VLAN}
# OPTIONAL: Preferred resource group
#PREFERRED_RESOURCE_GROUP ${PREFERRED_RESOURCE_GROUP}
# OPTIONAL: VLANs may be specified for multizone deployment
#PRIVATE_VLAN_ZONE_1 ${PRIVATE_VLAN_ZONE_1}
#PUBLIC_VLAN_ZONE_1 ${PUBLIC_VLAN_ZONE_1}
#PRIVATE_VLAN_ZONE_2 ${PRIVATE_VLAN_ZONE_2}
#PUBLIC_VLAN_ZONE_2 ${PUBLIC_VLAN_ZONE_2}
#PRIVATE_VLAN_ZONE_3 ${PRIVATE_VLAN_ZONE_3}
#PUBLIC_VLAN_ZONE_3 ${PUBLIC_VLAN_ZONE_3}
~~~  

### Collect information for the ibm_infra.properties files
1. Open up the ***ibm_infra.properties*** file.  
1. Replace `${CLUSTER_NAME}` with a unique name.  I will use ***NZCluster***
1. Replace `${ZONE}` with your preferred zone.  I'll use ***dal13***
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
1. Next you will need to get your API Key. Either by browser or command line
#### Browser:
   1. [Click here](https://cloud.ibm.com/iam/apikeys) to create your apikey to used in the file. You man need to log into IBM Cloud account to retrieve this.
   1. Click ***Create an IBM Cloud API key +***
     - Provide a **name** and **description**.
     - **Click** ***Create***
     - For your sanity sake, download the file as you can use it to log in and it saves the apikey value to use another time.   You will use this value to replace `${IBM_CLOUD_API_KEY}`

#### Command line  
   1. Create via command line using `ibmcloud`
     - run `ibmcloud iam api-key-create netezza  -d nz-key --file nzakikey.json `
      ~~~
      [root@bastion nz]# ibmcloud iam api-key-create netezza  -d nz-key --file nzakikey.json
      Creating API key netezza under f1af5bc92ad34287bbce250dfbe068a2 as mactom@us.ibm.com...
      OK
      API key netezza was created
      Successfully save API key information to nzakikey.json
      ~~~
    -  `cat nzakikey.json` and use the value from apikey in the file.
      ~~~
      {
     	"id": "ApiKey-3e1cb4d5-849c-429e-a04c-8f7c1efc091a",
       ...
      "apikey": "SBZ1bF5dTC3mzyg3HNY3EChTk2Co-dp63HQTVjCcCp",
      ...
      "created_by": "IBMid-100000A84G",
      "modified_at": "2020-09-14T18:51+0000"
      ~~~
1. Get the ***Public and Private VLAN*** `ibmcloud oc vlan ls --zone  dal13`
  ~~~
  [root@nz-install nz-cloud]# ibmcloud oc vlan ls --zone  dal13
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
  ZONE dal13
  APIKEY SBZ1bF5dTC3mzyg3HNY3EChTk2Co-dp63HQTVjCcCp
  #######################################
  #   USER OPTIONAL INPUT PROPERTIES    #
  #######################################
  PRIVATE_VLAN 2942828
  PUBLIC_VLAN 2942826
  PREFERRED_RESOURCE_GROUP Default
  ~~~
1. Test out the configuration and check for permissions needed.
~~~
[root@nz-install nz-cloud]# ./nz-cloud show-permissions -p ibm_infra.properties
========================
Required Access Policies
========================
Service                          Role(s)                         
Kubernetes Service               Administrator (Platform access) and Manager (Service access)
Cloud Object Storage             Administrator
All Account Management Services  Administrator (if creating resource group) or Viewer (using existing resource group)
For more information: https://cloud.ibm.com/docs/openshift?topic=openshift-access_reference#cluster_create_permissions
~~~
1. Verify on IBM Cloud that you have these permissions.
~~~
[root@nz-install nz-cloud]# ibmcloud iam user-policies mactom@us.ibm.com
Getting policies of user mactom@us.ibm.com under current account as mactom@us.ibm.com...
OK              
Policy ID:   23d74da4-7473-49de-908c-ebc6b84f9963   
Roles:       Manager, Writer, Reader, Administrator, Editor, Operator, Viewer   
Resources:                        
             Service Name   schematics                   
Policy ID:   829dcb61-cbd5-4462-a6ed-88a04bed1c12   
Roles:       Manager, Writer, Reader, Administrator, Editor, Operator, Viewer   
Resources:            
....
....
....
Policy ID:   e9905a3a-c48c-4006-be80-bb57fd9f92c4   
Roles:       Administrator, Manager   
Resources:                        
             Service Type   All resources in account               
~~~
1.  For some reason, I need to have the following IBM Account access policy roles on my specific account, not the group
   - "Kubernetes Service (Requires Administrator on Platform access and Manager on Service access)"
   - "Cloud Object Storage (Requires Administrator)"

### Provision the Bare Metal Servers
Run the installer to create the Openshift cluster on bare metal nodes of ROKS (IBM CLOUD). Note: The script will exit with a message to wait for the bare metal nodes to be in a normal state. This process can take up to a day, which is why the install is separated into different parts..

### Provision OpenShift Cluster
1. As these are bare metal servers on IBM Cloud, this can take some time to provision.  Go get a coffee.
```
./nz-cloud -i ocp -p ibm_infra.properties -v
```
1. Once you see 3 bare metal servers 4x32 running, you will rerun this command again.
```
./nz-cloud -i ocp -p ibm_infra.properties -v
```
1.  This will make changes to your OCP cluster.  As you watch the script you will see nodes cordoned and uncordoned, pods evicted, etc.  This is all fine.  This could take 30 minutes.
1. upon completion you should have a `envs/<cluster name>/assets/oc_login_details` directory.  
### Provision the infrastructure for the NPS Host and SPUs
1. Before we progress with installing the NPS engine and SPUs, we will want to procure the hardware.  When you create these, the cluster name should match
  1. To be consistent run these exports: **NOTE:**  Alter these values for your configuration.
  ~~~
  export NAMESPACE_NAME=nz
  export CLUSTER=nzcluster
  export ZONE=dal13
  ~~~
  1. find the vlan for the zone. `ibmcloud oc vlan ls --zone ${ZONE}`
  ~~~
  [root@bastion nz-cloud]# ibmcloud oc vlan ls --zone ${ZONE}
  OK
  ID        Name   Number   Type      Router         Supports Virtual Workers   
  2947050          857      private   bcr01a.dal13   true   
  2947048          1677     public    fcr01a.dal13   true     
  ~~~
  1. Export the following based on the previously run command
  ~~~
  # this comes from the private vlan
  export PR_VLAN=2947050  
  # this comes from the public vlan
  export PUB_VLAN=2947048  
  ~~~
  1. Create a worker pool with 2 nodes.  This pool will be used for the NZ engines.  **Run** `ibmcloud oc worker-pool create classic --name ${NAMESPACE_NAME}_host --cluster ${CLUSTER} --flavor mb3c.16x64.encrypted --size-per-zone 2 --label nodetype=hostworker --label namespace=${NAMESPACE_NAME} --entitlement cloud_pak --hardware dedicated`
  ~~~
  [root@bastion nz-cloud]# ibmcloud oc worker-pool create classic --name ${NAMESPACE_NAME}_host --cluster ${CLUSTER} --flavor mb3c.16x64.encrypted --size-per-zone 2 --label nodetype=hostworker --label namespace=${NAMESPACE_NAME} --entitlement cloud_pak --hardware dedicated
  OK
  ~~~
  1. Create a worker pool with 3 nodes.  This pool will be used for the SPU engines.  **Run** `ibmcloud oc worker-pool create classic --name ${NAMESPACE_NAME}_spu --cluster ${CLUSTER} --flavor ms3c.16x64.1.9tb.ssd.encrypted --size-per-zone 3 --label nodetype=spuworker --label namespace=${NAMESPACE_NAME} --entitlement cloud_pak --hardware dedicated`
  ~~~
  [root@bastion nz-cloud]# ibmcloud oc worker-pool create classic --name ${NAMESPACE_NAME}_spu --cluster ${CLUSTER} --flavor ms3c.16x64.1.9tb.ssd.encrypted --size-per-zone 3 --label nodetype=spuworker --label namespace=${NAMESPACE_NAME} --entitlement cloud_pak --hardware dedicated
  OK
  ~~~
  1. Add nodes to the VLAN. **Run** `ibmcloud ks zone add classic --cluster ${CLUSTER} --zone ${ZONE} -p ${NAMESPACE_NAME}_host -p ${NAMESPACE_NAME}_spu --private-vlan ${PR_VLAN} --public-vlan ${PUB_VLAN}`
   ~~~
   [root@bastion nz-cloud]# ibmcloud ks zone add classic --cluster nzcluster --zone dal13 -p nz_host -p nz_spu --private-vlan 2947050 -- public-vlan 2947048
   OK
   ~~~

### Provision NPS
