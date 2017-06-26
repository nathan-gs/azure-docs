---
title: Use Hadoop Oozie workflows in Linux-based HDInsight | Microsoft Docs
description: Use Hadoop Oozie in Linux-based HDInsight. Learn how to define an Oozie workflow, and submit an Oozie job.
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.assetid: d7603471-5076-43d1-8b9a-dbc4e366ce5d
ms.service: hdinsight
ms.custom: hdinsightactive
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2017
ms.author: larryfr

---
# Use Oozie with Hadoop to define and run a workflow on Linux-based HDInsight

[!INCLUDE [oozie-selector](../../includes/hdinsight-oozie-selector.md)]

Learn how to use Apache Oozie with Hadoop on HDInsight. Apache Oozie is a workflow/coordination system that manages Hadoop jobs. It is integrated with the Hadoop stack, and it supports Hadoop jobs for Apache MapReduce, Apache Pig, Apache Hive, and Apache Sqoop. It can also be used to schedule jobs that are specific to a system, like Java programs or shell scripts

> [!NOTE]
> Another option for defining workflows with HDInsight is Azure Data Factory. To learn more about Azure Data Factory, see [Use Pig and Hive with Data Factory][azure-data-factory-pig-hive].

> [!IMPORTANT]
> Oozie is not enabled on domain-joined HDInsight.

## Prerequisites

* **An HDInsight cluster**: See [Get Started with HDInsight on Linux](hdinsight-hadoop-linux-tutorial-get-started.md)

  > [!IMPORTANT]
  > The steps in this document require an HDInsight cluster that uses Linux. Linux is the only operating system used on HDInsight version 3.4 or greater. For more information, see [HDInsight retirement on Windows](hdinsight-component-versioning.md#hdi-version-33-nearing-retirement-date).

## Example workflow

The workflow used in this document contains two actions. Actions are definitions for tasks, such as running Hive, Sqoop, MapReduce, or other process:

![Workflow diagram][img-workflow-diagram]

1. A Hive action runs a HiveQL script to extract records from the **hivesampletable** included with HDInsight. Each row of data describes a visit from a specific mobile device. The record format appears similar to the following text:

        8       18:54:20        en-US   Android Samsung SCH-i500        California     United States    13.9204007      0       0
        23      19:19:44        en-US   Android HTC     Incredible      Pennsylvania   United States    NULL    0       0
        23      19:19:46        en-US   Android HTC     Incredible      Pennsylvania   United States    1.4757422       0       1

    The Hive script used in this document counts the total visits for each platform (such as Android or iPhone) and stores the counts to a new Hive table.

    For more information about Hive, see [Use Hive with HDInsight][hdinsight-use-hive].

2. A Sqoop action exports the contents of the new Hive table to a table in an Azure SQL database. For more information about Sqoop, see [Use Hadoop Sqoop with HDInsight][hdinsight-use-sqoop].

> [!NOTE]
> For supported Oozie versions on HDInsight clusters, see [What's new in the Hadoop cluster versions provided by HDInsight][hdinsight-versions].

## Create the working directory

Oozie expects resources required for a job to be stored in the same directory. This example uses **wasbs:///tutorials/useoozie**. Use the following command to create this directory, and the data directory that holds the new Hive table created by this workflow:

```
hdfs dfs -mkdir -p /tutorials/useoozie/data
```

> [!NOTE]
> The `-p` parameter causes all directories in the path to be created. The **data** directory is used to hold data used by the **useooziewf.hql** script.

Also run the following command, which ensures that Oozie can impersonate your user account when running Hive and Sqoop jobs. Replace **USERNAME** with your login name:

```
sudo adduser USERNAME users
```

> [!NOTE]
> You can ignore errors that the user is already a member of the `users` group.

## Add a database driver

Since this workflow uses Sqoop to export data to SQL Database, you must provide a copy of the JDBC driver used to talk to SQL Database. Use the following command to copy it to the working directory:

```
hdfs dfs -put /usr/share/java/sqljdbc_4.1/enu/sqljdbc*.jar /tutorials/useoozie/
```

If your workflow used other resources, such as a jar containing a MapReduce application, you would need to add these resources as well.

## Define the Hive query

Use the following steps to create a HiveQL script that defines a query, which is used in an Oozie workflow later in this document.

1. Connect to the cluster using SSH. The following command is an example of using the `ssh` command. Replace __USERNAME__ with the SSH user for the cluster. Replace __CLUSTERNAME__ with the name of the HDInsight cluster.

    ```
    ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    ```

    For more information, see [Use SSH with HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. From the SSH connection, use the following command to create a file:

    ```
    nano useooziewf.hql
    ```

3. Once the nano editor opens, use the following query as the contents of the file:

    ```hiveql
    DROP TABLE ${hiveTableName};
    CREATE EXTERNAL TABLE ${hiveTableName}(deviceplatform string, count string) ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
    INSERT OVERWRITE TABLE ${hiveTableName} SELECT deviceplatform, COUNT(*) as count FROM hivesampletable GROUP BY deviceplatform;
    ```

    There are two variables used in the script:

    * **${hiveTableName}**: contains the name of the table to be created

    * **${hiveDataFolder}**: contains the location to store the data files for the table

    The workflow definition file (workflow.xml in this tutorial) passes these values to this HiveQL script at run time

4. To exit the editor, press Ctrl-X. When prompted, select **Y** to save the file, then use **Enter** to use the **useooziewf.hql** file name.

5. Use the following commands to copy **useooziewf.hql** to **wasbs:///tutorials/useoozie/useooziewf.hql**:

    ```
    hdfs dfs -put useooziewf.hql /tutorials/useoozie/useooziewf.hql
    ```

    These commands store the **useooziewf.hql** file on the Azure Storage account associated with this cluster, which preserves the file even if the cluster is deleted.

## Define the workflow

Oozie workflows definitions are written in hPDL (an XML Process Definition Language). Use the following steps to define the workflow:

1. Use the following statement to create and edit a new file:

    ```
    nano workflow.xml
    ```

2. Once the nano editor opens, enter the following XML as the file contents:

    ```xml
    <workflow-app name="useooziewf" xmlns="uri:oozie:workflow:0.2">
        <start to = "RunHiveScript"/>
        <action name="RunHiveScript">
        <hive xmlns="uri:oozie:hive-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.job.queue.name</name>
                <value>${queueName}</value>
            </property>
            </configuration>
            <script>${hiveScript}</script>
            <param>hiveTableName=${hiveTableName}</param>
            <param>hiveDataFolder=${hiveDataFolder}</param>
        </hive>
        <ok to="RunSqoopExport"/>
        <error to="fail"/>
        </action>
        <action name="RunSqoopExport">
        <sqoop xmlns="uri:oozie:sqoop-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.compress.map.output</name>
                <value>true</value>
            </property>
            </configuration>
            <arg>export</arg>
            <arg>--connect</arg>
            <arg>${sqlDatabaseConnectionString}</arg>
            <arg>--table</arg>
            <arg>${sqlDatabaseTableName}</arg>
            <arg>--export-dir</arg>
            <arg>${hiveDataFolder}</arg>
            <arg>-m</arg>
            <arg>1</arg>
            <arg>--input-fields-terminated-by</arg>
            <arg>"\t"</arg>
            <archive>sqljdbc41.jar</archive>
            </sqoop>
        <ok to="end"/>
        <error to="fail"/>
        </action>
        <kill name="fail">
        <message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
        </kill>
        <end name="end"/>
    </workflow-app>
    ```

    There are two actions defined in the workflow:

   * **RunHiveScript**: This action is the start action, and runs the **useooziewf.hql** Hive script

   * **RunSqoopExport**: This action exports the data created from the Hive script to SQL Database using Sqoop. This action only runs if the **RunHiveScript** action is successful.

     The workflow has several entries, such as `${jobTracker}`. These entries are replaced by values you use in the job definition. The job definition is created later in this document.

     Also note the `<archive>sqljdbc4.jar</arcive>` entry in the Sqoop section. This entry instructs Oozie to make this archive available for Sqoop when this action runs.

3. Use Ctrl-X, then **Y** and **Enter** to save the file.

4. Use the following command to copy the **workflow.xml** file to **/tutorials/useoozie/workflow.xml**:

    ```
    hdfs dfs -put workflow.xml /tutorials/useoozie/workflow.xml
    ```

## Create the database

To create an Azure SQL Database, follow the steps in the [Create a SQL Database](../sql-database/sql-database-get-started.md) document. When creating the database, use `oozietest` as the database name. Also make a note of the name of the database server.

### Create the table

> [!NOTE]
> There are many ways to connect to SQL Database to create a table. The following steps use [FreeTDS](http://www.freetds.org/) from the HDInsight cluster.


1. Use the following command to install FreeTDS on the HDInsight cluster:

    ```
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

2. Once FreeTDS has been installed, use the following command to connect to the SQL Database server you created previously:

    ```
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <sqlLogin> -P <sqlPassword> -p 1433 -D oozietest
    ```

    You receive output similar to the following text:

        locale is "en_US.UTF-8"
        locale charset is "UTF-8"
        using default charset "UTF-8"
        Default database being set to oozietest
        1>

3. At the `1>` prompt, enter the following lines:

    ```
    CREATE TABLE [dbo].[mobiledata](
    [deviceplatform] [nvarchar](50),
    [count] [bigint])
    GO
    CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(deviceplatform)
    GO
    ```

    When the `GO` statement is entered, the previous statements are evaluated. These statements create a table named **mobiledata** that is used by the workflow.

    Use the following to verify that the table has been created:

    ```
    SELECT * FROM information_schema.tables
    GO
    ```

    You see output similar to the following text:

    ```
    TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
    oozietest       dbo     mobiledata      BASE TABLE
    ```

4. Enter `exit` at the `1>` prompt to exit the tsql utility.

## Create the job definition

The job definition describes where to find the workflow.xml. It also describes where to find other files used by the workflow (such as useooziewf.hql.) It also defines the values for properties used within the workflow and associated files.

1. Use the following command to get the full address of the default storage. This address is used in the configuration file in a moment:

    ```
    sed -n '/<name>fs.default/,/<\/value>/p' /etc/hadoop/conf/core-site.xml
    ```

    This command returns information similar to the following XML:

    ```xml
    <name>fs.defaultFS</name>
    <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net</value>
    ```

    > [!NOTE]
    > If the HDInsight cluster uses Azure Storage as the default storage, the `<value>` element contents begin with `wasbs://`. If Azure Data Lake Store is used instead, it begins with `adl://`.

    Save the contents of the `<value>` element, as it is used in the next steps.

2. Use the following command to get FQDN of the cluster headnode. This information is used for the JobTracker address for the cluster:

    ```
    hostname -f
    ```

    This returns information similar to the following text:

    ```hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net```

    The port used for the JobTracker is 8050, so the full address to use for the JobTracker is `hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:8050`.

3. Use the following to create the Oozie job definition configuration:

    ```
    nano job.xml
    ```

4. Once the nano editor opens, use the following XML as the contents of the file:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>

        <property>
        <name>nameNode</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net</value>
        </property>

        <property>
        <name>jobTracker</name>
        <value>JOBTRACKERADDRESS</value>
        </property>

        <property>
        <name>queueName</name>
        <value>default</value>
        </property>

        <property>
        <name>oozie.use.system.libpath</name>
        <value>true</value>
        </property>

        <property>
        <name>hiveScript</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/useooziewf.hql</value>
        </property>

        <property>
        <name>hiveTableName</name>
        <value>mobilecount</value>
        </property>

        <property>
        <name>hiveDataFolder</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/data</value>
        </property>

        <property>
        <name>sqlDatabaseConnectionString</name>
        <value>"jdbc:sqlserver://serverName.database.windows.net;user=adminLogin;password=adminPassword;database=oozietest"</value>
        </property>

        <property>
        <name>sqlDatabaseTableName</name>
        <value>mobiledata</value>
        </property>

        <property>
        <name>user.name</name>
        <value>YourName</value>
        </property>

        <property>
        <name>oozie.wf.application.path</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
    </configuration>
    ```

   * Replace all instances of **wasbs://mycontainer@mystorageaccount.blob.core.windows.net** with the value you received earlier for default storage.

     > [!WARNING]
     > If the path is a `wasb` path, you must use the full path. Do not shorten it to just `wasb:///`.

   * Replace **JOBTRACKERADDRESS** with the JobTracker/ResourceManager address you received earlier.
   * Replace **YourName** with your login name for the HDInsight cluster.
   * Replace **serverName**, **adminLogin**, and **adminPassword** with the information for your Azure SQL Database.

     Most of the information in this file is used to populate the values used in the workflow.xml or ooziewf.hql files (such as ${nameNode}.)

     > [!NOTE]
     > The **oozie.wf.application.path** entry defines where to find the workflow.xml file, which contains the workflow ran by this job.

5. Use Ctrl-X, then **Y** and **Enter** to save the file.

## Submit and manage the job

The following steps use the Oozie command to submit and manage Oozie workflows on the cluster. The Oozie command is a friendly interface over the [Oozie REST API](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html).

> [!IMPORTANT]
> When using the Oozie command, you must use the FQDN for the HDInsight headnode. This FQDN is only accessible from the cluster, or if the cluster is on an Azure Virtual Network, from other machines on the same network.


1. Use the following to obtain the URL to the Oozie service:

    ```
    sed -n '/<name>oozie.base.url/,/<\/value>/p' /etc/oozie/conf/oozie-site.xml
    ```

    This returns information similar to the following XML:

    ```xml
    <name>oozie.base.url</name>
    <value>http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:11000/oozie</value>
    ```

    The `http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:11000/oozie` portion is the URL to use with the Oozie command.

2. Use the following to create an environment variable for the URL, so you don't have to type it for every command:

    ```
    export OOZIE_URL=http://HOSTNAMEt:11000/oozie
    ```

    Replace the URL with the one you received earlier.
3. Use the following to submit the job:

    ```
    oozie job -config job.xml -submit
    ```

    This command loads the job information from **job.xml** and submits it to Oozie, but does not run it.

    Once the command completes, it should return the ID of the job. For example, `0000005-150622124850154-oozie-oozi-W`. This ID is used to manage the job.

4. View the status of the job using the following command:

    ```
    oozie job -info <JOBID>
    ```

    > [!NOTE]
    > Replace `<JOBID>` with the ID returned in the previous step.

    This returns information similar to the following text:

    ```
    Job ID : 0000005-150622124850154-oozie-oozi-W
    ------------------------------------------------------------------------------------------------------------------------------------
    Workflow Name : useooziewf
    App Path      : wasbs:///tutorials/useoozie
    Status        : PREP
    Run           : 0
    User          : USERNAME
    Group         : -
    Created       : 2015-06-22 15:06 GMT
    Started       : -
    Last Modified : 2015-06-22 15:06 GMT
    Ended         : -
    CoordAction ID: -
    ------------------------------------------------------------------------------------------------------------------------------------
    ```

    This job has a status of `PREP`. This indicates that it was submitted, but has not been started yet.

5. Use the following command to start the job:

    ```
    oozie job -start JOBID
    ```

    > [!NOTE]
    > Replace `<JOBID>` with the ID returned previously.

    If you check the status after this command, it is in a running state, and information is returned for the actions within the job.

6. Once the task completes successfully, you can verify that the data was generated and exported to the SQL Database table by using the following commands:

    ```
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -P <adminPassword> -p 1433 -D oozietest
    ```

    At the `1>` prompt, enter the following query:

    ```
    SELECT * FROM mobiledata
    GO
    ```

    The information returned is similar to the following text:

        deviceplatform  count
        Android 31591
        iPhone OS       22731
        proprietary development 3
        RIM OS  3464
        Unknown 213
        Windows Phone   1791
        (6 rows affected)

For more information on the Oozie command, see [Oozie Command Line Tool](https://oozie.apache.org/docs/4.1.0/DG_CommandLineTool.html).

## Oozie REST API

The Oozie REST API allows you to build your own tools that work with Oozie. The following are HDInsight specific information about using the Oozie REST API:

* **URI**: The REST API can be accessed from outside the cluster at `https://CLUSTERNAME.azurehdinsight.net/oozie`

* **Authentication**: Authenticate to the API using the cluster HTTP account (admin) and password. For example:

    ```
    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/oozie/versions
    ```

For more information on using the Oozie REST API, see [Oozie Web Services API](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html).

## Oozie Web UI

The Oozie Web UI provides a web-based view into the status of Oozie jobs on the cluster. The web UI allows you to view the following:

* Job status
* Job definition
* Configuration
* A graph of the actions in the job
* Logs for the job

You can also view details for actions within a job.

To access the Oozie Web UI, use the following steps:

1. Create an SSH tunnel to the HDInsight cluster. For information, see the [Use SSH Tunneling with HDInsight](hdinsight-linux-ambari-ssh-tunnel.md) document.

2. Once a tunnel has been created, open the Ambari web UI in your web browser. The URI for the Ambari site is **https://CLUSTERNAME.azurehdinsight.net**. Replace **CLUSTERNAME** with the name of your Linux-based HDInsight cluster.

3. From the left side of the page, select **Oozie**, then **Quick Links**, and finally **Oozie Web UI**.

    ![image of the menus](./media/hdinsight-use-oozie-linux-mac/ooziewebuisteps.png)

4. The Oozie Web UI defaults to displaying running Workflow Jobs. To see all workflow jobs, select **All Jobs**.

    ![All jobs displayed](./media/hdinsight-use-oozie-linux-mac/ooziejobs.png)

5. Select a job to view more information about the job.

    ![Job info](./media/hdinsight-use-oozie-linux-mac/jobinfo.png)

6. From the Job Info tab, you can see basic job information and the individual actions within the job. Using the tabs at the top you can view the Job Definition, Job Configuration, access the Job Log or view a Directed Acyclic Graph (DAG) of the job.

   * **Job Log**: Select the **GetLogs** button to get all logs for the job, or use the **Enter Search Filter** field to filter logs

       ![Job log](./media/hdinsight-use-oozie-linux-mac/joblog.png)

   * **JobDAG**: The DAG is a graphical overview of the data paths taken through the workflow

       ![Job DAG](./media/hdinsight-use-oozie-linux-mac/jobdag.png)

7. Selecting one of the actions from the **Job Info** tab brings up information for the action. For example, select the **RunHiveScript** action.

    ![Action info](./media/hdinsight-use-oozie-linux-mac/action.png)

8. You can see details for the action, such as a link to the **Console URL**. This link can be used to view JobTracker information for the job.

## Scheduling jobs

The coordinator allows you to specify a start, end, and occurrence frequency for jobs so that they can be scheduled for certain times.

To define a schedule for the workflow, use the following steps:

1. Use the following to create a file named **coordinator.xml**:

    ```
    nano coordinator.xml
    ```

    Use the following XML as the contents of the file:

    ```xml
    <coordinator-app name="my_coord_app" frequency="${coordFrequency}" start="${coordStart}" end="${coordEnd}" timezone="${coordTimezone}" xmlns="uri:oozie:coordinator:0.4">
        <action>
        <workflow>
            <app-path>${workflowPath}</app-path>
        </workflow>
        </action>
    </coordinator-app>
    ```

    > [!NOTE]
    > The `${...}` variables are replaced by values in the job definition at run-time. The variables are:
    >
    > * `${coordFrequency}`: Time between running instances of the job.
    > ** `${coordStart}`: The job start time.
    > * `${coordEnd}`: The job end time.
    > * `${coordTimezone}`: Coordinator jobs are in a fixed time zone with no daylight savings time (typically represented by using UTC). This time zone is referred as the "Oozie processing timezone."
    > * `${wfPath}`: The path to the workflow.xml.

2. To save the file, use Ctrl-X, **Y**, and **Enter**.

3. Use the following command to copy the file to the working directory for this job:

    ```
    hadoop fs -put coordinator.xml /tutorials/useoozie/coordinator.xml
    ```

4. Use the following to modify the **job.xml** file:

    ```
    nano job.xml
    ```

    Make the following changes:

   * Change `<name>oozie.wf.application.path</name>` to `<name>oozie.coord.application.path</name>`. This value instructs Oozie to run the coordinator file instead of the workflow file.

   * Add the following XML. This sets a variable used in the coordinator.xml to point to the location of the workflow.xml:

        ```xml
        <property>
            <name>workflowPath</name>
            <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
        ```

       Replace the `wasbs://mycontainer@mystorageaccount.blob.core.windows` text with the value used in other entries in the job.xml file.

   * Add the following XML. This defines the start, end, and frequency to use for the coordinator.xml file:

        ```xml
        <property>
            <name>coordStart</name>
            <value>2017-05-10T12:00Z</value>
        </property>

        <property>
            <name>coordEnd</name>
            <value>2017-05-12T12:00Z</value>
        </property>

        <property>
            <name>coordFrequency</name>
            <value>1440</value>
        </property>

        <property>
            <name>coordTimezone</name>
            <value>UTC</value>
        </property>
        ```

       These values set the start time to 12:00PM on May 10th, 2017, the end time to May 12th, 2017. The interval for running this job daily. The frequency is in minutes, so 24 hours x 60 minutes = 1440 minutes. Finally, the timezone is set to UTC.

5. Use Ctrl-X, then **Y** and **Enter** to save the file.

6. To run the job, use the following command:

    ```
    oozie job -config job.xml -run
    ```

    This command submits and starts the job.

7. If you visit the Oozie Web UI and select the **Coordinator Jobs** tab, you see information similar to the following image:

    ![coordinator jobs tab](./media/hdinsight-use-oozie-linux-mac/coordinatorjob.png)

    The **Next Materialization** entry contains the next time that the job runs.

8. Similar to the earlier workflow job, selecting the job entry in the web UI displays information on the job:

    ![Coordinator job info](./media/hdinsight-use-oozie-linux-mac/coordinatorjobinfo.png)

    > [!NOTE]
    > This only shows successful runs of the job, not individual actions within the scheduled workflow. To see that, select one of the **Action** entries.

    ![Action info](./media/hdinsight-use-oozie-linux-mac/coordinatoractionjob.png)

## Troubleshooting

The Oozie UI allows you to view Oozie logs. It also contains links to JobTracker logs for MapReduce tasks started by the workflow. The pattern for troubleshooting should be:

1. View the job in Oozie Web UI.

2. If there is an error or failure for a specific action, select the action to see if the **Error Message** field provides more information on the failure.

3. If available, use the URL from the action to view more details (such as JobTracker logs) for the action.

The following are specific errors you may encounter, and how to resolve them.

### JA009: Cannot initialize cluster

**Symptoms**: The job status changes to **SUSPENDED**. Details for the job show the RunHiveScript status as **START_MANUAL**. Selecting the action displays the following error message:

    JA009: Cannot initialize Cluster. Please check your configuration for map

**Cause**: The WASB addresses used in the **job.xml** file do not contain the storage container or storage account name. The WASB address format must be `wasbs://containername@storageaccountname.blob.core.windows.net`.

**Resolution**: Change the WASB addresses used by the job.

### JA002: Oozie is not allowed to impersonate &lt;USER>

**Symptoms**: The job status changes to **SUSPENDED**. Details for the job show the RunHiveScript status as **START_MANUAL**. Selecting the action shows the following error message:

    JA002: User: oozie is not allowed to impersonate <USER>

**Cause**: Current permission settings do not allow Oozie to impersonate the specified user account.

**Resolution**: Oozie is allowed to impersonate users in the **users** group. Use the `groups USERNAME` to see the groups that the user account is a member of. If the user is not a member of the **users** group, use the following command to add the user to the group:

    sudo adduser USERNAME users

> [!NOTE]
> It may take several minutes before HDInsight recognizes that the user has been added to the group.

### Launcher ERROR (Sqoop)

**Symptoms**: The job status changes to **KILLED**. Details for the job show the RunSqoopExport status as **ERROR**. Selecting the action shows the following error message:

    Launcher ERROR, reason: Main class [org.apache.oozie.action.hadoop.SqoopMain], exit code [1]

**Cause**: Sqoop is unable to load the database driver required to access the database.

**Resolution**: When using Sqoop from an Oozie job, you must include the database driver with the other resources (such as the workflow.xml) used by the job. Also Reference the archive containing the database driver from the `<sqoop>...</sqoop>` section of the workflow.xml.

For example, for the job in this document, you would use the following steps:

1. Copy the sqljdbc4.1.jar file to the /tutorials/useoozie directory:

    ```
    hdfs dfs -put /usr/share/java/sqljdbc_4.1/enu/sqljdbc41.jar /tutorials/useoozie/sqljdbc41.jar
    ```

2. Modify the workflow.xml to add the following XML on a new line above `</sqoop>`:

    ```xml
    <archive>sqljdbc41.jar</archive>
    ```

## Next steps

In this tutorial, you learned how to define an Oozie workflow and how to run an Oozie job. To learn more about working with HDInsight, see the following articles:

* [Use time-based Oozie Coordinator with HDInsight][hdinsight-oozie-coordinator-time]
* [Upload data for Hadoop jobs in HDInsight][hdinsight-upload-data]
* [Use Sqoop with Hadoop in HDInsight][hdinsight-use-sqoop]
* [Use Hive with Hadoop on HDInsight][hdinsight-use-hive]
* [Use Pig with Hadoop on HDInsight][hdinsight-use-pig]
* [Develop Java MapReduce programs for HDInsight][hdinsight-develop-mapreduce]

[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563
[azure-data-factory-pig-hive]: ../data-factory/data-factory-data-transformation-activities.md
[hdinsight-oozie-coordinator-time]: hdinsight-use-oozie-coordinator-time.md
[hdinsight-versions]:  hdinsight-component-versioning.md
[hdinsight-storage]: hdinsight-use-blob-storage.md
[hdinsight-get-started]: hdinsight-get-started.md
[hdinsight-use-sqoop]: hdinsight-use-sqoop-mac-linux.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-pig]: hdinsight-use-pig.md
[hdinsight-storage]: hdinsight-use-blob-storage.md
[hdinsight-get-started-emulator]: hdinsight-get-started-emulator.md
[hdinsight-develop-mapreduce]: hdinsight-develop-deploy-java-mapreduce-linux.md

[sqldatabase-create-configue]: sql-database-create-configure.md
[sqldatabase-get-started]: sql-database-get-started.md

[azure-create-storageaccount]: storage-create-storage-account.md

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: http://azure.microsoft.com/downloads/
[powershell-about-profiles]: http://go.microsoft.com/fwlink/?LinkID=113729
[powershell-install-configure]: /powershell/azureps-cmdlets-docs
[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/en-us/library/ee176961.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.Preparation.Output1.png
[img-runworkflow-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.RunWF.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
