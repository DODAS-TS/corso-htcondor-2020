# Demo walk-through

## References

k8s quick start
helm
fluxcd
htcondor dockers
dodas legacy

### Requirements
- k3d: for local k8s cluster playground

curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.19.0/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

git clone https://github.com/DODAS-TS/corso-htcondor-2020.git && cd corso-htcondor-2020

k3d cluster create -i docker.io/rancher/k3s:v1.19.3-k3s3

kubectl cluster-info

- helm client
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

### Manifests configuration


### Our first HTCondor on K8s deployment

- Create a namespace

kubectl create namespace condor


- Make this namespace the default

kubectl config set-context --current --namespace=condor

- Deploy our configuration files

cd kubernetes
kubectl apply -f ./


- Log on the client node and play a bit

kubectl exec -ti <client-pod name> -- bash
export _condor_SEC_DEFAULT_AUTHENTICATION_METHODS=PASSWORD
export _condor_SEC_PASSWORD_FILE=/etc/pwd/pool_password
export _condor_COLLECTOR_HOST=master.condor.svc.cluster.local
export _condor_SCHEDD_HOST=schedd.condor.svc.cluster.local
export _condot_TOOL_DEBUG=D_FULLDEBUG,D_SECURITY

condor_status -any
condor_q

- If fails, let's debug. Log into daemons and try to restart with condor_reconfig

### Destroy our deployment

kubectl delete -f ./

## Using Helm Charts

cd ../charts/htcondor

Definition: Charts.yaml

### Reproduce previous deployment with HELM

Templates

Values

### Deploy your first release of the cluster

fill the values

openssl rand -base64 32

helm install  htcondor ./charts/htcondor_demo/ --values values_deploy_demo.yaml --cleanup-on-fail

scale down wn and upgrade

helm upgrade htcondor ./charts/htcondor_demo/ --values values_deploy_demo.yaml --cleanup-on-fail

rollback to revision 1:

helm rollback htcondor 1

## Managing clusters with GitOps

### Requirements
- Flux-CD CLI
curl -s https://toolkit.fluxcd.io/install.sh | sudo bash
flux check --pre

### Managing HTCondor cluster with FluxCD

Generate github token
export GITHUB_TOKEN=9ca03d9974b4812798a03ff85afb89061d150cfd
https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token

flux bootstrap github \
  --owner=dciangot \
  --repository=corso-htcondor-flux-2020 \
  --branch=main \
  --path=charts/htcondor_demo \
  --personal

define a chart source

define a deployment


update deployment --> commit a change
## A cluster on private cloud in less than 5min

The template for a public cloud:


### Try a remote submission

## DODAS cluster example
