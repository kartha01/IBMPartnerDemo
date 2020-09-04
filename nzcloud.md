# Installing Netezza on IBM Cloud


##Installation Node
1. I created a 2x4 virtual server with CentOs and 100GB boot disk. This is used as an installation or Bastion Node.   I selected a place in the same Data Center as the OpenShift nodes.
1. Review the [installation instructions](https://www.ibm.com/support/knowledgecenter/SSTNZ3/com.ibm.ips.doc/postgresql/admin/adm_nps_cloud_ibm.html).  I will document how I did it and any gotchas.
1. I will install jq, python, which, gettext and unzip.  These will be needed by the installer.
Also needed will be the installer for ***nzcloud***
