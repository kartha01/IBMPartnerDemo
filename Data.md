# Install instructions for Cloud Pak for data

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
1. Scroll down and select the services that you desire to add to Cloud Pak for Data
1. On the right, you should see your entitlements and a yellow box, which will turn green after you run the script below.
1. Check the box and to accept the license.   

<iframe width="600" height="322" src="https://www.youtube.com/embed/Ic0xnlci47o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
