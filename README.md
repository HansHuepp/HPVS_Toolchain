## Introduction
IBM Cloud Continuous Delivery includes Delivery Pipelines to build, test, and deploy in a repeatable way with minimal human intervention. In a pipeline, sequences of stages retrieve input and run jobs, such as builds, tests, and deployments. Usually the IBM Delivery Pipeline is used to deploy builds on Kubernetes or Cloud Foundry.

This Page explains how to deploy with the IBM Delivery Pipelines on custom servers. The following example will demonstrate this by using the demo application AcmeAir. Its code will be automatically integrated, dockerized on a custom build-server, uploaded to a private container registry and deployed on multiple custom servers (in this case the IBM Hyper Protect Virtuel Servers). This will all be done while using only the resources given by a free IBM Cloud Account.

## Prerequisites 
To follow this tutorial, you will need the following:

* A computer to customise the Toolchain running Docker                               (https://docs.docker.com/install/)

* A free IBM Cloud Account (https://cloud.ibm.com/login)

* A configured IBM Cloud Container Registry (https://cloud.ibm.com/kubernetes/registry/main/start)
   
## Estimated time
Around one hour

## Setting up your Hyper Protect Virtual Servers
First we create the server needed to deploy our software. Setting up a Hyper Protect Virtual Server is really simple. Just open the Command Line Interface of your computer and generate a SSH-Key-Pair by entering the command ssh-keygen (you might need to install open-ssh bevor doing this). Now you can use the generated Private-Key to create your Hyper Protect Virtual Servers [here](https://cloud.ibm.com/catalog/services/ibm-cloud-hyper-protect-virtual-servers).

This process is explained in more detail under:
* https://cloud.ibm.com/docs/services/hp-virtual-servers?topic=hp-virtual-servers-overview
* https://www.ssh.com/ssh/keygen/#sec-Creating-an-SSH-Key-Pair-for-User-Authentication

## Enabling SSH Key login for your IBM Delivery Pipeline
To enable the Toolchain to deploy on custom servers we use a custom docker image running linux, which stores the SSH-Keys. It will be used to send commands onto your server. More on this later.

To generate this docker image download the sshimage folder from [this](https://github.com/HansHuepp/HyperProtectVirtualServer_Toolchain) git repository to your desktop.
Now use Shift+Command+G in Finder (on Mac)or Windows+R on your Desktop (Windows) to open the **Go to Folder** search mask. Enter `~/.ssh/` or `/.ssh/`.
A folder should open with three text files called id_rsa, id_rsa.pub and known_hosts inside.
Copy this three files in the sshimage folder on your desktop.

Open the command line interface and navigate into this folder. Run the command `docker build .`. Docker creates an image with a small linux version and your keys in place. With the command `docker image ls` you should be able to see your newly created image. 

Next you have to 
tag (`docker tag <local_image> us.icr.io/<my_namespace>/<my_repo>:<my_tag>`) and 
upload (`docker push us.icr.io/<my_namespace>/<my_repo>:<my_tag>`) 
your image to the the IBM Container Registry.

More details: https://cloud.ibm.com/kubernetes/registry/main/start

## Creating the Pipeline Part 1
Go to  https://cloud.ibm.com/devops/ and create a  Cloud Foundry Toolchain.
Link the Git Repository with the code you want to use and enter an API-Key (if you don't have one you can generate one).
After creating your Toolchain you should see a dashboard where you can configure multiple elements.
Klick the field with the label **Delivery Pipeline**.
You will see the two stages of your Pipeline.

Klick on the gear in the right corner of the **BUILD** Stage, then klick on **Configure Stage**.
Choose the Custom Docker Image as your **Builder type**. Enter the name of your sshimage form your Cloud Container Registry.
Now klick on **ADD JOB** and select **Deploy**.
Choose the Custom Docker Image as your **Builder type**. Enter the name of your sshimage form your Cloud Container Registry.
Now open the tap **Environment properties**. Add a secure property with the name **DOCKER_USERNAME** and the value **iamapikey**. Add a secure property with the name **DOCKER_PASSWORD** and an API-Key as your value (You can generate one [here](https://cloud.ibm.com/iam/apikeys)). Add a text property with the name **SERVER1** and the IP-Address of your Hyper Protect Server as its value.

The following steps are specific to the nodejs version of [AcmeAir](https://github.com/acmeair/acmeair-nodejs) but it should be easy to alter these steps for different Programms.
Go back to **Jobs** and open your **Build Stage** and enter the following Codelines in the Build Script window:

`ssh -t -t $SERVER_1 'rm -r acmeair-nodejs'`

`ssh -t -t $SERVER_1 'git clone https://github.com/acmeair/acmeair-nodejs.git'`

`ssh -t -t $SERVER_1 'cd acmeair-nodejs && docker build -t acmeair/web .'`

Go back to Jobs and your Deploy Stage and enter the following Codelines in the Build Script window:

`ssh -t -t $SERVER_1 'docker login -u iamapikey -p $DOCKER_PASSWORD uk.icr.io'`

`ssh -t -t $SERVER_1 'docker tag acmeair/web uk.icr.io/hansdocker/acmeair'`

`ssh -t -t $SERVER_1 'docker push uk.icr.io/hansdocker/acmeair'`

Now go back to your Toolchain overview and press the play button on your Build Stage to see if your toolchain is working. After it finished you should see the sshimage in form of a docker image in your IBM Container Registry.

## Creating the Pipeline Part 2

Go back to the overview of your pipeline and klick on the gear in the right corner of the **DEPLOY Stage**, then klick on **Configure Stage**.

Now open the tap **Environment properties**. Add a secure property with the name DOCKER_USERNAME and the value iamapikey. Add a secure property with the name DOCKER_PASSWORD and an API-Key as your value. Add a text property with the name SERVER1 and the IP-Address of your Hyper Protect Server as its value.

Go back to the JOBS tap.
Remove the **Rolling Deploy** stage. Create four new Deploy Stages and name them Prepare, Deploy, Availability Test and Info. Set all four to use the custom docker image **sshimge**, just like you did in part one. Now we set the code for these four stages.

For **Prepare** enter the following script code:

`ssh -t -t root@$SERVER 'docker stop acmeair_web_001'`

`ssh -t -t root@$SERVER 'docker rm acmeair_web_001'`

`ssh -t -t root@$SERVER 'docker ps'`

For **Deploy** enter the following script code:

`ssh -t -t root@$SERVER 'docker login -u iamapikey -p $DOCKER_PASSWORD uk.icr.io'`

`ssh -t -t root@$SERVER 'docker pull uk.icr.io/hansdocker/acmeair'`

`ssh -t -t root@$SERVER 'docker run -d -P -p 9080:9080 --name acmeair_web_001 --link mongo_001:mongo uk.icr.io/hansdocker/acmeair'`

For **Availability Test** enter the following script code:

`#!/bin/bash`

`if curl -ivs  http://$SERVER:9080`

`then echo "Server is UP"`

`else`

`echo "Server is down" 1>&2`

`fi`

For **Info** enter the following script code:

`ssh -t -t root@$SERVER 'docker ps'`

`echo http://$SERVER:9080`

Now go back to your Toolchain overview and press the play button on your Deploy Stage to see if your toolchain works.

After it finished you should be able to klick on Info in the list of your stages to see your logs. There will be a link to the AcmeAir website running on your Hyper Protect Virtual Server.

You can easily add more servers by simply duplicating the Deploy Stage and changing the IP address in the Environment properties to your new Server.

This Video shows the function of the code an the toolchain running: https://www.youtube.com/watch?v=CiP9Lvk5480&feature=youtu.be

## More information
If you want to know more about the IBM Delivery Pipeline and Toolchain you can find more information [here](https://cloud.ibm.com/docs/services/ContinuousDelivery?topic=ContinuousDelivery-deliverypipeline_about)
