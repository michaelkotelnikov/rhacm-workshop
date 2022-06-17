# Exercise 2 - Managing an existing cluster using Advanced Cluster Management

In this exercise you manage the existing cluster on the Red Hat Advanced Cluster Management stack - `local-cluster`. You will attach labels to the cluster, visualize its resources and perform updates to the OpenShift Platform.


## 2.1 Import an existing cluster

1. Modify the attributes of the managed cluster in Red Hat Advanced Cluster Management -
*   **Name**: local-cluster
*   **labels**: 
    * environment=dev
    * owner=&lt;your-name>
    
In order to associate the labels with local-cluster, follow the next steps (You may use the presentation for guidance) -

*   Navigate to **Clusters** -> **local-cluster** -> **Actions** -> **Edit labels**.
*   Add the labels in the `key=value` format.

2. Log into the cluster using the **oc** cli tool.

```
<managed cluster> $ oc login -u <admin-user> -p <password> https://api.cluster.2222.sandbox.opentlc.com:6443
```

3. Make sure that all of the agent pods are up and running on the cluster.

```
<managed cluster> $ oc get pods -n open-cluster-management-agent
NAME                                         	READY   STATUS	RESTARTS   AGE
klusterlet-645d98d7d5-hnn2z                  	1/1 	Running   0      	46m
klusterlet-registration-agent-66fdc479cf-ltlx6   1/1 	Running   0      	46m
klusterlet-registration-agent-66fdc479cf-qnhzj   1/1 	Running   0      	46m
klusterlet-registration-agent-66fdc479cf-t8x5n   1/1 	Running   0      	46m
klusterlet-work-agent-6b8b99b899-27ht9       	1/1 	Running   0      	46m
klusterlet-work-agent-6b8b99b899-95dkr       	1/1 	Running   1      	46m
klusterlet-work-agent-6b8b99b899-vdp9r       	1/1 	Running   0      	46m

<managed cluster> $ oc get pods -n open-cluster-management-agent-addon
NAME                                           READY   STATUS    RESTARTS   AGE
application-manager-7c8879d57f-4x7ft           1/1     Running   0          24m
cert-policy-controller-7584887cdf-2vkv5        1/1     Running   0          24m
config-policy-controller-56d8d84c8c-p8z72      1/1     Running   0          24m
governance-policy-framework-65c46c46c8-xtgfq   2/2     Running   0          24m
iam-policy-controller-56b5bf6486-795wd         1/1     Running   0          24m
klusterlet-addon-workmgr-55bc5d4fd-2jp55       1/1     Running   0          24m
```

## 2.2 Analyzing the managed cluster

In this exercise you will be using the Red Hat Advanced Cluster Management portal to analyze the managed cluster’s resources. You may use the workshop presentation for examples and guidance.

1. Using Red Hat Advanced Cluster Management, find out what is the cloud provider of the managed cluster.
2. Using Red Hat Advanced Cluster Management, find out the number of nodes that make up the managed cluster. How many CPUs does each node have?
3. Using Red Hat Advanced Cluster Management, check out whether all users can provision new projects on local-cluster (check if the **self-provisioners** ClusterRoleBinding has the system:authenticated:oauth group associated with it).
4. Using Red Hat Advanced Cluster Management, check what **channel version** is associated with local-cluster (stable / candidate / fast) - (Search for **kind:ClusterVersion** CR).
5. Using Red Hat Advanced Cluster Management -
*   Check the port number that the **alertmanager-main-0** pod listens on local-cluster (can be found using the pod logs and pod resource definition).
*   Check the full path of the **alertmanager-main-0** pod configuration file (can be found using the pod logs and pod resource definition).


## 2.3 Upgrade the cluster using Advanced Cluster Management

**NOTE**: Do this exercise towards the end of the day. The upgrading process may take up to an hour to complete.

1. Change the **channel version** on the local-cluster from stable-**4.x** to stable-**4.x+1**.
2. Upgrade the cluster using Red Hat Advanced Cluster Management.
