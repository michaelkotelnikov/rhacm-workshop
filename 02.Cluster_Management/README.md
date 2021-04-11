# Exercise 2 - Managing an existing cluster using Advanced Cluster Management

In this exercise you will import an existing cluster to Red Hat Advanced Cluster Management stack. You will attach labels to the cluster, visualize its resources and perform updates to the OpenShift Platform.


## 2.1 Import an existing cluster

1. Import an existing cluster to Red Hat Advanced Cluster Management. Use the [workshop’s presentation](https://docs.google.com/presentation/d/1jhsO7tSsFoNwYouF-iBnlw5kA7Nx0Hoi-4HK0fFP1YI/edit?usp=sharing) as a guideline to the process -
*   **Name**: cluster-a
*   **Additional labels**: 
    * environment=dev
    * owner=&lt;your-name>
2. Log into the managed cluster using the **oc** cli tool.

```
<managed cluster> $ oc login -u admin -p <password> https://api.cluster.2222.sandbox.opentlc.com:6443
```

3. Make sure that all of the agent pods are up and running on the managed cluster.

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
NAME                                                  READY   STATUS RESTARTS   AGE
klusterlet-addon-appmgr-64695b8d7b-q7jdg               2/2 	Running   0      	44m
klusterlet-addon-certpolicyctrl-75df48646-j5zpc        2/2 	Running   0      	44m
klusterlet-addon-iampolicyctrl-6867d45954-xv9c2        2/2 	Running   0      	44m
klusterlet-addon-operator-f85df9c7f-lb2cq              1/1 	Running   0      	45m
klusterlet-addon-policyctrl-config-policy-6fd8d88f5-chjls 1/1 	Running   0  44m
klusterlet-addon-policyctrl-framework-7544f4678-kwbz9  4/4 	Running   0      	44m
klusterlet-addon-search-7b697f899f-5vqwl               2/2 	Running   0      	44m
klusterlet-addon-workmgr-7f48869cd6-mqq8g               2/2 	Running   0      	44m
```

4. The cluster should now be available.


## 2.2 Analyzing the managed cluster

In this exercise you will be using the Red Hat Advanced Cluster Management portal to analyze the managed cluster’s resources.

1. Using Red Hat Advanced Cluster Management, find out what is the cloud provider of the managed cluster.
2. Using Red Hat Advanced Cluster Management, find out the number of nodes that make up the managed cluster. How many CPUs does each node have?
3. Using Red Hat Advanced Cluster Management, check out whether all users can provision new projects on cluster-a (check if the **self-provisioners** ClusterRoleBinding has the system:authenticated:oauth group associated with it).
4. Using Red Hat Advanced Cluster Management, check what **channel version** is associated with cluster-a (stable / candidate / fast) - (Search for **kind:ClusterVersion** CR).
5. Using Red Hat Advanced Cluster Management -
*   Check the port number that the **alertmanager-main-0** pod listens on cluster-a (can be found using the pod logs and pod resource definition).
*   Check the full path of the **alertmanager-main-0** pod configuration file (can be found using the pod logs and pod resource definition).


## 2.3 Upgrade cluster-a 

**NOTE**: Do this exercise after you are finished with the **whole** workshop. The upgrading process may take up to an hour to complete.

1. Change the **channel version** on the cluster-a from stable-**4.4** to stable-**4.5**.
2. Upgrade the cluster using Red Hat Advanced Cluster Management.