# Deploy RKE2 with Calico Cloud
Installing an RKE2 cluster that supports Calico Cloud

# Install on the master node

I'll be use an Ubuntu v.20.04 EC2 instance, as per the supported server options: <br/>
https://docs.rke2.io/install/requirements/

### Make sure you are a root user
```
sudo su
```
### Install cURL
```
agt-get update
apt install curl
```

### Run the installer
```
curl -sfL https://get.rke2.io | sh -
```
This will install the rke2-server service and the rke2 binary onto your machine.

### Experimental disabling of CNI plugins for RKE2
```
rke2 server --cni none
```
We can override the existing Canal CNI plugin with the above flag: <br/>
https://docs.rke2.io/install/network_options/

### Better to modify the configuration file to disable CNI plugins
To override Canal options you should be able to create HelmChartConfig resources <br/>
The HelmChartConfig resource must match the name and namespace of its corresponding HelmChart. <br/>
For example, to override Canal Options, you can create the following config: <br/>
File Name: rke2-canal-config.yml

```
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: rke2-calico
  namespace: kube-system
spec:
  valuesContent: |-
    calico:
      iface: "eth1"
```

This is referenced as a server config environmental variable: <br/>
https://docs.rke2.io/install/install_options/server_config/

```
mkdir -p /var/lib/rancher/rke2/server/manifests/
cp rke2-canal-config.yml /var/lib/rancher/rke2/server/manifests/
```

The config needs to be copied over to the manifests directory before installing RKE2

### Enable the rke2-server service
```
systemctl enable rke2-server.service
```

### Start the service
```
systemctl start rke2-server.service
```

### Follow the logs, if you like
```
journalctl -u rke2-server -f
```

### Wait a bit
```
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/var/lib/rancher/rke2/bin
kubectl get nodes
```
