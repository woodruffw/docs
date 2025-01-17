---
title: Evacuating a cluster node
intro: You can evacuate data services on a cluster node.
redirect_from:
  - /enterprise/admin/clustering/evacuating-a-cluster-node
  - /enterprise/admin/enterprise-management/evacuating-a-cluster-node
  - /admin/enterprise-management/evacuating-a-cluster-node
versions:
  ghes: '*'
type: how_to
topics:
  - Clustering
  - Enterprise
---

## About evacuation of cluster nodes

In a cluster configuration for {% data variables.product.product_name %}, you can evacuate a node before taking the node offline. Evacuation ensures that the remaining nodes in a service tier contain all of the service's data. For example, when you replace the virtual machine for a node in your cluster, you should first evacuate the node.

For more information about nodes and service tiers for {% data variables.product.prodname_ghe_server %}, see "[AUTOTITLE](/admin/enterprise-management/configuring-clustering/about-cluster-nodes)."

{% warning %}

**Warnings**:

- To avoid data loss, {% data variables.product.company_short %} strongly recommends that you evacuate a node before taking the node offline. 

- If you only have three nodes in your data services cluster, you can't evacuate the nodes because `ghe-spokes` doesn't have another place to make a copy. If you have four or more, `ghe-spokes` will move all the repositories off of the evacuated node.

{% endwarning %}

## Evacuating a cluster node

If you plan to take a node offline and the node runs a data service role like `git-server`, `pages-server`, or `storage-server`, evacuate each node before taking the node offline.

{% data reusables.enterprise_clustering.ssh-to-a-node %}
1. To find the UUID of the node to evacuate, run the following command. Replace `HOSTNAME` with the node's hostname.

   ```shell
   $ ghe-config cluster.HOSTNAME.uuid
   ```
1. Monitor the node's status while {% data variables.product.product_name %} copies the data. Don't take the node offline until the copy is complete. To monitor the status of your node, run any of the following commands, replacing `UUID` with the UUID from step 2.

   - **Git**:

     ```shell
     $ ghe-spokes evac-status git-server-UUID
     ```

   - **{% data variables.product.prodname_pages %}**:

     ```shell
     $ echo "select count(*) from pages_replicas where host = 'pages-server-UUID'" | ghe-dbconsole -y
     ```

   - **Storage**:

     ```shell
     $ ghe-storage evacuation-status storage-server-UUID
     ```
1. After the copy is complete, you can evacuate the node by running any of the following commands, replacing `UUID` with the UUID from step 2.

   - **Git**:

     ```shell
     $ ghe-spokes server evacuate git-server-UUID \'REASON FOR EVACUATION\'
     ```

   - **{% data variables.product.prodname_pages %}**:

     ```shell
     $ ghe-dpages evacuate pages-server-UUID
     ```

   - For **storage**, first take the node offline by running the following command.

     ```shell
     $ ghe-storage offline storage-server-UUID
     ```

     After the storage node is offline, you can evacuate the node by running the following command.

     ```shell
     $ ghe-storage evacuate storage-server-UUID
     ```
