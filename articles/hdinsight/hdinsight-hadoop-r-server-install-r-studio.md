---
title: Install RStudio with R Server on HDInsight - Azure | Microsoft Docs
description: How to install RStudio with R Server on HDInsight.
services: hdinsight
documentationcenter: ''
author: bradsev
manager: jhubbard
editor: cgronlun

ms.assetid: 918abb0d-8248-4bc5-98dc-089c0e007d49
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/19/2017
ms.author: bradsev

---
# Installing RStudio with R Server on HDInsight

This article describes how to install the community (free) version of [RStudio Server](https://www.rstudio.com/products/rstudio-server/) on the edge node of a cluster using a custom script. RStudio Server provides a browser-based IDE for use by remote clients and is widely used on Linux. There are multiple integrated development environments (IDEs) available for R today, including:

- Microsoft’s  [R Tools for Visual Studio](https://www.visualstudio.com/en-us/features/rtvs-vs.aspx) (RTVS) 
- [RStudio Server](https://www.rstudio.com/products/rstudio-server/) 
- Walware’s Eclipse-based [StatET](http://www.walware.de/goto/statet).

The advantage of installing RStudio Server on the edge node of an HDInsight cluster is that it provides a full IDE experience for the development and execution of R scripts with R Server on the cluster. This configuration can be considerably more productive than default use of the R Console.

> [!NOTE]
> The procedure described in this article is only relevant if you did not select to install RStudio Server community edition when provisioning your cluster. If you added it during provisioning, then to access it you click on the **R Server Dashboards** tile in the Azure portal entry for your cluster, then on the **R Studio Server** tile. 

If you prefer to use the commercially licensed Pro version of RStudio Server, you must follow the installation instructions from [RStudio Server](https://www.rstudio.com/products/rstudio/download-server/).

> [!NOTE]
> If you are using an HDInsight cluster for which R was installed using the [Install R Script Action](hdinsight-hadoop-r-scripts-linux.md), the steps in this document will not work correctly as they require an R Server on the HDInsight cluster.
>
> 

## Prerequisites

* An Azure HDInsight cluster with R Server installed. For instructions, see [Get started with R Server on HDInsight clusters](hdinsight-hadoop-r-server-get-started.md).
* An SSH client. For Linux and Unix distributions or Macintosh OS X, the `ssh` command is provided with the operating system. For Windows, we recommend [Cygwin](http://www.redhat.com/services/custom/cygwin/) with the [OpenSSH option](https://www.youtube.com/watch?v=CwYSvvGaiWU), or [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).  

## Install RStudio on the cluster using a custom script

Here are the steps:

1. Identify the edge node of the cluster. For an HDInsight cluster with R Server, following is the naming convention for head node and edge node.
   * Head node `CLUSTERNAME-ssh.azurehdinsight.net`
   * Edge node `CLUSTERNAME-ed-ssh.azurehdinsight.net` 

2. SSH into the edge node of the cluster using the naming pattern provided in step 1. For more information, see [Use SSH with HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

3. Once you are connected, become a root user on the cluster. In the SSH session, use the following command:

        sudo su -

4. Download the custom script to install RStudio. Use the following command:

        wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/InstallRStudio.sh

5. Change the permissions on the custom script file and run the script. Use the following commands:

        chmod 755 InstallRStudio.sh
        ./InstallRStudio.sh

6. If you used an SSH password while creating an HDInsight cluster with R Server, you can skip this step and proceed to the next. If you used an SSH key instead to create the cluster, you must set a password for your SSH user. You need this password when connecting to RStudio. Run the following commands:

        passwd USERNAME
        Current Kerberos password:
        New password:
        Retype new password:
        Current Kerberos password:


7. When prompted for **Current Kerberos password**, press **ENTER**.  Note that you must replace `USERNAME` with an SSH user for your HDInsight cluster. If your password is successfully set, you should see the following message:

        passwd: password updated successfully

	Exit the SSH session.

8. Create an SSH tunnel to the cluster by mapping `ssh -L localhost:8787:localhost:8787 USERNAME@CLUSTERNAME-ed-ssh.azurehdinsight.net` on the HDInsight cluster to the client machine. You must create an SSH tunnel before opening a new browser session.

   * On a Linux client or a Windows client with [Cygwin](http://www.redhat.com/services/custom/cygwin/), open a terminal session and use the following command:

             ssh -L localhost:8787:localhost:8787 USERNAME@CLUSTERNAME-ed-ssh.azurehdinsight.net

       Replace **USERNAME** with an SSH user for your HDInsight cluster, and replace **CLUSTERNAME** with the name of your HDInsight cluster.
       You can also use an SSH key rather than a password by adding `-i id_rsa_key`.        
   * If using a Windows client with PuTTY then

     1. Open PuTTY, and enter your connection information.
     2. In the **Category** section to the left of the dialog, expand **Connection**, expand **SSH**, and then select **Tunnels**.
     3. Provide the following information on the **Options controlling SSH port forwarding** form:

        * **Source port** - The port on the client that you wish to forward. For example, **8787**.
        * **Destination** - The destination that must be mapped to the local client machine. For example, **localhost:8787**.

        	![Create an SSH tunnel](./media/hdinsight-hadoop-r-server-install-r-studio/createsshtunnel.png "Create an SSH tunnel")

     4. Click **Add** to add the settings, and then click **Open** to open an SSH connection.
     5. When prompted, log in to the server to establish an SSH session and enable the tunnel.

9. Open a web browser and enter the following URL based on the port you entered for the tunnel:

        http://localhost:8787/ 

10. You are prompted to enter the SSH username and password to connect to the cluster. If you used an SSH key while creating the cluster, you must enter the password you created in step 5.

    ![Connect to R Studio](./media/hdinsight-hadoop-r-server-install-r-studio/connecttostudio.png "Create an SSH tunnel")

11. To test whether the RStudio installation was successful, you can run a test script that executes R-based MapReduce and Spark jobs on the cluster. To download the test script to run in RStudio, go back to the SSH console and enter the following commands:

	*    If you created a Hadoop cluster with R, use this command:

        	wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi.r
	*    If you created a Spark cluster with R, use this command:

        	wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi_spark.r

12. In RStudio, you see the test script you downloaded. Double-click the file to open it, select the contents of the file, and then click **Run**. You should see the output in the **Console** pane:

   ![Test the installation](./media/hdinsight-hadoop-r-server-install-r-studio/test-r-script.png "Test the installation")

Another option would be to type `source(testhdi.r)` or `source(testhdi_spark.r)` to execute the script.

## See also

* [Compute context options for R Server on HDInsight clusters](hdinsight-hadoop-r-server-compute-contexts.md)
* [Azure Storage options for R Server on HDInsight](hdinsight-hadoop-r-server-storage.md)

