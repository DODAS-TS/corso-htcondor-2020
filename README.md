# Demo walk-through

## References

- [k8s quick start](https://kubernetes.io/docs/setup/)
- [k8s concepts](https://kubernetes.io/docs/concepts/)
- [helm](https://helm.sh/docs/chart_template_guide/#getting-started-with-a-chart-template)
- [fluxcd](https://toolkit.fluxcd.io/get-started/)
- [HTCondor dockers](https://github.com/htcondor/htcondor/tree/master/build/docker/services/base)
- [DODAS HTCondor guide](https://dodas-ts.github.io/dodas-apps/condor-helm/)

### Requirements
- k3d: for local k8s cluster playground
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.19.0/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

git clone https://github.com/DODAS-TS/corso-htcondor-2020.git && cd corso-htcondor-2020

k3d cluster create -i docker.io/rancher/k3s:v1.19.3-k3s3

kubectl cluster-info
```

- helm client
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

### Manifests configuration

Clone the demo repo:

```bash
git clone https://github.com/dodas-ts/corso-htcondor-2020 & cd corso-htcondor-2020
```

You'll find the kubernetes resource files in `kubernetes` folder.

### Our first HTCondor on K8s deployment

- Create a namespace to isolate the deployement from others

```bash
kubectl create namespace condor
```

- Make this namespace the default

```bash
kubectl config set-context --current --namespace=condor
```

- Deploy our configuration files

``` bash
cd kubernetes
kubectl apply -f ./
```

- Log on the client node and play a bit

``` bash
kubectl exec -ti <client-pod name> -- bash
export _condor_SEC_DEFAULT_AUTHENTICATION_METHODS=PASSWORD
export _condor_SEC_PASSWORD_FILE=/etc/pwd/pool_password
export _condor_COLLECTOR_HOST=master.condor.svc.cluster.local
export _condor_SCHEDD_HOST=schedd.condor.svc.cluster.local
export _condot_TOOL_DEBUG=D_FULLDEBUG,D_SECURITY

condor_status -any
condor_q
```

- If fails, let's debug. Log into daemons and try to restart with condor_reconfig

### Destroy our deployment

```bash
kubectl delete -f ./
```

## Using Helm Charts

The equivalent of the previous exercise can be found in the form of a Helm Chart defined in:

```bash
cd ../charts/htcondor
```

Let's take some time to look at how it works. (details available on helm official doc, see refs at the beggining of the page)


### Deploy your first release of the cluster


Let's fill the values with our preferences.

For example generata a cluster secret with:
```bash
openssl rand -base64 32
```

Then, let's install our release:

```bash
helm install  htcondor ./charts/htcondor_demo/ --values values_deploy_demo.yaml --cleanup-on-fail
```

Now try to change values of the wn replicas and upgrade the installation:

```bash
helm upgrade htcondor ./charts/htcondor_demo/ --values values_deploy_demo.yaml --cleanup-on-fail
```

You are also able to rollback to revision 1:

```bash
helm rollback htcondor 1
```

### Eventually uninstall the release

```bash
helm delete htcondor
```
## Managing clusters with GitOps

### Requirements
- Flux-CD CLI
```bash
curl -s https://toolkit.fluxcd.io/install.sh | sudo bash
flux check --pre
```

### Managing HTCondor cluster with FluxCD

Generate github access token to repo scope: https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token

Then register a repository for cluster control:

```bash
export GITHUB_TOKEN=<....85afb89061d150cfd>

flux bootstrap github \
  --owner=dciangot \
  --repository=corso-htcondor-flux-2020 \
  --branch=main \
  --path=charts/htcondor_demo \
  --personal
```

define a chart source

define a deployment

To update deployment --> commit a change
## A cluster with IAM token authN on private cloud in less than 5min

The Helm chart for a base installation onr a public cloud is in ./charts/htcondor/ 

And once filled the values with cluster secrets and the cluster public IP and host you can go with

```bash
helm install  htcondor ./charts/htcondor/ --values values_deploy_public.yaml --cleanup-on-fail
```

No you can try a remote submission with IAM token for example just remember to source:

```bash
export _condor_AUTH_SSL_CLIENT_CAFILE=/ca.crt
export _condor_SEC_DEFAULT_AUTHENTICATION_METHODS=SCITOKENS
export _condor_SCITOKENS_FILE=/tmp/token
export _condor_COLLECTOR_HOST=<public IP>:30618
export _condor_SCHEDD_HOST=schedd.condor.svc.cluster.local
export _condot_TOOL_DEBUG=D_FULLDEBUG,D_SECURITY
```

Also you need to copy the self signed CA from the cluster (that was automaticcaly generated) in `_condor_AUTH_SSL_CLIENT_CAFILE` and put your access token in _condor_SCITOKENS_FILE
