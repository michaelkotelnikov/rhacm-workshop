# Exercise 7 - Ansible Tower Integration

In this exercise you will go through Ansible Tower integration with Red Hat Advanced Cluster Management for Kubernetes. You will associate AnsibleJob hooks for applications and integrate AnsibleJobs with policy violations. Ansible Tower has already been configured for your use by the instructor, you will only configure Red Hat Advanced Cluster Management for Kubernetes.

The instructor will provide you with -

* Ansible Tower URL
* Ansible Tower web UI username / password
* Ansible Tower Access Token for API requests

## Before You Begin

In this section you will create the basic integration between RHACM and Ansible Tower. The integration is based on `Ansible Automation Platform Resource Operator`. Make sure to install the operator before you begin the next exercises. Installing the operator can be done by running the next commands on the hub cluster -

```
<hub> $ oc new-project ansible-resource-operator

<hub> $ cat >> ansible-operator.yaml << EOF
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: awx-resource-operator
  namespace: ansible-resource-operator
spec:
  channel: release-0.1
  installPlanApproval: Automatic
  name: awx-resource-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  startingCSV: awx-resource-operator.v0.1.1
EOF

<hub> $ oc apply -f ansible-operator.yaml
```

The operator will now begin the installation process.

## Ansible Tower Application Integration

In this section, you will configure Ansible Tower Jobs to run as your RHACM Application deploys. The first job will run as a _prehook_ while the second job will run as a _posthook_.

Both Ansible Jobs will initiate the same Job Template on Ansible Tower called _Logger_. The _Logger_ Job Template logs each running instance of an Ansible Job to a file on the Ansible Tower server. Afterwards, the _Logger_ Job Template will expose the log files on a local web server on Ansible Tower. The participants can view all log files on the Ansible Tower server by navigating to the URL provided by the instructor in **port 80**.

### Setting up Authentication

In order to allow RHACM to access Ansible Tower you must set up a Namespace scoped secret for RHACM to use. The secret will contain the Ansible Tower URL and Access Token.
