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

### Add prereqs to the bastion node and test permissions on IBM Cloud Account
1. I install IBM Cloud CLI to run a few sanity checks on my account prior to trying to install ***NZ CLoud***
 - Run `curl -sL https://ibm.biz/idt-installer | bash`
1. Log in through the CLI, using your IBM Cloud ID.
  - `ibmcloud login`
  - Select the account you want to use.
  - Select the region - in my case ***us-east***
  - Send usage statistics (up to you on the answer)
1. Let's check our Kubernetes infrastructure permissions.
 - Run `ibmcloud ks infra-permissions get`
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
nzcloud-linux-v11.1.0.0.tar.gz                                                                                                                             100%   45MB  29.4MB/s   00:01    
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



 pXfxxVdg0IhfD39uKQnisQ3sDr9zNO2Q5QsxC6EJFSAq
