# Install instructions for Cloud Pak for Data

## Provision the Control plane or lite assembly
1. Click Catalog
1. Click Software
1. In the search box enter Cloud Pak and press enter.  This should list the available Cloud Paks that can be provisioned. In these instructions, you are going to install Cloud Pak for Data.
1. Click on the Cloud Pak for Data tile.
1. Click on the Readme tab to review what you can provision using Schematics (Terraform).

## Building the configuration
1. On the create tab, this is where we will build the configuration of the Cloud Pak for Data clusters
1. First section describes the minimum configuration.  For Cloud Pak for Data it is as follows.  `Each cluster must meet a set of minimum requirements: 3 nodes with 16 cores, 64GB memory, and 25GB disk per node.`
1. Select the cluster that was previously created.  Where you created project `zen`.
1. Select the project.   While you can pick Default, you would be better served picking the previously create project like `zen`.
1. Rename the workspace to something you will remember like `CPDDemo-HealthCare` for a health care demo.
1. Pick your resource group, if you did not create one, just use `Default`.
1. You can tag your environment for easy search.  This is not required.
1. Scroll down and select Run Script if you do not have permissions, you can share a link and have an administrator execute this.
1. Scroll down and select the services that you desire to add to Cloud Pak for Data. *Note* This is a first time set up action.  Currently, toggling true then false will lead to no further action.  You will need to uninstall or install again, which will be shown later.
1. On the right, you should see your entitlements and a yellow box, which will turn green after you run the script below.
1. Check the box and to accept the license.   

<iframe width="600" height="322" src="https://www.youtube.com/embed/Ic0xnlci47o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Walk around Cloud Pak for Data
1. From the Schematics workspace, click `Offering dashboard`
1. Depending on your browser, you will need to accept the certificate.  General path is Advanced then accept risk.  This will bring you to the login page. Bookmark this for future usage.
1. Login using default credentials as seen in video.
1. Along the black menu bar at the top the icon to the right on the magnifying glass is "Services".  Click on this link.  
1. Take a look at the services that are `enabled`.   These have been installed or deployed.   You should see 5 services.  AI (Watson Studio, Watson Machine Learning, Watson OpenScale), Data Governance (Watson Knowledge Catalog) and Data Sources (Data Virtualization).  
1. Some are set up automatically and some you need to provision.  If you fo to the Data Virtualization tile, you will see you need to "Provision Instance".  
1. Let's provision the instance.  Click on "Provision Instance".
1. then click "Configure" for this Cloud Pak for Data on ROKS, the semaphores are already configured, so make sure to uncheck the box to the left of `Set up semaphore parameters for Data Virtualization automatically`, otherwise the provision can fail.
1. Once complete, you will see Data Virtualization under the Collect area on the left table of contents.  From here you can add data sources or pointers to file folders.
1.  If you pick Watson Studio or Watson Machine Learning tiles, there will be no actions to take.   
1. For Watson OpenScale and Watson Knowledge Catalog, there is an `Open` button which launch a UI to help configure the initial information.

## adding additional services.

export NAMESPACE=zen
export STORAGE_CLASS=ibmc-file-gold-gid

./cpd-linux adm --repo ../repo.yaml  --namespace $NAMESPACE --apply --accept-all-licenses --assembly <assembly name>
