# Installation Node
- [Create a bastion node (Linux VM)](#create-a-bastion-node-linux-vm)
- [Log into the bastion node](#log-into-the-bastion-node)
- [Move the installer to bastion node](#move-the-installer-to-bastion-node)
## Create a bastion node (Linux VM)
1. I created a 2x4GB virtual server with CentOs and 100GB boot disk. This is used as an installation or Bastion Node.   I selected a place in the same Data Center as the OpenShift nodes.
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

[Back to Table of Contents](bastion.md)

## Log into the bastion node
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

[Back to Table of Contents](bastion.md)

## Move the installer to bastion node
1. Log into the newly provisioned VM
  - `ssh root@52.116.5.59`
  - Create a directory called to put the install media. I created ***/root/nz***.
1. From your laptop in a different terminal, move the installer to the bastion node.
~~~
Toms-MBP:nzcloud tjm$ scp nzcloud-linux-v11.1.0.0.tar.gz root@52.116.5.59:/root/nz
root@52.116.5.59's password:
nzcloud-linux-v11.1.0.0.tar.gz      100%   45MB  29.4MB/s   00:01    
~~~
1. Proceed to the [installation instructions for NZ CLoud](nzcloud.md).

[Back to Table of Contents](bastion.md)
