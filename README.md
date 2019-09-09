# eks-gitops


```
kubectl delete storageclass gp2 #eks ships with gp2 on and default
```
```
# ensure Helm is installed/configured
cd ~/environment/
kubectl apply -f ~/environment/rbac.yaml
helm init --service-account tiller
```
```
# Install the flux CLI (optional)
cd ~/environment/
wget https://github.com/fluxcd/flux/releases/download/1.14.2/fluxctl_linux_amd64
sudo mv fluxctl_linux_amd64 /usr/local/bin/fluxctl
sudo chmod +x /usr/local/bin/fluxctl
```
```
# Add the Flux helm repo and apply the flux CRDs
helm repo add fluxcd https://charts.fluxcd.io
kubectl apply -f https://raw.githubusercontent.com/fluxcd/flux/helm-0.10.1/deploy-helm/flux-helm-release-crd.yaml
```
```
# Install flux, and configure it to pull from our repo
helm install --name flux \
--set rbac.create=true \
--set helmOperator.create=true \
--set helmOperator.createCRD=false \
--set git.url=git@github.com:brentlangston/eks-gitops \
--namespace flux \
fluxcd/flux

watch kubectl -n flux get pods 
```
```
# Pull the ssh key and copy 
fluxctl --k8s-fwd-ns flux identity

# paste as deploy key w/write access:
https://github.com/brentlangston/eks-gitops/settings/keys
```
```
# Trigger a sync (optional)
fluxctl --k8s-fwd-ns flux sync

# Observe new packages are installed (optional)
helm ls

# Fluxctl to view workloads (optional)
fluxctl list-workloads --k8s-fwd-ns flux 

# Fix grafana (if necessary)
helm upgrade --reuse-values --set persistence.enabled=true grafana stable/grafana
```