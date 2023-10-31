# RHACM Configuration for RHACS

This repository provides a good starting point for a Red Hat Advanced Cluster Security demo, powered by Red Hat Advanced Cluster Management.

## Deployment

1. On the Hub Cluster, run the following command:

```bash
# Install the RHACM Operator
oc apply -k bootstrap/rhacm-operator-install/overlays/release-2.8/

# Deploy RHACM MultiClusterHub
oc apply -k bootstrap/rhacm-instance/base/

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
