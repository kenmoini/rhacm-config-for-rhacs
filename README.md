# RHACM Configuration for RHACS

This repository provides a good starting point for a Red Hat Advanced Cluster Security demo, powered by Red Hat Advanced Cluster Management.

## Deployment

On the Hub Cluster with RHACM installed, run the following command:

```bash
oc apply -k bootstrap/rhacm/

oc apply -f rhacm-config/placement-rules/
oc apply -f rhacm-config/placement-bindings/
oc apply -f rhacm-config/policies/

kustomize build --enable-alpha-plugins rhacm-config/policy-generator/rhacs-central | oc apply -f -

# Create a cluster-init bundle and apply it to the Hub cluster in the stackrox namespace

kustomize build --enable-alpha-plugins rhacm-config/policy-generator/rhacs-secured-cluster | oc apply -f -
```

## Bad Faith Workloads

In order for OCP and RHACS to really shine, you need some bad actors.

```bash
kustomize build --enable-alpha-plugins rhacm-config/policy-generator/bad-faith-workloads | oc apply -f -
```

Or apply the AppSubs in the `extras` folder.