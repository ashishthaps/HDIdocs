---
title: Install third-party Hadoop applications on Azure HDInsight | Microsoft Docs
description: Learn how to install third-party Hadoop applications on Azure HDInsight.
services: hdinsight
documentationcenter: ''
author: mumian
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.assetid: eaf5904d-41e2-4a5f-8bec-9dde069039c2
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 04/25/2017
ms.author: jgao

---
# Install third-party Hadoop applications on Azure HDInsight

In this article, you will learn how to install an already published third-party Hadoop application on Azure HDInsight. For instructions on installing your own application, see [Install custom HDInsight applications](hdinsight-apps-install-custom-applications.md).

## The HDInsight application platform

HDInsight is the only fully-managed cloud Hadoop offering that provides optimized open source analytical clusters for Spark, Hive, Interactive Hive, MapReduce, HBase, Storm, Kafka, and R Server backed by a 99.9% SLA. Each of these big data technologies are easily deployable as managed clusters with enterprise-level security and monitoring.

The ecosystem of applications in Big data has grown with the goal of making it easier for customers to solve their big data and analytical problems faster. Today, customers often find it challenging to discover these productivity applications and then in turn, struggle to install and configure these apps.

To address this gap, the HDInsight Application Platform provides an experience unique to HDInsight where Independent Software Vendors (ISVs) can directly offer their applications to customers - and customers can easily discover, install and use these applications built for the Big data ecosystem.

Installing HDInsight applications helps customers be more productive with the [range of available Hadoop cluster types](hdinsight-component-versioning). These solutions can span a variety of scenarios from data ingestion, data wrangling, monitoring, visualization, optimizing performance, security, analyzing, visualization, reporting, and many more. Applications can be developed by Microsoft, ISVs, or by yourself.

Most of the HDInsight applications are installed on an empty edge node. An empty edge node is a Linux virtual machine with the same client tools installed and configured as in the head node, but with no Hadoop services running. You can use the edge node for accessing the cluster, testing your client applications, and hosting your client applications. For more information, see [Use empty edge nodes in HDInsight](hdinsight-apps-use-edge-node).

HDInsight applications offer the following benefits:

**Ease of authoring, deploying, and management**

You can install these apps on an existing HDInsight cluster as well as during new cluster creation. 

**Native access to cluster**

The apps are installed on the Edge node and have access to the entire cluster.

**Ease of configuring the app on the cluster**

End users of these applications do not have to install or manage packages on each and every node and configure the application.

**Install solutions on existing HDInsight clusters**

Solution providers can make their solutions available to users who already have an HDInsight cluster running. This allows them to use these solutions easily and increase their productivity.

## About ISVs

ISVs, or Independent Software Vendors, are organizations who build products, provide custom development services, and serve as a valuable resource to Microsoft and our customers. We invite ISVs to leverage the capability to make it easier for customers to discover and use your solution through the [Azure Marketplace](hdinsight-apps-publish-applications). Please reach out to [hdipartners@microsoft.com](mailto:hdipartners@microsoft.com) if you would like to participate. 


## Published ISV applications

There are seven currently seven published applications. Click each for more information:

* [**DATAIKU DDS on HDInsight**](hdinsight-install-published-app-dataiku.md): Dataiku DSS (Data Science Studio) is a software that allows data professionals (data scientists, business analysts, developers...) to prototype, build, and deploy highly specific services that transform raw data into impactful business predictions.
* [**Datameer**](hdinsight-install-published-app-datameer.md): [Datameer](http://www.datameer.com/documentation/display/DAS50/Home?ls=Partners&lsd=Microsoft&c=Partners&cd=Microsoft) offers analysts an interactive way to discover, analyze, and visualize the results on Big Data. Pull in additional data sources easily to discover new relationships and get the answers you need quickly.
* [**Streamsets Data Collector for HDnsight**](hdinsight-install-published-app-streamsets.md) provides a full-featured integrated development environment (IDE) that lets you design, test, deploy, and manage any-to-any ingest pipelines that mesh stream and batch data, and include a variety of in-stream transformations—all without having to write custom code. 
* [**Cask CDAP 3.5/4.0/4.1 for HDInsight**](https://blogs.msdn.microsoft.com/azuredatalake/2016/10/17/using-cask-data-application-platform-on-azure-hdinsight/) provides the first unified integration platform for big data that cuts down the time to production for data applications and data lakes by 80%. This application only supports Standard HBase 3.4 clusters.
* [**H2O Artificial Intelligence for HDInsight (Beta)**](hdinsight-install-published-app-h2o.md) H2O Sparkling Water supports the following distributed algorithms: GLM, Naïve Bayes, Distributed Random Forest, Gradient Boosting Machine, Deep Neural Networks , Deep learning, K-means , PCA, Generalized Low Rank Models, Anomaly Detection, and Autoencoders.
* **AtScale** makes BI work on Hadoop. With AtScale, business users get interactive and multi-dimensional analysis capabilities, directly on Hadoop, at maximum speed, using the tools they already know, own and love – from Microsoft Excel to Tableau Software to QlikView.
* **Talena** ensures data resiliency in the event of disasters or corruption, enabling companies to get back online faster, by leveraging the power of machine learning. Talena backs up and recovers terabyte and petabyte-sized data sets and beyond 5-10 times faster than any other solution on the market, minimizing the impact of data loss associated with human and application errors and reducing downtime to minutes and hours, as opposed to days and weeks.


## How to install a published application

The instructions provided in this article use Azure portal. You can also export the Azure Resource Manager template from the portal or obtain a copy of the Resource Manager template from vendors, and use Azure PowerShell and Azure CLI to deploy the template.  See [Create Linux-based Hadoop clusters in HDInsight using Resource Manager templates](hdinsight-hadoop-create-linux-clusters-arm-templates.md).

## Prerequisites
All that's needed is an existing HDInsight cluster, or you can follow steps to [create an HDInsight cluster](hdinsight-hadoop-linux-tutorial-get-started.md#create-cluster).

## Install applications to existing clusters
The following procedure shows you how to install HDInsight applications to an existing HDInsight cluster.

**To install an HDInsight application**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Click **HDInsight Clusters** in the left menu.  If you don't see it, click **More Services**, and then click **HDInsight Clusters**.
3. Click an HDInsight cluster.  If you don't have one, you must create one first.  see [Create clusters](hdinsight-hadoop-linux-tutorial-get-started.md#create-cluster).
4. Click **Applications** under the **Configurations** category. You can see a list of installed applications if there are any. If you cannot find Applications, that means there is no applications for this version of the HDInsight cluster.
   
    ![HDInsight applications portal menu](./media/hdinsight-apps-install-applications/hdinsight-apps-portal-menu.png)
5. Click **Add** from the blade menu. 
   
    ![HDInsight applications installed apps](./media/hdinsight-apps-install-applications/hdinsight-apps-installed-apps.png)
   
    You can see a list of existing HDInsight applications.
   
    ![HDInsight applications available applications](./media/hdinsight-apps-install-applications/hdinsight-apps-list.png)
6. Click one of the applications, accept the legal terms, and then click **Select**.

You can see the installation status from the portal notifications (click the bell icon on the top of the portal). After the application is installed, the application will appear on the Installed Apps blade.

## Install applications during cluster creation
You have the option to install HDInsight applications when you create a cluster. During the process, HDInsight applications are installed after the cluster is created and is in the running state. The following procedure shows you how to install HDInsight applications when you create a cluster.

**To install an HDInsight application**

1. Sign in to the [Azure  portal](https://portal.azure.com).
2. Click **NEW**, Click **Data + Analytics**, and then click **HDInsight**.
3. Select **Custom (size, settings, apps)**.
4. Enter **Cluster Name**: This name must be globally unique.
5. Click **Subscription** to select the Azure subscription that will be used for the cluster.
6. Click **Select cluster Type**, and then select:
   
   * **Cluster Type**: If you don't know what to choose, select **Hadoop**. It is the most popular cluster type.
   * **Operating System**: Select **Linux**.
   * **Version**: Use the default version if you don't know what to choose. For more information, see [HDInsight cluster versions](hdinsight-component-versioning.md).
   * **Cluster Tier**: Azure HDInsight provides the big data cloud offerings in two categories: Standard tier and Premium tier. For more information, see [Cluster tiers](hdinsight-hadoop-provision-linux-clusters.md#cluster-tiers).
7. Click **Credentials** and then enter a password for the admin user. You must also enter an **SSH Username** and either a **PASSWORD** or **PUBLIC KEY**, which will be used to authenticate the SSH user. Using a public key is the recommended approach. Click **Select** at the bottom to save the credentials configuration.
8. Click **Resource Group** to select an existing resource group, or click **New** to create a new resource group.
9. Select the cluster **Location**.
10. Click **Next**.
11. Select your **Storage Account Settings**, selecting one of the existing storage accounts or creating a new one to be used as the default storage account for the cluster. In this blade, you may also select additional storage accounts as well as configure metastore settings to preserve your metadata outside of the cluster.
12. Click **Next**.
13. Search or browse from available applications in the **Applications** blade. Select one, accept the license agreement, then click **Next**.
14. Complete the remaining steps, such as cluster size and advanced settings. On the **Summary** blade, confirm your settings, and then click **Create**.

## List installed HDInsight apps and properties
The portal shows a list of the installed HDInsight applications for a cluster, and the properties of each installed application.

**To list HDInsight application and display properties**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Click **HDInsight Clusters** in the left menu.  If you don't see it, click **Browse**, and then click **HDInsight Clusters**.
3. Click an HDInsight cluster.
4. From the **Settings** blade, click **Applications** under the **General** category. The Installed Apps blade lists all the installed applications. 
   
    ![HDInsight applications installed apps](./media/hdinsight-apps-install-applications/hdinsight-apps-installed-apps-with-apps.png)
5. Click one of the installed applications to show the property. The property blade lists:
   
   * App name: application name.
   * Status: application status. 
   * Webpage: The URL of the web application that you have deployed to the edge node if there is any. The credential is the same as the HTTP user credentials that you have configured for the cluster.
   * HTTP endpoint: The credential is the same as the HTTP user credentials that you have configured for the cluster. 
   * SSH endpoint: You can use SSH to connect to the edge node. The SSH credentials are the same as the SSH user credentials that you have configured for the cluster. For information, see [Use SSH with HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).
6. To delete a application, right-click the application, and then click **Delete** from the context menu.

## Connect to the edge node
You can connect to the edge node using HTTP and SSH. The endpoint information can be found from the [portal](#list-installed-hdinsight-apps-and-properties). For information, see [Use SSH with HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

The HTTP endpoint credentials are the HTTP user credentials that you have configured for the HDInsight cluster; the SSH endpoint credentials are the SSH credentials that you have configured for the HDInsight cluster.

## Troubleshoot
See [Troubleshoot the installation](hdinsight-apps-install-custom-applications.md#troubleshoot-the-installation).

## Next steps
* [Install custom HDInsight applications](hdinsight-apps-install-custom-applications.md): learn how to deploy an un-published HDInsight application to HDInsight.
* [Publish HDInsight applications](hdinsight-apps-publish-applications.md): Learn how to publish your custom HDInsight applications to Azure Marketplace.
* [MSDN: Install an HDInsight application](https://msdn.microsoft.com/library/mt706515.aspx): Learn how to define HDInsight applications.
* [Customize Linux-based HDInsight clusters using Script Action](hdinsight-hadoop-customize-cluster-linux.md): learn how to use Script Action to install additional applications.
* [Create Linux-based Hadoop clusters in HDInsight using Resource Manager templates](hdinsight-hadoop-create-linux-clusters-arm-templates.md): learn how to call Resource Manager templates to create HDInsight clusters.
* [Use empty edge nodes in HDInsight](hdinsight-apps-use-edge-node.md): learn how to use an empty edge node for accessing HDInsight cluster, testing HDInsight applications, and hosting HDInsight applications.

