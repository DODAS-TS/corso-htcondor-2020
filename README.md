# Demo walk-through

## HTCondor on any K8s cluster in a nutshell

### Overview

### Requirements
- k3d: for local k8s cluster playground
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
k3d cluster create -s 4
kubectl cluster-info

- helm client
- kubectl client
- cert-manager app installed
- longhorn (not required in general, but needed for easy shared FS of this example)

### Manifests configuration

### Our first HTCondor on K8s deployment

### Playing with configuration changes

### Destroy our deployment

## Using Helm Charts

### Requirements

### Reproduce previous deployment with HELM

## Managing clusters with GitOps

### Requirements

### Managing HTCondor cluster with FluxCD
