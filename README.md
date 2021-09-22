# Deploy RKE2 with Calico Cloud
Installing an RKE2 cluster that supports Calico Cloud<br/>
https://github.com/rancher/rke2

If your cluster is configured with the networking plugin calico or canal you must edit the configuration and switch it to none to ensure future updates do not attempt to manage the networking configuration: <br/>
https://docs.calicocloud.io/install/system-requirements

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
<br/>
Grabbed the environmental variables from this link:<br/>
https://github.com/rancher/rke2-charts/blob/main-source/packages/rke2-canal/charts/values.yaml


File Name: rke2-canal-config.yml

```
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: rke2-canal
  namespace: kube-system
spec:
  valuesContent: |-
    calico:
      cniImage:
        repository: "rancher/hardened-calico"
        tag: "v3.19.1-build20210611"
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

# Linux Agent (Worker) Node Installation
### Run the installer
You need to install on at least one worker node: <br/>
https://docs.rke2.io/install/quickstart/

```
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
```
This will install the rke2-agent service and the rke2 binary onto your machine.

### Enable the rke2-agent service
```
systemctl enable rke2-agent.service
```
### Configure the rke2-agent service
```
mkdir -p /etc/rancher/rke2/
vim /etc/rancher/rke2/config.yaml
```

### Content for config.yaml:
```
server: https://<server>:9345
token: <token from server node>
```

Note: The rke2 server process listens on port 9345 for new nodes to register.<br/> 
The Kubernetes API is still served on port 6443, as normal.

### Start the service
```
systemctl start rke2-agent.service
```
### Follow the logs, if you like
```
journalctl -u rke2-agent -f
```

Note: Each machine must have a unique hostname. If your machines do not have unique hostnames, set the node-name parameter in the config.yaml file and provide a value with a valid and unique hostname for each node.
