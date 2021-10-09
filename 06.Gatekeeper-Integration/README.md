# Exercise 6 - Open Policy Agent Gatekeeper

In this exercise you will go through the Compliance features that come with Open Policy Agent Gatekeeper and the Compliance Operator. You will apply a number of policies to the cluster in order to comply with global security and management standards.

## Gatekeeper

In this section you create and manage Gatekeeper policies. The policies are based on the REGO policy language.

Apply the next policy to the hub cluster. The policy installs the Gatekeeper operator on the managed cluster.

```
<hub> $ cat >> policy-gatekeeper-operator.yaml << EOF
---
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-gatekeeper-operator
  namespace: rhacm-policies
  annotations:
    policy.open-cluster-management.io/standards: NIST SP 800-53
    policy.open-cluster-management.io/categories: CM Configuration Management
    policy.open-cluster-management.io/controls: CM-2 Baseline Configuration
spec:
  remediationAction: enforce
  disabled: false
  policy-templates:
  - objectDefinition:
      apiVersion: policy.open-cluster-management.io/v1
      kind: ConfigurationPolicy
      metadata:
        name: gatekeeper-operator-product-sub
      spec:
        remediationAction: enforce
        severity: high
        object-templates:
          - complianceType: musthave
            objectDefinition:
              apiVersion: operators.coreos.com/v1alpha1
              kind: Subscription
              metadata:
                name: gatekeeper-operator-product
                namespace: openshift-operators
              spec:
                channel: stable
                installPlanApproval: Automatic
                name: gatekeeper-operator-product
                source: redhat-operators
                sourceNamespace: openshift-marketplace
  - objectDefinition:
      apiVersion: policy.open-cluster-management.io/v1
      kind: ConfigurationPolicy
      metadata:
        name: gatekeeper
      spec:
        remediationAction: enforce
        severity: high
        object-templates:
          - complianceType: musthave
            objectDefinition:
              apiVersion: operator.gatekeeper.sh/v1alpha1
              kind: Gatekeeper
              metadata:
                name: gatekeeper
              spec:
                audit:
                  logLevel: INFO
                  replicas: 1
                image:
                  image: 'registry.redhat.io/rhacm2/gatekeeper-rhel8:v3.3.0'
                validatingWebhook: Enabled
                mutatingWebhook: Disabled
                webhook:
                  emitAdmissionEvents: Enabled
                  logLevel: INFO
                  replicas: 2
---
apiVersion: policy.open-cluster-management.io/v1
kind: PlacementBinding
metadata:
  name: binding-policy-gatekeeper-operator
  namespace: rhacm-policies
placementRef:
  name: placement-policy-gatekeeper-operator
  kind: PlacementRule
  apiGroup: apps.open-cluster-management.io
subjects:
- name: policy-gatekeeper-operator
  kind: Policy
  apiGroup: policy.open-cluster-management.io
---
apiVersion: apps.open-cluster-management.io/v1
kind: PlacementRule
metadata:
  name: placement-policy-gatekeeper-operator
  namespace: rhacm-policies
spec:
  clusterConditions:
    - status: "True"
      type: ManagedClusterConditionAvailable
  clusterSelector:
    matchExpressions:
      - { key: environment, operator: In, values: ["production"] }
EOF

<hub> $ oc apply -f policy-gatekeeper-operator.yaml
```

### Policy #1 - Disallow unencrypted routes

Apply the next policy to the hub cluster in order to deny the creation of http (not encrypted traffic) routes on the managed clusters -

```
<hub> $ cat >> policy-gatekeeper-httpsonly.yaml << EOF
---
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-gatekeeper-route-httpsonly
  namespace: rhacm-policies
  annotations:
    policy.open-cluster-management.io/standards: NIST SP 800-53
    policy.open-cluster-management.io/categories: CM Configuration Management
    policy.open-cluster-management.io/controls: CM-2 Baseline Configuration
spec:
  remediationAction: enforce
  disabled: false
  policy-templates:
    - objectDefinition:
        apiVersion: policy.open-cluster-management.io/v1
        kind: ConfigurationPolicy
        metadata:
          name: policy-gatekeeper-route-httpsonly
        spec:
          remediationAction: enforce
          severity: low
          object-templates:
            - complianceType: musthave
              objectDefinition:
                apiVersion: templates.gatekeeper.sh/v1beta1
                kind: ConstraintTemplate
                metadata:
                  name: k8shttpsonly
                  annotations:
                    description: Requires Route resources to be HTTPS only.
                spec:
                  crd:
                    spec:
                      names:
                        kind: K8sHttpsOnly
                  targets:
                    - target: admission.k8s.gatekeeper.sh
                      rego: |
                        package k8shttpsonly
                        violation[{"msg": msg}] {
                          input.review.object.kind == "Route"
                          re_match("^(route.openshift.io)/", input.review.object.apiVersion)
                          route := input.review.object
                          not https_complete(route)
                          msg := sprintf("Route should be https. tls configuration is required for %v", [route.metadata.name])
                        }
                        https_complete(route) = true {
                          route.spec["tls"]
                          count(route.spec.tls) > 0
                        }
            - complianceType: musthave
              objectDefinition:
                apiVersion: constraints.gatekeeper.sh/v1beta1
                kind: K8sHttpsOnly
                metadata:
                  name: route-https-only
                spec:
                  match:
                    kinds:
                      - apiGroups: ["route.openshift.io"]
                        kinds: ["Route"]
    - objectDefinition:
        apiVersion: policy.open-cluster-management.io/v1
        kind: ConfigurationPolicy
        metadata:
          name: policy-gatekeeper-audit-httpsonly
        spec:
          remediationAction: inform # will be overridden by remediationAction in parent policy
          severity: low
          object-templates:
            - complianceType: musthave
              objectDefinition:
                apiVersion: constraints.gatekeeper.sh/v1beta1
                kind: K8sHttpsOnly
                metadata:
                  name: route-https-only
                status:
                  totalViolations: 0
    - objectDefinition:
        apiVersion: policy.open-cluster-management.io/v1
        kind: ConfigurationPolicy
        metadata:
          name: policy-gatekeeper-admission-httpsonly
        spec:
          remediationAction: inform # will be overridden by remediationAction in parent policy
          severity: low
          object-templates:
            - complianceType: mustnothave
              objectDefinition:
                apiVersion: v1
                kind: Event
                metadata:
                  namespace: openshift-gatekeeper-system # set it to the actual namespace where gatekeeper is running if different
                  annotations:
                    constraint_action: deny
                    constraint_kind: K8sHttpsOnly
                    constraint_name: route-https-only
                    event_type: violation
---
apiVersion: policy.open-cluster-management.io/v1
kind: PlacementBinding
metadata:
  name: binding-policy-gatekeeper-route-httpsonly
  namespace: rhacm-policies
placementRef:
  name: placement-policy-gatekeeper-route-httpsonly
  kind: PlacementRule
  apiGroup: apps.open-cluster-management.io
subjects:
  - name: policy-gatekeeper-route-httpsonly
    kind: Policy
    apiGroup: policy.open-cluster-management.io
---
apiVersion: apps.open-cluster-management.io/v1
kind: PlacementRule
metadata:
  name: placement-policy-gatekeeper-route-httpsonly
  namespace: rhacm-policies
spec:
  clusterConditions:
    - status: "True"
      type: ManagedClusterConditionAvailable
  clusterSelector:
    matchExpressions:
      - { key: environment, operator: In, values: ["production"] }
EOF

<hub> $ oc apply -f policy-gatekeeper-httpsonly.yaml
```

Wait until both policies are in a compliant state before you move forward with the exercise.

Login to the managed cluster and try creating a web server using the next commands -

```
<managed cluster> $ oc new-project httpd-test

<managed cluster> $ oc new-app httpd
```

Try exposing the web server using an unsecure route

```
<managed cluster> $ oc expose svc/httpd
```

Try exposing the web server using a secure route

```
<managed cluster> $ oc create route edge --service=httpd
```

### Policy #2 - Namespace Management

In this section you will create a Gatekeeper based policy. The policy will disallow namespaces with the `state: dangerous` label. If a namespace has this label, its creation will be disallowed. Make sure to create a message that indicates the error.

An example of a disallowed namespace:

```
{
  "apiVersion": "v1",
  "kind": "Namespace",
  "metadata": {
    "labels": {
      "state": "dangerous"
    },
    "name": "michael"
  }
}
```

You make use the presentation and the previously created policies as a reference for this policy. Use the [rego playground](https://play.openpolicyagent.org/) to check the validity of your rego policy.

Check the validity of your policy by creating a violating namespace. The creation of the namespace should be disallowed -

```
<managed cluster> $ cat >> gatekeeper-disallowed-namespace.yaml << EOF
apiVersion: v1
kind: Namespace
metadata:
  labels:
    state: dangerous
  name: michael
EOF

<managed cluster> $ oc apply -f gatekeeper-disallowed-namespace.yaml
```

## Compliance Operator Integration

In this section you will perform an integration between Red Hat Advanced Cluster Management and the OpenSCAP Compliance Operator. You will create an RHACM policy that deploys the Compliance Operator. Afterwards, you will create an RHACM policy that initiates a compliance scan and monitors the results.

Run the next command to deploy the Compliance Operator using an RHACM policy -

```
<hub> $ oc apply -f https://raw.githubusercontent.com/michaelkotelnikov/rhacm-workshop/master/06.Gatekeeper-Integration/exercise-compliance-operator/policy-compliance-operator.yaml
```

Make sure that the policy has been deployed successfully in RHACM's Governance dashboard - The policy status needs to be **compliant**. The Compliance Operator is deployed in the `openshift-compliance` namespace on the managed cluster.

```
<managed cluster> $ oc get pods -n openshift-compliance
NAME                                                    READY   STATUS      RESTARTS   AGE
compliance-operator-8c9bc7466-8h4js                     1/1     Running     1          7m27s
ocp4-openshift-compliance-pp-6d7c7db4bd-wb5vf           1/1     Running     0          4m51s
rhcos4-openshift-compliance-pp-c7b548bd-8pbhq           1/1     Running     0          4m51s
```

Now that the Compliance Operator is deployed, initiate a compliance scan using an RHACM policy. To initiate a compliance scan, run the next command -

```
<hub> $ oc apply -f https://raw.githubusercontent.com/michaelkotelnikov/rhacm-workshop/master/06.Gatekeeper-Integration/exercise-compliance-operator/policy-moderate-scan.yaml
```

After running the command, a compliance scan is initiated. The scan will take about 5 minutes to complete. Run the next command on the managed cluster to check the status of the scan -

```
<managed cluster> $ oc get compliancescan -n openshift-compliance
NAME                     PHASE     RESULT
ocp4-moderate            RUNNING   NOT-AVAILABLE
rhcos4-moderate-master   RUNNING   NOT-AVAILABLE
rhcos4-moderate-worker   RUNNING   NOT-AVAILABLE
```

When the scan completes, the `PHASE` field will change to `DONE`.

After the scan completes, navigate to the RHACM governance dashboard. Note that the newly created policy is in a non-compliant state. Click on the policy name and navigate to **Status**. The `compliance-suite-moderate-results` ConfigurationPolicy is non-compliant because multiple ComplianceCheckResult objects indicate a `FAIL` check-status. To investigate the failing rules, press on _View details_ next to the `compliance-suite-moderate-results` ConfigurationPolicy.

Scroll down, you will notice all failing compliance check results. To understand why these rules failed the scan press on `View yaml` next to the failing rule name.

- Investigate the `ocp4-moderate-banner-or-login-template-set` ComplianceCheckResult. See what you can do to remediate the issue.
- Investigate the `ocp4-moderate-configure-network-policies-namespaces` ComplianceCheckResult. See what you can do to remediate the issue.
- Investigate the `rhcos4-moderate-master-no-empty-passwords` ComplianceCheckResult. See what you can do to remediate the issue. 