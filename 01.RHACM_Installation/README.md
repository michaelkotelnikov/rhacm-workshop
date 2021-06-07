# Exercise 1 - Advanced Cluster Management Installation 

In your lab environment, the Advanced Cluster Management for Kubernetes operator is already installed. Usually, the installed operator is at an older version than the latest operator available. Therefore, it is recommended to remove the old Advanced Cluster Management operator, and replace it with a new one.


## 1.1 Removing the old Advanced Cluster Management instance

To remove the old version of Advanced Cluster Management -

*   Log into the OpenShift Console of the HUB cluster using the admin credentials (e.g - [https://console-openshift-console.apps.cluster.1111.sandbox.opentlc.com](https://console-openshift-console.apps.cluster-4ae4.4ae4.sandbox565.opentlc.com))
*   Navigate to **Operators** -> **Installed Operators** -> **Advanced Cluster Management for Kubernetes** -> **MultiClusterHub**.
*   Delete the MultiClusterHub multiclusterhub instance.
*   Wait until there are no pods in a _Terminating_ state in the _open-cluster-management_ namespace on the HUB cluster.

```
<hub> $ oc login -u admin -p <password> https://api.cluster.1111.sandbox.opentlc.com:6443

<hub> $ watch -n 1 oc get pods -n open-cluster-management
```

*   Uninstall the Advanced Cluster Management for Kubernetes operator by click on **Actions** -> **Uninstall Operator**.

## 1.2 Installing the new Advanced Cluster Management instance

To install the up-to-date instance of Advanced Cluster Management, follow the steps presented in the **Installation** section of the workshop’s presentation - [https://docs.google.com/presentation/d/1jhsO7tSsFoNwYouF-iBnlw5kA7Nx0Hoi-4HK0fFP1YI/edit?usp=sharing](https://docs.google.com/presentation/d/1jhsO7tSsFoNwYouF-iBnlw5kA7Nx0Hoi-4HK0fFP1YI/edit?usp=sharing)
