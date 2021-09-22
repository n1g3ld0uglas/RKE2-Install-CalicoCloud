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

This is referenced as a server config environmental variable: <br/>
https://docs.rke2.io/install/install_options/server_config/

### Better to modify the configuration file to disable CNI plugins
By default, RKE2 will launch with the values present in the YAML file located at: ```/etc/rancher/rke2/config.yaml```

```
write-kubeconfig-mode: "0644"
tls-san:
  - "foo.local"
node-label:
  - "foo=bar"
  - "something=amazing"
cni:
  - "none"
```

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
