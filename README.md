# HyperProtectVirtualServer_Toolchain
IBM Cloud Toolchain Solution for deploying a demo application on an IBM Hyper Protect Virtual Server or other custom Servers


IBM Cloud Continuous Delivery includes Delivery Pipelines to build, test, and deploy in a repeatable way with minimal human intervention. In a pipeline, sequences of stages retrieve input and run jobs, such as builds, tests, and deployments. Usually the IBM Delivery Pipeline is used to deploy builds on Kubernetes or Cloud Foundry.

This Page explains how to deploy with the IBM Delivery Pipelines on custom servers. The following example will demonstrate this by using the demo application AcmeAir. Its code will be automatically integrated, dockerized on a custom build-server, uploaded to a private container registry and deployed on multiple custom servers (In this case the IBM Hyper Protect Virtuel Servers). This will all be done while using only the Resources given by a free IBM Cloud Account.

* [Prerequisites](https://github.com/HansHuepp/HPVS_Toolchain/wiki/Prerequisites)

* [Setting up your Hyper Protect Virtual Servers](https://github.com/HansHuepp/HPVS_Toolchain/wiki/Setting-up-your-Hyper-Protect-Virtual-Servers)

* [Enabling SSH Key login for your IBM Delivery Pipeline](https://github.com/HansHuepp/HPVS_Toolchain/wiki/Enabling-SSH-Key-login-for-your-IBM-Delivery-Pipeline)

* [Creating the Pipeline Part 1](https://github.com/HansHuepp/HPVS_Toolchain/wiki/Creating-the-Pipeline-Part-1)

* [Creating the Pipeline Part 2](https://github.com/HansHuepp/HPVS_Toolchain/wiki/Creating-the-Pipeline-Part-2)

* [More information](https://github.com/HansHuepp/HPVS_Toolchain/wiki/More-Information)
