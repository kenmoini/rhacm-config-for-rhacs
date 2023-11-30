# RHACM Configuration for RHACS

This repository provides a good starting point for a Red Hat Advanced Cluster Security demo, powered by Red Hat Advanced Cluster Management.

## Deployment

1. On the Hub Cluster, run the following command:

```bash
# Install the RHACM Operator
oc apply -k bootstrap/rhacm-operator-install/overlays/release-2.9/

# Deploy RHACM MultiClusterHub
oc apply -k bootstrap/rhacm-instance/base/
# or, if deploying to infra nodes
oc apply -k bootstrap/rhacm-instance/overlays/infra-nodes/

# Configure RHACM for RHACS, Compliance Operator, and more
oc apply -k bootstrap/rhacm-config/base/
```

2. Next, label the `local-cluster` in RHACM with `rhacs-central-cluster=true` - this will deploy RHACS Central.
3. Log into RHACS, create a cluster-init bundle and apply it to the Hub cluster in the `stackrox` namespace
4. Label any clusters with `rhacs-secured-cluster=true` to deploy as a Secured Cluster and have it automatically be bound to RHACS.

## Bad Faith Workloads

In order for OCP and RHACS to really shine, you need some bad actors.  You'll find a few ACM Policies applied that will deploy Webgoat and a Cryptominer.

## Compliance Operator

There is also a RHACM Policy that will deploy the Compliance Operator to any OpenShift cluster and run a CIS and E8 scan on all the nodes.

> Your Control Plane nodes need to be able to bind PVCs to Pods.

With ODF you will likely need ODF configured to run the CSI Plugin on the control plane nodes as well:

- https://access.redhat.com/solutions/6047841
- https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.6/html-single/managing_and_allocating_storage_resources/index#managing-container-storage-interface-component-placements_rhocs

Add the following to the ConfigMap in `openshift-storage/rook-ceph-operator-config`:

```yaml=
  CSI_PLUGIN_TOLERATIONS: |
    - key: node-role.kubernetes.io/master
      effect: NoSchedule
```

## Issue: RHACM AppSub Not Syncing

You probably need to add your user to the `open-cluster-management:subscription-admin` ClusterRoleBinding: https://github.com/open-cluster-management-io/policy-collection#subscription-administrator

## Issue: RHACM Application Controller not working through a proxy

This whole thing is a pain in the ass if you're behind a proxy.  Just deploy things manually instead of using the AppSub:

```bash=
oc apply -f bootstrap/base/00-namespaces.yml
oc apply -f bootstrap/base/00-hub-cluster-set.yml
oc apply -f bootstrap/base/10-managedclustersetbindings.yml
oc apply -f bootstrap/base/10-placement.yml
oc apply -f bootstrap/base/10-placementrules.yml

oc apply -k rhacm-config/placement-rules/
oc apply -k rhacm-config/placement-bindings/
oc apply -k rhacm-config/policies/

# Download the PolicyGenerator Kustomize plugin if needed: https://github.com/open-cluster-management-io/policy-generator-plugin

cd rhacm-config/policy-generator
kustomize build --enable-alpha-plugins | oc apply -f -
```